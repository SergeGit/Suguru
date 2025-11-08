## 1. Project Specification

### 1.1 System Architecture

#### Web Application Stack
```
┌─────────────────────────────────────┐
│         User Interface              │
│  (HTML + Tailwind CSS + JavaScript) │
├─────────────────────────────────────┤
│         Core Modules                │
│  - Parser Module                    │
│  - Validator Module                 │
│  - Solver Module (Backtracking)     │
│  - Renderer Module                  │
│  - Camera Module (v1.1)             │
│  - Image Processing Module (v1.1)   │
└─────────────────────────────────────┘
```

#### Android Application Stack
```
┌─────────────────────────────────────┐
│      UI Layer (Activities/Fragments)│
├─────────────────────────────────────┤
│      ViewModel Layer (MVVM)         │
├─────────────────────────────────────┤
│      Business Logic Layer           │
│  - Puzzle Repository                │
│  - Solver Engine (Kotlin/Java)      │
│  - Camera Manager                   │
│  - Image Processor                  │
├─────────────────────────────────────┤
│      Data Layer                     │
│  - Room Database (puzzle history)   │
│  - SharedPreferences (settings)     │
└─────────────────────────────────────┘
```

### 1.2 Data Structures

#### Puzzle String Format (v1.0)
```
Format: [W][H][CAGE_DATA][VALUE_DATA]

Example: "451112341223415264152203040040001005035062"
- W = 4 (width)
- H = 5 (height)
- CAGE_DATA (20 chars): Column-major cage IDs
- VALUE_DATA (20 chars): Column-major cell values (0 = empty)
```

#### Internal Puzzle Representation
```javascript
puzzleState = {
  width: number,
  height: number,
  grid: number[][],           // Cell values (0 = empty)
  cageGrid: number[][],       // Cage IDs
  initialPuzzle: number[][],  // Original state for reset
  cages: {                    // Cage metadata
    [cageId]: {
      size: number,
      cells: [{r: number, c: number}]
    }
  }
}
```

#### Camera Capture Data (v1.1+)
```javascript
captureData = {
  originalImage: ImageData,
  processedImage: ImageData,
  gridCorners: [{x, y}, {x, y}, {x, y}, {x, y}],
  detectedCells: [{
    position: {row, col},
    value: number | null,
    confidence: number,
    cageId: number
  }],
  cageBoundaries: [{
    cageId: number,
    cells: [{row, col}]
  }]
}
```

### 1.3 Module Specifications

#### 1.3.1 Parser Module
**Responsibility:** Convert string format to internal representation

**Functions:**
- `parsePuzzleString(str: string): PuzzleState`
  - Input: Puzzle string
  - Output: PuzzleState object
  - Validates: String length, dimension consistency
  - Throws: Error on invalid format

- `exportPuzzleString(state: PuzzleState): string`
  - Input: PuzzleState object
  - Output: String representation
  - Use case: Sharing puzzles

#### 1.3.2 Validator Module
**Responsibility:** Verify puzzle validity and constraints

**Functions:**
- `validatePuzzle(state: PuzzleState): ValidationResult`
  - Checks: Cage size consistency, rule violations
  - Returns: {valid: boolean, errors: string[]}

- `validateCellValue(row, col, value, state): boolean`
  - Checks: Adjacent cells, cage uniqueness
  - Returns: true if placement is valid

- `detectConflicts(state: PuzzleState): Conflict[]`
  - Finds: All rule violations in current state
  - Returns: Array of conflict descriptions

#### 1.3.3 Solver Module
**Responsibility:** Find puzzle solutions using backtracking

**Core Algorithm:**
```python
# Conceptual pseudocode (actual implementation in JavaScript)
def backtrack(puzzle):
    cell = find_empty_cell()
    if cell is None:
        return True  # Solved
    
    cage = get_cage(cell)
    for num in range(1, cage.size + 1):
        if is_valid(cell, num):
            place(cell, num)
            if backtrack(puzzle):
                return True
            remove(cell)  # Backtrack
    
    return False
```

**Functions:**
- `backtrackingSolver(state: PuzzleState): boolean`
- `findEmptyCell(): {r, c} | null`
- `isValidPlacement(r, c, num, cage): boolean`

**Optimizations (Future):**
- Constraint propagation (naked singles, hidden singles)
- Most constrained variable heuristic
- Forward checking

#### 1.3.4 Renderer Module
**Responsibility:** Display puzzle grid with cage boundaries

**Functions:**
- `renderGrid(state: PuzzleState, container: HTMLElement)`
  - Creates: Grid cells with proper styling
  - Applies: Cage borders, value classes
  
