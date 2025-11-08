# Appendix

## Appendix A: Suguru Rules Reference

For users who need a refresher on Suguru puzzle rules:

### Basic Rules
1. **Grid Structure**: The puzzle consists of a rectangular grid divided into irregular regions called "cages"
2. **Number Placement**: Each cage of N cells must contain the numbers 1 through N exactly once
3. **Adjacency Constraint**: No two adjacent cells (including diagonally adjacent) can contain the same number, even if they belong to different cages

### Example
```
┌───┬───────┬───┐
│ 2 │ 1 │ 3 │ 2 │
├───┤   ├───┼───┤
│ 1 │   │ 2 │ 1 │
├───┼───┴───┤   │
│ 3 │ 1 │ 2 │   │
└───┴───────┴───┘

Cage sizes in this puzzle:
- Top-left cage: 2 cells (contains 1, 2)
- Middle-top cage: 3 cells (contains 1, 2, 3)
- Top-right cage: 2 cells (contains 1, 2)
- Bottom-left cage: 1 cell (contains 3)
- Bottom-middle cage: 2 cells (contains 1, 2)
```

### Common Solving Strategies
1. **Naked Singles**: A cell where only one number is possible
2. **Hidden Singles**: A number that can only go in one cell within a cage
3. **Adjacency Elimination**: Eliminate options based on adjacent filled cells
4. **Cage Completion**: When only one number is missing from a cage

---

## Appendix B: Image Processing Algorithms

### B.1 Canny Edge Detection

**Purpose**: Detect edges in the image to find grid lines

**Algorithm Steps**:
1. Apply Gaussian blur to reduce noise
2. Calculate gradient magnitude and direction using Sobel operators
3. Apply non-maximum suppression to thin edges
4. Apply double threshold to identify strong and weak edges
5. Track edges by hysteresis (connect weak edges to strong edges)

**Parameters**:
- `low_threshold`: 50 (typical)
- `high_threshold`: 150 (typical)
- `gaussian_kernel`: 5x5

**Python Pseudocode**:
```python
def canny_edge_detection(image, low_thresh=50, high_thresh=150):
    # Step 1: Gaussian blur
    blurred = gaussian_blur(image, kernel_size=5)
    
    # Step 2: Gradient calculation
    grad_x = sobel_x(blurred)
    grad_y = sobel_y(blurred)
    magnitude = sqrt(grad_x^2 + grad_y^2)
    direction = arctan2(grad_y, grad_x)
    
    # Step 3: Non-maximum suppression
    suppressed = non_max_suppression(magnitude, direction)
    
    # Step 4 & 5: Double threshold and hysteresis
    edges = hysteresis_threshold(suppressed, low_thresh, high_thresh)
    
    return edges
```

### B.2 Hough Line Transform

**Purpose**: Detect straight lines from edge-detected image

**Algorithm**:
1. For each edge pixel, vote for all possible lines passing through it in Hough space
2. Lines are represented in polar coordinates (ρ, θ)
3. Accumulate votes in a 2D array (accumulator)
4. Extract local maxima as detected lines

**Parameters**:
- `rho_resolution`: 1 pixel
- `theta_resolution`: 1 degree (π/180 radians)
- `threshold`: Minimum votes to be considered a line
- `min_line_length`: 100 pixels
- `max_line_gap`: 10 pixels

**Python Pseudocode**:
```python
def hough_lines(edges, rho=1, theta=pi/180, threshold=100):
    accumulator = create_accumulator(edges.shape, rho, theta)
    
    for y, x in edge_pixels(edges):
        for t in range(0, 180, theta):
            r = x * cos(t) + y * sin(t)
            accumulator[r, t] += 1
    
    lines = []
    for r, t in local_maxima(accumulator, threshold):
        lines.append((r, t))
    
    return lines
```

### B.3 Perspective Transformation (Homography)

**Purpose**: Correct perspective distortion when photo is taken at an angle

**Algorithm**:
1. Identify four corner points of the grid
2. Define destination rectangle (corrected view)
3. Calculate 3x3 homography matrix H
4. Apply transformation to all pixels

**Homography Matrix Calculation**:
```
For each correspondence (x, y) → (x', y'):
x' = (h11*x + h12*y + h13) / (h31*x + h32*y + h33)
y' = (h21*x + h22*y + h23) / (h31*x + h32*y + h33)

Solve for H = [h11 h12 h13]
              [h21 h22 h23]
              [h31 h32 h33]
```

