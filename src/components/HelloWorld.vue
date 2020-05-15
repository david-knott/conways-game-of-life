<template>
    <div class="grid">
        <canvas
            v-on:click.stop="setCell"
            id="canvas"
            width="620px"
            height="620px"
            style="background: #fff; margin:20px"
        ></canvas>
        <div>
            <button v-show="!started" v-on:click.stop="next()">Next</button>
            <button v-show="!started" v-on:click.stop="start()">
                Start
            </button>
            <button v-show="started" v-on:click.stop="stop()">Stop</button>
            <input
                v-model="fps"
                type="number"
                placeholder="edit me"
                value="1"
            />
        </div>
        <p>
            The universe of the Game of Life is an infinite, two-dimensional
            orthogonal grid of square cells, each of which is in one of two
            possible states, live or dead, (or populated and unpopulated,
            respectively). Every cell interacts with its eight neighbours, which
            are the cells that are horizontally, vertically, or diagonally
            adjacent. At each step in time, the following transitions occur:
        </p>
        <ul>
            <li>
                Any live cell with fewer than two live neighbours dies, as if by
                underpopulation.
            </li>
            <li>
                Any live cell with two or three live neighbours lives on to the
                next generation.
            </li>
            <li>
                Any live cell with more than three live neighbours dies, as if
                by overpopulation.
            </li>
            <li>
                Any dead cell with exactly three live neighbours becomes a live
                cell, as if by reproduction.
            </li>
        </ul>
        <p>
            These rules, which compare the behavior of the automaton to real
            life, can be condensed into the following: Any live cell with two or
            three live neighbours survives. Any dead cell with three live
            neighbours becomes a live cell. All other live cells die in the next
            generation. Similarly, all other dead cells stay dead. The initial
            pattern constitutes the seed of the system. The first generation is
            created by applying the above rules simultaneously to every cell in
            the seed; births and deaths occur simultaneously, and the discrete
            moment at which this happens is sometimes called a tick. Each
            generation is a pure function of the preceding one. The rules
            continue to be applied repeatedly to create further generations.
        </p>
    </div>
</template>

<script lang="ts">
import { Component, Prop, Vue } from "vue-property-decorator";
import { BootstrapVue, BootstrapVueIcons } from "bootstrap-vue";

import "bootstrap/dist/css/bootstrap.css";
import "bootstrap-vue/dist/bootstrap-vue.css";

Vue.use(BootstrapVue);
Vue.use(BootstrapVueIcons);

interface Renderer {
    renderGrid(grid: Grid): void;
    addLife(grid: Grid, updater: Updater, x: number, y: number): void;
}

class DefaultRenderer implements Renderer {
    private p = 0;

    constructor(
        private bw: number,
        private bh: number,
        private context: CanvasRenderingContext2D
    ) {}

    addLife(grid: Grid, updater: Updater, x: number, y: number): void {
        const p = this.p;
        const incW = this.bw / grid.getRows();
        const incH = this.bh / grid.getCols();
        const xpos = Math.floor((x - p) / incW);
        const ypos = Math.floor((y - p) / incH);
        if (grid.getLives()[xpos][ypos] == null) {
            updater.add(xpos, ypos);
        } else {
            updater.remove(xpos, ypos);
        }
    }

    renderGrid(grid: Grid): void {
        const p = this.p;
        const incW = this.bw / grid.getRows();
        const incH = this.bh / grid.getCols();
        this.context.clearRect(0, 0, this.bw + p, this.bh + p);
        for (let x = 0; x <= this.bw; x += incW) {
            this.context.moveTo(0.0 + x + p, p);
            //       this.context.lineTo(0.0 + x + p, this.bh + p);
        }
        for (let x = 0; x <= this.bh; x += incH) {
            this.context.moveTo(p, 0 + x + p);
            //     this.context.lineTo(this.bw + p, 0.0 + x + p);
        }
        this.context.strokeStyle = "black";
        this.context.stroke();
        const lives = grid.getLives();
        for (let i = 0; i < lives.length; i++) {
            if (grid.isEmptyRow(i)) {
                continue;
            }
            for (let j = 0; j < lives[i].length; j++) {
                const life = lives[i][j];
                if (life) {
                    this.context.fillRect(
                        i * incW + p,
                        j * incH + p,
                        incW,
                        incH
                    );
                }
            }
        }
    }
}

class Life {
    constructor(private colour: number) {}
}

