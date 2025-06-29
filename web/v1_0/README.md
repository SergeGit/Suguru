# Suguru Puzzle Solver

This is a web-based Suguru puzzle solver built with HTML, CSS (Tailwind CSS), and JavaScript. It allows a user to input a puzzle defined by a specific string format, visualize it on a grid, and find the solution using a backtracking algorithm.

## Features

* **Dynamic Puzzle Loading**: Load any Suguru puzzle by providing a compact string that defines its dimensions, cage layout, and initial values.
* **Visual Grid Rendering**: Displays the puzzle on a clear, interactive grid with distinct borders for each cage.
* **One-Click Solver**: Solves the puzzle using a backtracking algorithm that respects all Suguru rules.
* **Reset Functionality**: Allows the user to reset the puzzle to its initial state after solving.
* **Responsive UI**: The interface is designed to work smoothly on different screen sizes.

## How to Use

1. **Open the `Suguru.html` file** in any modern web browser.
2. The application will automatically load a default puzzle string.
3. To load a different puzzle, **paste your puzzle string** into the input field.
4. Click the **"Load Puzzle"** button. The grid will be rendered visually.
5. Click the **"Solve"** button to find and display the solution. The newly placed numbers will appear in blue.
6. Click the **"Reset"** button to clear the solution and return the grid to its initial state.

## Puzzle String Format

The application uses a specific string format to define puzzles. The string is a concatenation of three parts:

`[Dimensions][Cage Data][Value Data]`

**Example String:** `451112341223415264152203040040001005035062`

1. **Dimensions (2 characters):**
   * The first character is the **width** of the grid.
   * The second character is the **height** of the grid.
   * *Example*: `45` -> 4x5 grid.

2. **Cage Data (`width` * `height` characters):**
   * This section defines the cage for each cell.
   * The data is read **column by column**. Each character is an ID for the cage that the cell belongs to.
   * *Example*: For a 4x5 grid, the first 20 characters after the dimensions define the cages.

3. **Value Data (`width` * `height` characters):**
   * This section defines the initial numbers in the grid.
   * This data is also read **column by column**.
   * A `0` represents an empty cell. Any other digit is a pre-filled value.

## Code Breakdown

The entire application is self-contained within a single `index.html` file.

### HTML Structure

* The body contains a main container styled with Tailwind CSS.
* `<input type="text" id="puzzleString">`: Field for the puzzle string.
* `<button>` elements (`#loadBtn`, `#solveBtn`, `#resetBtn`): User controls.
* `<div id="grid-container">`: The container where the JavaScript dynamically generates the puzzle grid.
* `<div id="message-box">`: Displays status messages (e.g., "Puzzle Solved!", "Error").

### CSS Styling

* **Tailwind CSS** is used for the overall layout and styling.
* **Custom CSS** in the `<style>` tag is used for specific grid details:
  * `.grid-cell`: Defines the size and font properties of each cell.
  * `.initial-val` & `.solved-val`: Differentiates between pre-filled numbers and solved numbers.
  * `.border-*-cage`: Defines the thick borders used to outline the cages.

### JavaScript Logic

The core logic is within the `<script>` tag.

* **Global State (`puzzleState`)**: A single object holds the grid's `width`, `height`, the current `grid` values, the `cageGrid` layout, the `cages` object (processed for easier access), and the `initialPuzzle` state for the reset functionality.
* `parsePuzzleString(str)`: Reads the input string, validates it, and populates the `puzzleState` object. It reads the cage and value data in a column-major order to match the string format.
* `renderGrid()`: Clears the existing grid and redraws it based on the current `puzzleState`. It dynamically adds CSS classes for values and cage borders.
* `backtrackingSolver()`: The recursive algorithm that solves the puzzle.
  1. It finds the next empty cell (`findEmptyCell`).
  2. It iterates through possible numbers (from 1 to the size of the cell's cage).
  3. For each number, it checks if the placement is valid (`isValidPlacement`).
  4. If valid, it places the number and recursively calls itself.
  5. If the recursive call fails, it "backtracks" by resetting the cell to 0 and trying the next number.
* `isValidPlacement(r, c, num, cage)`: Checks if a number can be placed in a given cell based on Suguru rules:
  1. The number is not present in any orthogonally or diagonally adjacent cell.
  2. The number is not already present within the same cage.