**Python Pseudocode**:
```python
def calculate_homography(src_points, dst_points):
    # src_points: 4 corner points in original image
    # dst_points: 4 corner points in corrected image
    
    A = []
    for i in range(4):
        x, y = src_points[i]
        x_prime, y_prime = dst_points[i]
        A.append([x, y, 1, 0, 0, 0, -x_prime*x, -x_prime*y, -x_prime])
        A.append([0, 0, 0, x, y, 1, -y_prime*x, -y_prime*y, -y_prime])
    
    # Solve Ah = 0 using SVD
    H = solve_svd(A)
    return H.reshape(3, 3)

def warp_perspective(image, H, width, height):
    output = create_image(width, height)
    H_inv = inverse(H)
    
    for y_dst in range(height):
        for x_dst in range(width):
            # Map destination to source
            src_point = H_inv @ [x_dst, y_dst, 1]
            x_src = src_point[0] / src_point[2]
            y_src = src_point[1] / src_point[2]
            
            # Interpolate pixel value
            output[y_dst, x_dst] = interpolate(image, x_src, y_src)
    
    return output
```

### B.4 Adaptive Thresholding

**Purpose**: Convert grayscale to binary, handling varying lighting conditions

**Algorithm**:
1. Divide image into small regions (blocks)
2. Calculate threshold for each block based on local statistics
3. Apply threshold to convert to binary (black/white)

**Methods**:
- **Mean**: Threshold = mean(block) - constant
- **Gaussian**: Threshold = weighted_mean(block) - constant

**Python Pseudocode**:
```python
def adaptive_threshold(image, block_size=11, constant=2):
    output = create_image_like(image)
    
    for y in range(image.height):
        for x in range(image.width):
            # Extract local block
            block = get_block(image, x, y, block_size)
            
            # Calculate local threshold
            threshold = mean(block) - constant
            
            # Apply threshold
            output[y, x] = 255 if image[y, x] > threshold else 0
    
    return output
```

### B.5 Connected Components (Cage Detection)

**Purpose**: Group adjacent cells into cages based on boundary lines

**Algorithm**: Flood Fill
1. Start with an unlabeled cell
2. Assign it a unique cage ID
3. Recursively label all connected cells (not separated by thick lines)
4. Repeat until all cells are labeled

**Python Pseudocode**:
```python
def flood_fill_cages(grid, boundaries):
    labeled = create_grid(grid.width, grid.height, value=-1)
    cage_id = 0
    
    for r in range(grid.height):
        for c in range(grid.width):
            if labeled[r][c] == -1:
                flood_fill(labeled, boundaries, r, c, cage_id)
                cage_id += 1
    
    return labeled

def flood_fill(labeled, boundaries, r, c, cage_id):
    if labeled[r][c] != -1:
        return
    
    labeled[r][c] = cage_id
    
    # Check 4 neighbors
    for dr, dc in [(-1,0), (1,0), (0,-1), (0,1)]:
        nr, nc = r + dr, c + dc
        if is_valid(nr, nc) and not has_boundary(boundaries, r, c, nr, nc):
            flood_fill(labeled, boundaries, nr, nc, cage_id)
```

---

## Appendix C: Performance Optimization Techniques

### C.1 Web Workers for Background Processing

**Purpose**: Keep UI responsive during heavy computation

**Implementation**:
```javascript
// main.js
const solverWorker = new Worker('solver-worker.js');

solverWorker.postMessage({
    type: 'solve',
    puzzleState: puzzleState
});

solverWorker.onmessage = function(e) {
    if (e.data.type === 'solution') {
        puzzleState = e.data.puzzleState;
        renderGrid();
    }
};

// solver-worker.js
self.onmessage = function(e) {
    if (e.data.type === 'solve') {
        const solver = new SuguruSolver(e.data.puzzleState);
        const solved = solver.solve();
        
        self.postMessage({
            type: 'solution',
            puzzleState: e.data.puzzleState,
            solved: solved
        });
    }
};
```

### C.2 Constraint Propagation (Solver Optimization)

**Technique**: Eliminate impossible values before backtracking