class Neighbours {
    private north: Life;
    private northEast: Life;
    private northWest: Life;
    private south: Life;
    private southEast: Life;
    private southWest: Life;
    private east: Life;
    private west: Life;
    constructor(grid: Grid, row: number, col: number) {
        this.north = grid.getLives()[grid.getWrappedRow(row + 1)][col];
        this.northEast = grid.getLives()[grid.getWrappedRow(row + 1)][
            grid.getWrappedRow(col + 1)
        ];
        this.northWest = grid.getLives()[grid.getWrappedRow(row + 1)][
            grid.getWrappedRow(col - 1)
        ];
        this.south = grid.getLives()[grid.getWrappedRow(row - 1)][col];
        this.southEast = grid.getLives()[grid.getWrappedRow(row - 1)][
            grid.getWrappedRow(col + 1)
        ];
        this.southWest = grid.getLives()[grid.getWrappedRow(row - 1)][
            grid.getWrappedRow(col - 1)
        ];
        this.west = grid.getLives()[row][grid.getWrappedCol(col - 1)];
        this.east = grid.getLives()[row][grid.getWrappedCol(col + 1)];
    }

    live(): number {
        let c = 0;
        if (this.north != null) c++;
        if (this.northEast != null) c++;
        if (this.northWest != null) c++;
        if (this.south != null) c++;
        if (this.southEast != null) c++;
        if (this.southWest != null) c++;
        if (this.east != null) c++;
        if (this.west != null) c++;
        return c;
    }
}

class Grid {
    private lives: any[][];
    private rowState: any[];

    constructor(private rows: number, private cols: number) {
        this.lives = Array(rows);
        this.rowState = Array(rows);
        for (let r = 0; r < rows; r++) {
            this.rowState[r] = 0;
            this.lives[r] = Array(rows);
            for (let c = 0; c < cols; c++) {
                this.lives[r][c] = null;
            }
        }
    }

    getRows(): number {
        return this.rows;
    }

    getCols(): number {
        return this.cols;
    }

    isEmptyRow(i: number): boolean {
        return this.rowState[i] === 0;
    }

    getWrappedRow(row: number): number {
        if (row >= this.rows) {
            row = 0;
        }
        if (row < 0) {
            row = this.rows + row;
        }
        return row;
    }

    getWrappedCol(col: number): number {
        if (col >= this.cols) {
            col = 0;
        }
        if (col < 0) {
            col = this.cols + col;
        }
        return col;
    }

    getLives() {
        return this.lives;
    }

    add(life: Life, row: number, col: number) {
        if (row >= this.rows)
            throw new Error(
                "Cannot add to row greater that total defined rows"
            );
        if (col >= this.cols)
            throw new Error(
                "Cannot add to col greater that total defined cols"
            );
        this.lives[row][col] = life;
        this.rowState[row] = this.rowState[row] + 1;
    }

    delete(row: number, col: number) {
        if (row >= this.rows)
            throw new Error(
                "Cannot add to row greater that total defined rows"
            );
        if (col >= this.cols)
            throw new Error(
                "Cannot add to col greater that total defined cols"
            );
        this.rowState[row] = this.rowState[row] - 1;
        this.lives[row][col] = null;
    }

    neighbours(row: number, col: number): Neighbours {
        return new Neighbours(this, row, col);
    }

    render(renderer: Renderer) {
        renderer.renderGrid(this);
    }

    update(updater: Updater, move: boolean) {
        updater.update(this, move);
    }
}

class Updater {
    private deleteQ: Array<any> = [];
    private addQ: Array<any> = [];

    add(i: number, j: number) {
        this.addQ.push([i, j]);
    }

    remove(i: number, j: number) {
        this.deleteQ.push([i, j]);
    }

    update(grid: Grid, move: boolean) {
        const lives = grid.getLives();
        if (move) {
            for (let i = 0; i < lives.length; i++) {
                for (let j = 0; j < lives[i].length; j++) {
                    if (lives[i][j] != null) {
                        if (this.hasLessThanTwoNeighbours(i, j, grid)) {
                            this.deleteQ.push([i, j]);
                        } else if (this.hasTwoOrThreeNeighbours(i, j, grid)) {
                            //live on..
                        } else if (
                            this.hasMoreThanThreeNeighbours(i, j, grid)
                        ) {
                            this.deleteQ.push([i, j]);
                        } else {
                            this.deleteQ.push([i, j]);
                        }
                    } else {
                        if (this.hasThreeNeighbours(i, j, grid)) {
                            this.addQ.push([i, j]);
                        }
                    }
                }
            }
        }
        while (this.deleteQ.length > 0) {
            const pos = this.deleteQ.pop();
            grid.delete(pos[0], pos[1]);
        }
        while (this.addQ.length > 0) {
            const pos = this.addQ.pop();
            grid.add(new Life(1), pos[0], pos[1]);
        }
    }

    hasLessThanTwoNeighbours(i: number, j: number, grid: Grid) {
        const neighbours = grid.neighbours(i, j);
        return neighbours.live() < 2;
    }

    hasTwoOrThreeNeighbours(i: number, j: number, grid: Grid) {
        const neighbours = grid.neighbours(i, j);
        return neighbours.live() === 2 || neighbours.live() === 3;
    }

    hasThreeNeighbours(i: number, j: number, grid: Grid) {
        const neighbours = grid.neighbours(i, j);
        return neighbours.live() === 3;
    }
    hasMoreThanThreeNeighbours(i: number, j: number, grid: Grid) {
        const neighbours = grid.neighbours(i, j);
        return neighbours.live() > 3;
    }
}

