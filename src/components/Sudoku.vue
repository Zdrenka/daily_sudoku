<template>
  <div class="hello">
    <h1>{{ msg }}</h1>
    <h1>Daily Sudoku</h1>
    <hr />
    <div>
      Time: <span id="timer">{{ timer }}</span>
    </div>
    <hr />
    <div>
      <label for="difficulty-toggle">Difficulty:</label>
      <select id="difficulty-toggle" v-model="selectedDifficulty">
        <option value="simple">Simple</option>
        <option value="Medium">Medium</option>
        <option value="hard">Hard</option>
        <option value="expert">Expert</option>
      </select>
      <button @click="startGame">Start Game</button>
    </div>
    <h2 :class="{ 'error-display': errors > 0 }">{{ errors }}</h2>
    <!-- 9x9 board -->
    <div id="board">
      <div v-for="(row, rowIndex) in board" :key="rowIndex">
        <div
          v-for="(cell, colIndex) in row"
          :key="colIndex"
          :id="rowIndex + '-' + colIndex"
          :class="[
            'tile',
            {
              'tile-wrong': tileWrong,
              'vertical-line': line(rowIndex),
              'horizontal-line': line(colIndex),
              'tile-start': cell !== '-',
            },
          ]"
          @click="selectTile($event)"
        >
          {{ cell === "-" ? "" : cell }}
        </div>
      </div>
    </div>
    <br />
    <div id="digits">
      <div
        v-for="num in [1, 2, 3, 4, 5, 6, 7, 8, 9]"
        :key="num"
        :id="num"
        :class="['number']"
        @click="selectNumber($event)"
      >
        {{ num }}
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { Options, Vue } from "vue-class-component";

@Options({
  props: {
    msg: String,
  },
})
export default class HelloWorld extends Vue {
  msg!: string;
  numSelected: HTMLElement | null = null;
  tileSelected: HTMLElement | null = null;

  errors = 0;
  board: Array<Array<string>> = [];
  solution: Array<Array<string>> = [];
  selectedDifficulty = "simple";
  startTime: number | null = null;
  elapsedTime = 0;
  timerInterval: number | null = null;
  timer = "00:00";

  //css classes
  tileWrong = "";

  //

  Difficulty: { [key: string]: number } = {
    SIMPLE: 54,
    MEDIUM: 41,
    HARD: 33,
    EXPERT: 25,
  };

  line(ent: number): boolean {
    return ent === 2 || ent === 5;
  }

  startGame(): void {
    this.resetBoard();
    this.generateSolution();
    this.generatePuzzle(this.solution, this.selectedDifficulty);
    this.startTimer();
  }

  resetBoard(): void {
    this.errors = 0;
    this.board = [];
    this.timer = "00:00";
    this.elapsedTime = 0;
  }

  /*Timer*/
  startTimer(): void {
    this.startTime = Date.now() - this.elapsedTime;
    this.timerInterval = setInterval(this.updateTimer, 1000);
    document.addEventListener("visibilitychange", this.handleVisibilityChange);
  }

  stopTimer(): void {
    clearInterval(this.timerInterval ?? 0);
    document.removeEventListener(
      "visibilitychange",
      this.handleVisibilityChange
    );
  }

  updateTimer(): void {
    let currentTime: number = Date.now();
    this.elapsedTime = currentTime - (this.startTime ?? 0);
    let minutes: number = Math.floor(this.elapsedTime / 60000);
    let seconds: number = Math.floor((this.elapsedTime % 60000) / 1000);
    this.timer = `${minutes.toString().padStart(2, "0")}:${seconds
      .toString()
      .padStart(2, "0")}`;
  }

  handleVisibilityChange(): void {
    if (document.hidden) {
      this.stopTimer();
    } else {
      this.startTime = Date.now() - this.elapsedTime;
      this.timerInterval = setInterval(this.updateTimer, 1000);
    }
  }
  /*Initalise game*/
  generateSolution(): Array<string> {
    // Initialize an empty 9x9 board
    if (this.solution) {
      this.solution = [];
    }
    this.solution = Array.from({ length: 9 }, () =>
      Array.from({ length: 9 }, () => "0")
    );

    let firstRow: Array<number> = this.shuffle([1, 2, 3, 4, 5, 6, 7, 8, 9]);
    for (let i = 0; i < 9; i++) {
      this.solution[0][i] = firstRow[i].toString();
    }
    // Call the recursive helper function to fill the board
    if (this.solveSudoku(this.solution, 0, 0)) {
      return this.solution.map((row: Array<string>) => row.join(""));
    } else {
      throw new Error("No solution found");
    }
  }