- `updateCell(row, col, value, isInitial: boolean)`
  - Updates: Single cell appearance
  
- `highlightCell(row, col, color: string)`
  - Use case: Show hints, conflicts

#### 1.3.5 Camera Module (v1.1)
**Responsibility:** Capture and prepare images for processing

**Functions:**
- `initializeCamera(): Promise<MediaStream>`
- `captureFrame(): ImageData`
- `releaseCamera()`

**Requirements:**
- Request camera permissions
- Support front/back camera selection
- Provide real-time preview
- Handle permission denials gracefully

#### 1.3.6 Image Processing Module (v1.1)
**Responsibility:** Extract puzzle data from camera images

**Pipeline Stages:**

1. **Preprocessing**
   - `grayscaleConversion(image): ImageData`
   - `adaptiveThreshold(image): ImageData`
   - `gaussianBlur(image, kernelSize): ImageData`
   - `perspectiveCorrection(image, corners): ImageData`

2. **Grid Detection**
   - `detectGridLines(image): Line[]`
   - `findGridIntersections(lines): Point[]`
   - `constructGrid(intersections): Grid`

3. **Cage Boundary Detection**
   - `detectCageBoundaries(image, grid): CageBoundary[]`
   - `groupCellsIntoCages(boundaries): Cage[]`
   - Methods: Flood fill, connected components

4. **OCR (Number Recognition)**
   - `extractCellRegions(image, grid): ImageData[]`
   - `recognizeDigit(cellImage): {digit: number, confidence: number}`
   - Libraries: Tesseract.js (web), ML Kit (Android)

5. **Post-processing**
   - `filterLowConfidenceCells(cells, threshold): Cell[]`
   - `manualReviewRequired(cells): Cell[]`

**Image Preprocessing Techniques:**
- **Edge Detection:** Canny, Sobel for line detection
- **Perspective Correction:** Homography transformation for angled photos
- **Adaptive Thresholding:** Handles uneven lighting
- **Morphological Operations:** Erosion/dilation for noise removal
- **Contrast Enhancement:** CLAHE (Contrast Limited Adaptive Histogram Equalization)
- **Sharpening:** Unsharp mask for clearer digits

#### 1.3.7 Hint System Module
**Responsibility:** Provide intelligent solving assistance

**Functions:**
- `getSingleHint(state: PuzzleState): Hint | null`
  - Strategy: Find cell with fewest valid options
  - Returns: {row, col, value, reason: string}

- `getMultipleHints(state: PuzzleState, count): Hint[]`
  - Progressive difficulty: Easy cells first

### 1.4 User Interface Specifications

#### 1.4.1 Web UI Components (v1.1)

**Main Interface:**
- Input method selector (String / Camera)
- Puzzle string input field
- Camera capture button with preview
- Grid display area (responsive, centered)
- Control buttons (Load / Solve / Reset / Hint)
- Status message area
- Manual edit toggle

**Camera Capture Interface:**
- Live camera feed
- Grid overlay (guide for alignment)
- Capture button
- Detected cells preview (editable)
- Confirm/Retry buttons
- Manual correction tools

**Grid Interaction:**
- Click cell to edit value (in edit mode)
- Right-click to clear cell
- Keyboard shortcuts (1-9 for values, 0 for clear)
- Hover highlights cage
- Visual feedback for conflicts

#### 1.4.2 Android UI Components (v2.0)

**Main Activity:**
- Bottom navigation (Solve / History / Settings)
- FAB (Floating Action Button) for new puzzle
- Recent puzzles list

**Puzzle Input Activity:**
- Camera view with overlay
- Manual string input option
- Gallery import option

**Solver Activity:**
- Grid view (touch-optimized)
- Tool palette (Edit / Hint / Solve / Reset)
- Solution steps (optional playback)

**Settings:**
- Camera settings (resolution, focus mode)
- Solver preferences (timeout, hint behavior)
- Theme selection
- About/Help

### 1.5 API Specifications (Internal)

#### Solver API
```javascript
class SuguruSolver {
  constructor(puzzleState)
  solve(): boolean
  getSolution(): PuzzleState
  getStepByStep(): Step[]
  cancel()
}
```

#### Image Processor API
```javascript
class ImageProcessor {
  constructor(image: ImageData)
  detectGrid(): Promise<GridInfo>
  detectCages(): Promise<CageInfo[]>
  recognizeNumbers(): Promise<CellData[]>
  getPuzzleState(): Promise<PuzzleState>
}
```

#### Validator API
```javascript
class PuzzleValidator {
  static validate(state: PuzzleState): ValidationResult
  static checkCell(row, col, value, state): boolean
  static findConflicts(state): Conflict[]
}
```

---