class Pattern {
    constructor(private name: string, private cells: number[][]) {}

    add(grid: Grid, left: number, top: number) {
        for (let i = 0; i < this.cells.length; i++) {
            for (let j = 0; j < this.cells[i].length; j++) {
                const l = this.cells[i][j];
                if (l) {
                    grid.add(new Life(1), top + j, left + i);
                }
            }
        }
    }
}

@Component
export default class HelloWorld extends Vue {
    private started = true;
    private fps = 10;
    private context: any;
    private grid: Grid;
    private updater: Updater;
    private renderer: any;
    private animationFrame: any;
    private timeoutHandler: any;
    private patterns: Array<Pattern>;

    next() {
        this.grid.update(this.updater, true);
        this.grid.render(this.renderer);
    }

    constructor() {
        super();
        this.grid = new Grid(60, 60);
        this.patterns = [];

        this.patterns.push(
            new Pattern("block", [
                [1, 1],
                [1, 1],
            ])
        );
        this.patterns.push(
            new Pattern("beehive", [
                [0, 1, 1, 0],
                [1, 0, 0, 1],
                [0, 1, 1, 0],
            ])
        );
        this.patterns.push(
            new Pattern("loaf", [
                [0, 1, 1, 0],
                [1, 0, 0, 1],
                [0, 1, 0, 1],
                [0, 0, 1, 0],
            ])
        );
        this.patterns.push(
            new Pattern("boat", [
                [1, 1, 0, 0],
                [1, 0, 1, 0],
                [0, 1, 0, 0],
            ])
        );
        this.patterns.push(
            new Pattern("tub", [
                [0, 1, 0],
                [1, 0, 1],
                [0, 1, 0],
            ])
        );
        this.patterns.push(new Pattern("blinker", [[1, 1, 1]]));
        this.patterns.push(
            new Pattern("toad", [
                [0, 1, 1, 1],
                [1, 1, 1, 0],
            ])
        );
        this.patterns.push(
            new Pattern("toad", [
                [0, 1, 1, 1],
                [1, 1, 1, 0],
            ])
        );
        this.patterns.push(
            new Pattern("glider", [
                [0, 1, 0],
                [0, 0, 1],
                [1, 1, 1],
            ])
        );


        //    this.grid.add(new Life(1), 3, 3);
        //   this.grid.add(new Life(1), 2, 1);
        //   this.grid.add(new Life(1), 2, 3);
        //   this.grid.add(new Life(1), 1, 2);
        //   this.grid.add(new Life(1), 1, 3);
        //   this.grid.add(new Life(1), 1, 4);
        // this.grid.add(new Life(1), 1, 0);
        // this.grid.add(new Life(1), 1, 5);
        //  this.grid.add(new Life(1), 1, 6);
        ///        this.grid.add(new Life(1), 5, 5);
        //        this.grid.add(new Life(1), 5, 6);
        //      this.grid.add(new Life(1), 5, 7);
        //    this.grid.add(new Life(1), 6, 5);
        //  this.grid.add(new Life(1), 7, 6);

        this.updater = new Updater();
    }

    mounted() {
        const canvas = document.getElementById("canvas") as HTMLCanvasElement;
        this.context = canvas.getContext("2d");
        this.renderer = new DefaultRenderer(
            canvas.width,
            canvas.height,
            this.context
        );
        this.patterns[this.patterns.length - 1].add(this.grid, 15, 15);
        this.patterns[this.patterns.length - 1].add(this.grid, 1, 1);
        this.patterns[this.patterns.length - 1].add(this.grid, 4, 40);
        this.patterns[this.patterns.length - 1].add(this.grid, 4, 10);
        this.patterns[this.patterns.length - 2].add(this.grid, 4, 10);
        this.grid.render(this.renderer);
        this.draw();
    }

    stop() {
        this.started = false;
    }

    start() {
        this.started = true;
    }

    setCell(e: any) {
        const canvas = document.getElementById("canvas") as HTMLCanvasElement;
        let x;
        let y;
        if (e.pageX || e.pageY) {
            x = e.pageX;
            y = e.pageY;
        } else {
            x =
                e.clientX +
                document.body.scrollLeft +
                document.documentElement.scrollLeft;
            y =
                e.clientY +
                document.body.scrollTop +
                document.documentElement.scrollTop;
        }
        x -= canvas.offsetLeft;
        y -= canvas.offsetTop;
        this.renderer.addLife(this.grid, this.updater, x, y);
    }

    draw() {
        setTimeout(() => {
            this.grid.update(this.updater, this.started);
            this.grid.render(this.renderer);
            window.requestAnimationFrame(this.draw);
            return false;
        }, 1000 / this.fps);
    }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
h3 {
    margin: 40px 0 0;
}
ul {
    list-style-type: none;
    padding: 0;
}
li {
    display: inline-block;
    margin: 0 10px;
}
a {
    color: #42b983;
}
</style>
