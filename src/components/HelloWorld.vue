<template>
    <div class="grid">
        <canvas
            v-on:click.stop="setCell()"
            id="canvas"
            width="420px"
            height="420px"
            style="background: #fff; margin:20px"
        ></canvas>

        <button v-on:click.stop="next()">Next</button>
        <button v-show="!started" v-on:click.stop="start()">
            Start
        </button>
        <button v-show="started" v-on:click.stop="stop()">Stop</button>
        <input v-model="fps" type="number" placeholder="edit me" value="1" />
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

interface Renderer {
    renderGrid(grid: Grid): void;
}

class DefaultRenderer implements Renderer {
    constructor(private context: CanvasRenderingContext2D) {}

    renderGrid(grid: Grid): void {
        // Box width
        const bw = 400;
        // Box height
        const bh = 400;
        // Padding
        const p = 10;
        const incW = 400 / grid.getRows();
        const incH = 400 / grid.getCols();
        this.context.clearRect(0, 0, bw + p, bh + p);
        for (let x = 0; x <= bw; x += incW) {
            this.context.moveTo(0.0 + x + p, p);
            this.context.lineTo(0.0 + x + p, bh + p);
        }
        for (let x = 0; x <= bh; x += incH) {
            this.context.moveTo(p, 0 + x + p);
            this.context.lineTo(bw + p, 0.0 + x + p);
        }
        this.context.strokeStyle = "black";
        this.context.stroke();
        const lives = grid.getLives();
        for (let i = 0; i < lives.length; i++) {
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

    constructor(private rows: number, private cols: number) {
        this.lives = Array(rows);
        for (let r = 0; r < rows; r++) {
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
        this.lives[row][col] = null;
    }

    neighbours(row: number, col: number): Neighbours {
        return new Neighbours(this, row, col);
    }

    render(renderer: Renderer) {
        renderer.renderGrid(this);
    }

    update(updater: Updater) {
        updater.update(this);
    }
}

class Updater {
    private deleteQ: Array<any> = [];
    private addQ: Array<any> = [];

    update(grid: Grid) {
        const lives = grid.getLives();
        for (let i = 0; i < lives.length; i++) {
            for (let j = 0; j < lives[i].length; j++) {
                if (lives[i][j] != null) {
                    if (this.hasLessThanTwoNeighbours(i, j, grid)) {
                        this.deleteQ.push([i, j]);
                    } else if (this.hasTwoOrThreeNeighbours(i, j, grid)) {
                        //live on..
                    } else if (this.hasMoreThanThreeNeighbours(i, j, grid)) {
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

@Component
export default class HelloWorld extends Vue {
    private started = true;
    private fps = 1;
    private context: any;
    private grid: Grid;
    private updater: Updater;
    private renderer: any;
    private animationFrame: any;
    private timeoutHandler: any;

    next() {
        this.grid.update(this.updater);
        this.grid.render(this.renderer);
    }

    constructor() {
        super();
        this.grid = new Grid(30, 30);

        this.grid.add(new Life(1), 3, 3);
        this.grid.add(new Life(1), 2, 1);
        //   this.grid.add(new Life(1), 2, 3);
        this.grid.add(new Life(1), 1, 2);
        //   this.grid.add(new Life(1), 1, 3);
        this.grid.add(new Life(1), 1, 4);
        this.grid.add(new Life(1), 1, 0);
        this.grid.add(new Life(1), 1, 5);
        this.grid.add(new Life(1), 1, 6);
        this.updater = new Updater();
    }

    mounted() {
        const canvas = document.getElementById("canvas") as HTMLCanvasElement;
        this.context = canvas.getContext("2d");
        this.renderer = new DefaultRenderer(this.context);
        this.grid.render(this.renderer);
        this.draw();
    }

    stop() {
        //cancelAnimationFrame(this.animationFrame);
        this.started = false;
    }

    start() {
        this.started = true;
        this.draw();
    }

    draw() {
        if(!this.started) {
            return;
        }
        setTimeout(() => {
            this.grid.update(this.updater);
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