**Implementation**:
```python
def propagate_constraints(puzzle):
    changed = True
    while changed:
        changed = False
        
        for r, c in empty_cells(puzzle):
            valid_options = get_valid_options(r, c, puzzle)
            
            # Naked single: only one option
            if len(valid_options) == 1:
                puzzle.grid[r][c] = valid_options[0]
                changed = True
            
            # Hidden single: only cell in cage for this number
            cage = puzzle.cages[puzzle.cage_grid[r][c]]
            for num in valid_options:
                if is_only_position_in_cage(r, c, num, cage, puzzle):
                    puzzle.grid[r][c] = num
                    changed = True
                    break
```

### C.3 Most Constrained Variable (MCV) Heuristic

**Purpose**: Choose cells with fewest options first (faster convergence)

**Implementation**:
```python
def find_most_constrained_cell(puzzle):
    min_options = float('inf')
    best_cell = None
    
    for r in range(puzzle.height):
        for c in range(puzzle.width):
            if puzzle.grid[r][c] != 0:
                continue
            
            options = count_valid_options(r, c, puzzle)
            if options < min_options:
                min_options = options
                best_cell = (r, c)
    
    return best_cell
```

### C.4 Android Performance Tips

1. **Use RecyclerView efficiently**:
   - Implement ViewHolder pattern
   - Use DiffUtil for list updates
   - Set `hasFixedSize(true)` when possible

2. **Optimize bitmap handling**:
   - Downsample large images before processing
   - Use `inBitmap` for bitmap reuse
   - Process images in background threads

3. **Reduce overdraw**:
   - Remove unnecessary backgrounds
   - Use `clipRect()` in custom views
   - Enable "Debug GPU overdraw" to identify issues

4. **Database optimization**:
   - Use indexes on frequently queried columns
   - Batch operations in transactions
   - Use Flow for reactive queries

---

## Appendix D: Testing Strategy

### D.1 Test Puzzle Dataset

Create a comprehensive test dataset with:

**Easy Puzzles** (20 puzzles):
- Small grids (3×3 to 5×5)
- Regular cage shapes
- Many pre-filled cells

**Medium Puzzles** (30 puzzles):
- Medium grids (5×5 to 7×7)
- Irregular cage shapes
- Moderate pre-filled cells

**Hard Puzzles** (30 puzzles):
- Large grids (7×7 to 10×10)
- Complex cage patterns
- Few pre-filled cells

**Edge Cases** (20 puzzles):
- Puzzles with multiple solutions
- Unsolvable puzzles (for validation testing)
- Maximum size grids
- Minimum number of clues

### D.2 Image Test Dataset

**Printed Puzzles** (50 images):
- High quality scans
- Newspaper quality
- Magazine quality
- Various angles (0°-30°)
- Different lighting conditions

**Handwritten Puzzles** (50 images):
- Neat handwriting
- Messy handwriting
- Various pen types
- Partial solutions

**Challenging Conditions** (30 images):
- Low light
- Glare/reflections
- Shadows
- Wrinkled paper
- Slight blur

### D.3 Unit Test Examples

```javascript
// JavaScript/Jest
describe('SuguruSolver', () => {
    test('solves simple 3x3 puzzle', () => {
        const puzzle = parsePuzzleString('33111122233000001020');
        const solver = new SuguruSolver(puzzle);
        expect(solver.solve()).toBe(true);
        expect(puzzle.grid[0][1]).toBe(2);
    });
    
    test('detects unsolvable puzzle', () => {
        const puzzle = createUnsolvablePuzzle();
        const solver = new SuguruSolver(puzzle);
        expect(solver.solve()).toBe(false);
    });
});

// Kotlin/JUnit
class SuguruSolverTest {
    @Test
    fun `solve simple 3x3 puzzle`() {
        val puzzle = parsePuzzleString("33111122233000001020")
        val solver = SuguruSolver(puzzle)
        assertTrue(solver.solve())
        assertEquals(2, puzzle.grid[0][1])
    }
    
    @Test
    fun `detect unsolvable puzzle`() {
        val puzzle = createUnsolvablePuzzle()
        val solver = SuguruSolver(puzzle)
        assertFalse(solver.solve())
    }
}
```

### D.4 Integration Test Examples