  generatePuzzle(
    solution: Array<Array<string>>,
    difficulty: string
  ): Array<string> {
    if (this.board) {
      this.board = [];
    }
    this.board = JSON.parse(JSON.stringify(solution));

    // Remove cells until the desired difficulty level is reached
    let numClues: number = 81 - this.Difficulty[difficulty.toUpperCase()];
    while (numClues > 0) {
      let row: number = Math.floor(Math.random() * 9);
      let col: number = Math.floor(Math.random() * 9);
      if (row % 3 === 0) {
        // rowIndex + "-" + colIndex;
      }
      if (this.board[row][col] != "-") {
        this.board[row][col] = "-";
        numClues--;
      }
    }

    return this.board.map((row: Array<string>) => row.join(""));
  }
  /*end*/

  solveSudoku(board: Array<Array<string>>, row: number, col: number): boolean {
    // If we've reached the end of the board, we're done
    if (row == 9) {
      return true;
    }

    // Calculate the next row and column
    let nextRow: number = col == 8 ? row + 1 : row;
    let nextCol: number = col == 8 ? 0 : col + 1;

    // If the current cell is already filled, move on to the next cell
    if (board[row][col] != "0") {
      return this.solveSudoku(board, nextRow, nextCol);
    }

    // Try each possible value for the current cell
    for (let num = 1; num <= 9; num++) {
      if (this.isSafe(board, row, col, num.toString())) {
        board[row][col] = num.toString();
        if (this.solveSudoku(board, nextRow, nextCol)) {
          return true;
        }
        board[row][col] = "0";
      }
    }

    // If none of the values work, backtrack
    return false;
  }

  shuffle(array: Array<number>): Array<number> {
    // Fisher-Yates shuffle algorithm
    for (let i: number = array.length - 1; i > 0; i--) {
      let j: number = Math.floor(Math.random() * (i + 1));
      [array[i], array[j]] = [array[j], array[i]];
    }
    return array;
  }

  isSafe(
    board: Array<Array<string>>,
    row: number,
    col: number,
    num: string
  ): boolean {
    // Check if the number is already in the same row or column
    for (let i = 0; i < 9; i++) {
      if (board[row][i] == num || board[i][col] == num) {
        return false;
      }
    }

    // Check if the number is already in the same 3x3 box
    let boxRow: number = Math.floor(row / 3) * 3;
    let boxCol: number = Math.floor(col / 3) * 3;
    for (let i: number = boxRow; i < boxRow + 3; i++) {
      for (let j: number = boxCol; j < boxCol + 3; j++) {
        if (board[i][j] == num) {
          return false;
        }
      }
    }

    // If the number is safe, return true
    return true;
  }

  /*user selection*/
  selectNumber(event: MouseEvent): void {
    if (this.numSelected != null) {
      this.numSelected.classList.remove("number-selected");
    }
    this.numSelected = event.target as HTMLElement;
    this.numSelected?.classList.add("number-selected");
  }

  selectTile(event: MouseEvent): void {
    this.tileSelected = event.target as HTMLElement;

    this.tileSelected.textContent = this.numSelected?.textContent ?? null;
    let coords: Array<string> = this.tileSelected.id.split("-"); // [0, 0]
    let r: number = parseInt(coords[0]);
    let c: number = parseInt(coords[1]);

    if (
      this.numSelected &&
      this.solution[r][c] != this.numSelected?.textContent
    ) {
      this.errors += 1;
      this.tileSelected.classList.add("tile-wrong");
    } else {
      this.board[r][c] = this.tileSelected.textContent ?? "";
      this.tileSelected.classList.remove("tile-wrong");
    }
    if (this.isComplete()) {
      this.stopTimer();
      alert("Complete! time: " + this.timer);
    }
  }

  isComplete() {
    if (this.board.length !== this.solution.length) {
      return false;
    }
    for (let i = 0; i < this.board.length; i++) {
      for (let j = 0; j < this.board[i].length; j++) {
        if (this.board[i][j] !== this.solution[i][j]) {
          return false;
        }
      }
    }
    return true;
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
hr {
  width: 500px;
}

#errors {
  color: red;
}

#board {
  width: 450px;
  height: 450px;
  margin: 0 auto;
  display: flex;
  flex-wrap: wrap;
}

#digits {
  width: 450px;
  height: 50px;
  margin: 0 auto;
  display: flex;
  flex-wrap: wrap;
}

.tile {
  width: 48px;
  height: 48px;
  border: 1px solid lightgray;
  font-size: 20px;
  font-weight: bold;
  display: flex;
  justify-content: center;
  align-items: center;
}

.number {
  width: 44px;
  height: 44px;
  border: 1px solid black;
  margin: 2px;
  font-size: 20px;
  font-weight: bold;
  display: flex;
  justify-content: center;
  align-items: center;
}

.number-selected {
  background-color: gray;
}

.tile-start {
  background-color: whitesmoke;
}

.horizontal-line {
  border-bottom: 1px solid black;
}

.vertical-line {
  border-right: 1px solid black;
}

.tile-wrong {
  color: lightcoral;
}
</style>
