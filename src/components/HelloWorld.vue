<template>
    <div class="grid">
        <canvas
            v-on:click.stop="setCell()"
            id="canvas"
            width="420px"
            height="420px"
            style="background: #fff; margin:20px"
        ></canvas>
        <button v-show="!started" v-on:click.stop="started = true">
            Start
        </button>
        <button v-show="started" v-on:click.stop="started = false">Stop</button>
        <p>
            The universe of the Game of Life is an infinite, two-dimensional
            orthogonal grid of square cells, each of which is in one of two
            possible states, live or dead, (or populated and unpopulated,
            respectively). Every cell interacts with its eight neighbours, which
            are the cells that are horizontally, vertically, or diagonally
            adjacent. At each step in time, the following transitions occur:
        </p>
        <p>
            Any live cell with fewer than two live neighbours dies, as if by
            underpopulation. Any live cell with two or three live neighbours
            lives on to the next generation. Any live cell with more than three
            live neighbours dies, as if by overpopulation. Any dead cell with
            exactly three live neighbours becomes a live cell, as if by
            reproduction.
        </p>
        These rules, which compare the behavior of the automaton to real life,
        can be condensed into the following:

        <p>
            Any live cell with two or three live neighbours survives. Any dead
            cell with three live neighbours becomes a live cell. All other live
            cells die in the next generation. Similarly, all other dead cells
            stay dead. The initial pattern constitutes the seed of the system.
            The first generation is created by applying the above rules
            simultaneously to every cell in the seed; births and deaths occur
            simultaneously, and the discrete moment at which this happens is
            sometimes called a tick. Each generation is a pure function of the
            preceding one. The rules continue to be applied repeatedly to create
            further generations.
        </p>
    </div>
</template>

<script lang="ts">
import { Component, Prop, Vue } from "vue-property-decorator";

class Grid {
    render(context: any) {
        console.log("done");
        // Box width
        const bw = 400;
        // Box height
        const bh = 400;
        // Padding
        const p = 10;
        for (let x = 0; x <= bw; x += 10) {
            context.moveTo(0.5 + x + p, p);
            context.lineTo(0.5 + x + p, bh + p);
        }
        for (let x = 0; x <= bh; x += 10) {
            context.moveTo(p, 0.5 + x + p);
            context.lineTo(bw + p, 0.5 + x + p);
        }
        context.strokeStyle = "black";
        context.stroke();
    }
}

@Component
export default class HelloWorld extends Vue {
    private started = true;

    mounted() {
        const canvas = document.getElementById("canvas") as HTMLCanvasElement;
        const context = canvas.getContext("2d");
        const grid = new Grid();
        grid.render(context);
    }

    draw() {
        console.log("draw");
        window.requestAnimationFrame(this.draw);
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