```kotlin
// Android Espresso
@Test
fun testCameraCaptureFlow() {
    // Grant camera permission
    grantPermission(Manifest.permission.CAMERA)
    
    // Launch camera activity
    onView(withId(R.id.fab)).perform(click())
    
    // Wait for camera preview
    onView(withId(R.id.camera_preview))
        .check(matches(isDisplayed()))
    
    // Capture image
    onView(withId(R.id.capture_button)).perform(click())
    
    // Verify processing started
    onView(withText("Processing..."))
        .check(matches(isDisplayed()))
    
    // Wait for completion (with timeout)
    onView(withId(R.id.confirm_button))
        .perform(waitUntilDisplayed(timeout = 10000))
    
    // Confirm puzzle
    onView(withId(R.id.confirm_button)).perform(click())
    
    // Verify grid displayed
    onView(withId(R.id.puzzle_grid))
        .check(matches(isDisplayed()))
}
```

---

## Appendix E: Troubleshooting Common Issues

### E.1 Camera Issues

**Problem**: Camera permission denied
- **Solution**: Show explanation dialog, provide link to app settings

**Problem**: Camera not available
- **Solution**: Check if device has camera, fallback to manual input

**Problem**: Poor image quality
- **Solution**: Guide user with overlay, suggest better lighting

### E.2 OCR Issues

**Problem**: Numbers not recognized
- **Solutions**:
  - Increase contrast preprocessing
  - Adjust confidence threshold
  - Provide manual correction interface
  - Try different OCR engines (Tesseract, ML Kit, custom model)

**Problem**: Wrong numbers detected
- **Solutions**:
  - Show confidence scores
  - Highlight low-confidence cells
  - Enable easy manual correction

### E.3 Grid Detection Issues

**Problem**: Grid lines not detected
- **Solutions**:
  - Adjust Canny thresholds
  - Enhance line contrast
  - Manual grid corner selection fallback

**Problem**: Cage boundaries unclear
- **Solutions**:
  - Use thickness-based detection
  - Combine multiple detection methods
  - Allow manual cage editing

### E.4 Solver Issues

**Problem**: Solver times out
- **Solutions**:
  - Implement constraint propagation
  - Use MCV heuristic
  - Set reasonable timeout (30s)
  - Show progress indicator

**Problem**: No solution found for valid puzzle
- **Solutions**:
  - Verify input data correctness
  - Check for bugs in validation logic
  - Test with known-good puzzles

---

## Appendix F: Future Enhancements

### Potential Features for v1.2 and Beyond

1. **Step-by-Step Solution Playback**
   - Show solving process
   - Explain each step
   - Educational value

2. **Puzzle Generation**
   - Create random valid puzzles
   - Difficulty levels
   - Share generated puzzles

3. **Multiplayer Mode**
   - Competitive solving
   - Leaderboards
   - Time challenges

4. **Advanced Solving Techniques**
   - X-Wing, Swordfish patterns
   - Constraint programming
   - SAT solver integration

5. **AR Mode (Android)**
   - Point camera at puzzle
   - Overlay solution in real-time
   - No capture needed

6. **Cloud Sync**
   - Sync puzzles across devices
   - Backup/restore
   - Social features

7. **Accessibility Improvements**
   - Screen reader support
   - High contrast mode
   - Colorblind-friendly colors
   - Voice input

8. **Puzzle Import/Export**
   - JSON format
   - QR code sharing
   - URL-based sharing

---

## Appendix G: Resources and References

### Learning Resources

**Sudoku/Suguru Solving Algorithms**:
- "Algorithm X" by Donald Knuth (Dancing Links)
- Backtracking algorithms in "Introduction to Algorithms" (CLRS)
- Constraint Satisfaction Problems (CSP) techniques

**Computer Vision**:
- OpenCV documentation: https://docs.opencv.org/
- "Digital Image Processing" by Gonzalez & Woods
- Hough Transform tutorial: OpenCV tutorials

**Android Development**:
- Android Developers: https://developer.android.com/
- CameraX documentation
- ML Kit documentation
- Room Database guide

**Web Development**:
- Tesseract.js: https://tesseract.projectnaptha.com/
- Web Workers MDN: https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API
- Canvas API documentation

### Libraries and Tools

**Web**:
- Tesseract.js (OCR)
- OpenCV.js (image processing)
- Tailwind CSS (styling)

**Android**:
- CameraX (camera)
- ML Kit (OCR)
- OpenCV for Android (image processing)
- Room (database)
- Kotlin Coroutines (async)

### Community and Support

- Stack Overflow
- GitHub Issues (for libraries)
- Android Developers Community
- r/computerVision subreddit

---

## Document Version History

| Version | Date | Changes | Author |
|---------|------|---------|--------|
| 1.0 | 2025-11-08 | Initial comprehensive documentation | - |

---
