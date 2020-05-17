<template>
  <div class="grid">
    <canvas
      v-on:click.stop="setCell"
      id="canvas"
      width="600px"
      height="600px"
      style="background: #fff; margin:20px"
    ></canvas>
    <div>
      <v-btn v-show="!started" v-on:click.stop="next()">Next</v-btn>
      <v-btn v-show="!started" v-on:click.stop="start()">Start</v-btn>
      <v-btn v-show="started" v-on:click.stop="stop()">Stop</v-btn>
      <v-text-field v-model="fps" type="number" placeholder="edit me" value="1" />
    </div>
  </div>
</template>

<script lang="ts">
import { Component, Prop, Vue } from "vue-property-decorator";

interface Renderer {
  renderGrid(grid: Grid): void;
  addLife(grid: Grid, updater: Updater, x: number, y: number): void;
  addPattern(
    grid: Grid,
    updater: Updater,
    x: number,
    y: number,
    pattern: Pattern
  ): void;
}

class DefaultRenderer implements Renderer {
  constructor(
    private bw: number,
    private bh: number,
    private context: CanvasRenderingContext2D
  ) {}

  addLife(grid: Grid, updater: Updater, x: number, y: number): void {
    const incW = this.bw / grid.getRows();
    const incH = this.bh / grid.getCols();
    const xpos = Math.floor(x / incW);
    const ypos = Math.floor(y / incH);
    if (grid.getLives()[xpos][ypos] == null) {
      updater.add(xpos, ypos);
    } else {
      updater.remove(xpos, ypos);
    }
  }

  addPattern(
    grid: Grid,
    updater: Updater,
    x: number,
    y: number,
    pattern: Pattern
  ): void {
    const incW = this.bw / grid.getRows();
    const incH = this.bh / grid.getCols();
    const xpos = Math.floor(x / incW);
    const ypos = Math.floor(y / incH);
    pattern.add(grid, xpos, ypos);
  }

  renderGrid(grid: Grid): void {
    const incW = this.bw / grid.getRows();
    const incH = this.bh / grid.getCols();
    this.context.fillStyle = "#FFFFFF";
    this.context.fillRect(0, 0, this.bw, this.bh);
    this.context.fillStyle = "#ff0000";
    for (let x = 0; x <= this.bw; x += incW) {
      this.context.moveTo(0.0 + x, 0);
      this.context.lineTo(0.0 + x, this.bh);
    }
    for (let x = 0; x <= this.bh; x += incH) {
      this.context.moveTo(0, 0 + x);
      this.context.lineTo(this.bw, 0.0 + x);
    }
    this.context.strokeStyle = "grey";
    this.context.stroke();
    const lives = grid.getLives();
    for (let i = 0; i < lives.length; i++) {
      if (grid.isEmptyRow(i)) {
        continue;
      }
      for (let j = 0; j < lives[i].length; j++) {
        const life = lives[i][j];
        if (life) {
          this.context.fillStyle = "#ff0000";
          this.context.fillRect(i * incW, j * incH, incW, incH);
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
      throw new Error("Cannot add to row greater that total defined rows");
    if (col >= this.cols)
      throw new Error("Cannot add to col greater that total defined cols");
    this.lives[row][col] = life;
    this.rowState[row] = this.rowState[row] + 1;
  }

  delete(row: number, col: number) {
    if (row >= this.rows)
      throw new Error("Cannot add to row greater that total defined rows");
    if (col >= this.cols)
      throw new Error("Cannot add to col greater that total defined cols");
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
          grid.add(new Life(1), left + i, top + j);
        }
      }
    }
  }
}

@Component
export default class Flat extends Vue {
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
    this.grid = new Grid(50, 50);
    this.patterns = [];

    this.patterns.push(
      new Pattern("block", [
        [1, 1],
        [1, 1]
      ])
    );
    this.patterns.push(
      new Pattern("beehive", [
        [0, 1, 1, 0],
        [1, 0, 0, 1],
        [0, 1, 1, 0]
      ])
    );
    this.patterns.push(
      new Pattern("loaf", [
        [0, 1, 1, 0],
        [1, 0, 0, 1],
        [0, 1, 0, 1],
        [0, 0, 1, 0]
      ])
    );
    this.patterns.push(
      new Pattern("boat", [
        [1, 1, 0, 0],
        [1, 0, 1, 0],
        [0, 1, 0, 0]
      ])
    );
    this.patterns.push(
      new Pattern("tub", [
        [0, 1, 0],
        [1, 0, 1],
        [0, 1, 0]
      ])
    );
    this.patterns.push(new Pattern("blinker", [[1, 1, 1]]));
    this.patterns.push(
      new Pattern("toad", [
        [0, 1, 1, 1],
        [1, 1, 1, 0]
      ])
    );
    this.patterns.push(
      new Pattern("toad", [
        [0, 1, 1, 1],
        [1, 1, 1, 0]
      ])
    );
    this.patterns.push(
      new Pattern("glider", [
        [0, 1, 0],
        [0, 0, 1],
        [1, 1, 1]
      ])
    );
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
    this.started = false;
    const rect = e.target.getBoundingClientRect();
    const x = e.clientX - rect.left; //x position within the element.
    const y = e.clientY - rect.top; //y position within the element.
    this.renderer.addPattern(
      this.grid,
      this.updater,
      x,
      y,
      this.patterns[this.patterns.length - 1]
    );
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
