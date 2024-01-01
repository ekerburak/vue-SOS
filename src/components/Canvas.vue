<template>
    <!--utilize the negative margin trick to place the canvas precisely on top of the grid-->
    <canvas :width="gridWidth" :height="gridHeight"
        :style="{ width: gridWidth + 'px', height: gridHeight + 'px', marginTop: '-' + gridHeight + 'px' }"
        ref="canvas"></canvas>
</template>

<script>
export default {
    //lines is an array storing lines with respect to its endpoints in (row, column) format
    //'lines' also stores the corresponding line color 
    //lineCount is nothing but lines.length. it was added as a prop to be able to watch changes to it
    //i.e. when new lines are added to the array 'lines'
    //watching the array 'lines' itself is problematic due to grid reinitialization   
    props: ["gridWidth", "gapWidth", "cellWidth", "gridHeight", "lines", "lineCount", "maxCellWidth"],
    methods: {
        getPixelPosition(row, column) {
            var x = Math.floor(column * this.gapWidth + (column + 0.5) * this.cellWidth);
            var y = Math.floor(row * this.gapWidth + (row + 0.5) * this.cellWidth);
            return { x, y };
        },
        drawLine(line) {
            //context needs to be get every time due to width and height changes in the canvas
            var context = this.$refs.canvas.getContext("2d");
            context.lineWidth = 20 * this.cellWidth / this.maxCellWidth;
            context.beginPath();
            for (let i = 0; i < 3; i++) {
                let { x, y } = this.getPixelPosition(line.coordinates[i].row, line.coordinates[i].column);
                if (i == 0)
                    context.moveTo(x, y);
                else
                    context.lineTo(x, y);
            }
            context.globalAlpha = 0.5;
            context.lineCap = "round";
            context.strokeStyle = line.color;
            context.stroke();
        },
        drawAll() {
            for (let i = 0; i < this.lineCount; i++)
                this.drawLine(this.lines[i]);
        }
    },
    watch: {
        gridWidth: { //only changes when viewport width changes
            handler() {
                //no need to clear canvas as it handles that itself when width and height changes
                this.drawAll();
            },
            //drawAll calls drawLine, which in turn accesses the context
            //since it is a direct dom access, we set flush to post 
            flush: "post" //see vuejs guide
        },
        lineCount: {
            handler(newValue, oldValue) {
                for (let i = oldValue; i < newValue; i++)
                    this.drawLine(this.lines[i]);
            },
            //drawAll calls drawLine, which in turn accesses the context
            //since it is a direct dom access, we set flush to post
            flush: "post"
        }
    }
};
</script>

<style scoped>
canvas {
    display: block;
    margin: auto;
    pointer-events: none;
    /*for grid to catch input*/
}
</style>