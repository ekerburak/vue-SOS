<template>
    <div class="grid-container" :style="{ 'grid-template': gridTemplateString }">
        <!--cell element is set to input to trigger mobile keyboard on click-->
        <input type="text" class="cell" v-for="item in flattenedGrid" :row="item.row" :column="item.column" tabindex="-1"
            @input="fillCell" :style="{ 'font-size': cellFontSizeString }">
    </div>
    <Canvas :gridWidth="gridWidth" :gridHeight="gridHeight" :gapWidth="gapWidth" :cellWidth="cellWidth" :lines="lines"
        :lineCount="lines.length" :maxCellWidth="maxCellWidth" />
</template>

<script>
import Canvas from "./Canvas.vue";
export default {
    data() {
        return {
            flattenedGrid: [],
            filledCellCount: 0,
            gapWidth: 3,
            canvasContext: undefined,
            maxCellWidth: (500 - 2 * 3) / 3,
            lines: []
        };
    },
    props: ["rowCount", "columnCount", "windowWidth", "currentPlayerColor"],
    components: { Canvas },
    emits: ["endTurn", "addScore", "endGame"],
    methods: {
        getFlattenedIndex(row, column) {
            return row * this.columnCount + column;
        },
        fillCell(event) {
            if (event.target.value != "S" && event.target.value != "O" && event.target.value != "s" && event.target.value != "o") {
                event.target.value = "";
                return;
            }
            //once a valid letter is taken, make it readonly to prevent further unsolicited input events 
            event.target.setAttribute("readOnly", "");
            var row = Number(event.target.getAttribute("row"));
            var column = Number(event.target.getAttribute("column"));
            this.flattenedGrid[this.getFlattenedIndex(row, column)].letter = event.target.value.toUpperCase();
            var score = this.countScore(row, column);
            //player made a move with no SOS triplets formed
            if (score === 0)
                this.$emit("endTurn");
            else
                this.$emit("addScore", score);
            this.filledCellCount++;
            if (this.filledCellCount == this.rowCount * this.columnCount)
                this.$emit("endGame");
        },
        isCellValid(row, column) {
            return row >= 0 && row < this.rowCount &&
                column >= 0 && column < this.columnCount;
        },
        countScore(row, column) {
            //for shorter name
            var flatten = this.getFlattenedIndex;
            var letter = this.flattenedGrid[flatten(row, column)].letter;
            var letters = ["S", "O"];
            var score = 0;
            var countScoreUtil = (op1, op2) => {
                for (let i = 0; i < 3; i++) {
                    if (letter !== letters[i % 2])
                        continue;
                    let triplet = true;
                    for (let j = 0; j < 3; j++) {
                        if (!this.isCellValid(op1(row, j - i), op2(column, j - i))) {
                            triplet = false;
                            break;
                        }
                        let shouldMatch = !((j - i + 2) % 2);
                        let currentLetter = this.flattenedGrid[flatten(op1(row, j - i), op2(column, j - i))].letter;
                        if (currentLetter === "") {
                            triplet = false;
                            break;
                        }
                        if (shouldMatch && currentLetter !== letter ||
                            !shouldMatch && currentLetter === letter) {
                            triplet = false;
                            break;
                        }
                    }
                    if (!triplet)
                        continue;
                    score += triplet;
                    let line = { coordinates: [], color: this.currentPlayerColor };
                    for (let j = 0; j < 3; j++)
                        line.coordinates.push({ row: op1(row, j - i), column: op2(column, j - i) });
                    this.lines.push(line);
                }
            };
            //same row, iterate column
            countScoreUtil((a) => a, (a, b) => a + b);
            //same column, iterate row
            countScoreUtil((a, b) => a + b, (a) => a);
            //diagonal \
            countScoreUtil((a, b) => a + b, (a, b) => a + b);
            //diagonal /
            countScoreUtil((a, b) => a + b, (a, b) => a - b);
            return score;
        },
        initGrid() {
            this.lines = [];
            this.flattenedGrid = [];
            for (let i = 0; i < this.rowCount; i++)
                for (let j = 0; j < this.columnCount; j++)
                    this.flattenedGrid.push({ row: i, column: j, letter: "" });
            //no vue-style way to remove an attribute altogether from elements in DOM
            var cells = document.querySelectorAll(".grid-container > input");
            cells.forEach((cell) => {
                cell.removeAttribute("readOnly"),
                    cell.value = "";
            });
            this.filledCellCount = 0;
        }
    },
    watch: {
        rowCount: {
            handler() {
                this.initGrid();
            },
            post: "flush" //initGrid manipulates DOM
        },
        columnCount: {
            handler() {
                this.initGrid();
            },
            post: "flush" //initGrid manipulates DOM
        }
    },
    mounted() {
        this.initGrid();
    },
    computed: {
        gridTemplateString() {
            return "repeat(" + this.rowCount + ", 1fr) / " + "repeat(" + this.columnCount + ", 1fr)";
        },
        cellFontSizeString() {
            return (Math.floor((6 * (this.cellWidth / this.maxCellWidth)) * 100) / 100).toString() + "rem";
        },
        gridWidth() {
            //min(90vw, 500px)
            return Math.min(this.windowWidth * 0.9, 500);
        },
        gridHeight() {
            return this.cellWidth * this.rowCount + this.gapWidth * (this.rowCount - 1);
        },
        cellWidth() {
            return (this.gridWidth - (this.columnCount - 1) * this.gapWidth) / this.columnCount;
        }
    }
}
</script>

<style scoped>
.grid-container {
    display: grid;
    row-gap: 3px;
    column-gap: 3px;
    margin: 10px auto;
    width: 90vw;
    max-width: 500px;
}

.cell {
    text-align: center;
    width: 100%;
    padding: 0;
    aspect-ratio: 1;
    background-color: hsl(196, 27%, 44%);
    user-select: none;
    font-family: Nunito, sans-serif;
    font-weight: 800;
    color: whitesmoke;
    border-radius: 4%;
    caret-color: transparent;
    /*to hide caret */
    border-style: none;
}

.cell:focus {
    outline-style: none;
    background-color: hsl(196, 27%, 34%);
}
</style>