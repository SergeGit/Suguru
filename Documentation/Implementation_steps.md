## 1. Implementation Steps

### Phase 1: Web Application Enhancement (v1.1)

#### Step 1: Set Up Camera Module Foundation
**Estimated Time:** 4 hours

**Tasks:**
1. Add camera permission request functionality
2. Create camera UI component (button, preview area)
3. Implement getUserMedia() for camera access
4. Add error handling for permission denial
5. Test on multiple browsers

**Code Structure:**
```javascript
// Add to Suguru.html
const cameraModule = {
  stream: null,
  
  async requestCamera() {
    try {
      this.stream = await navigator.mediaDevices.getUserMedia({
        video: { facingMode: 'environment' }
      });
      return true;
    } catch (error) {
      console.error('Camera access denied:', error);
      return false;
    }
  },
  
  releaseCamera() {
    if (this.stream) {
      this.stream.getTracks().forEach(track => track.stop());
      this.stream = null;
    }
  }
};
```

**Deliverables:**
- Working camera preview
- Permission handling
- Browser compatibility testing report

---

#### Step 2: Implement Basic Image Capture
**Estimated Time:** 3 hours

**Tasks:**
1. Add "Capture" button to camera interface
2. Implement frame capture to canvas
3. Display captured image for review
4. Add "Retry" and "Process" buttons

**Code Structure:**
```javascript
function captureImage() {
  const video = document.getElementById('camera-preview');
  const canvas = document.createElement('canvas');
  canvas.width = video.videoWidth;
  canvas.height = video.videoHeight;
  const ctx = canvas.getContext('2d');
  ctx.drawImage(video, 0, 0);
  return ctx.getImageData(0, 0, canvas.width, canvas.height);
}
```

**Deliverables:**
- Captured image display
- UI for confirmation/retry

---

#### Step 3: Implement Image Preprocessing
**Estimated Time:** 8 hours

**Tasks:**
1. Add preprocessing library (or implement from scratch)
2. Implement grayscale conversion
3. Implement adaptive thresholding
4. Add Gaussian blur for noise reduction
5. Test preprocessing pipeline on sample images

**Libraries to Consider:**
- **OpenCV.js** (comprehensive but large)
- **Custom implementation** (lightweight, specific to needs)

**Code Structure:**
```javascript
const imageProcessor = {
  toGrayscale(imageData) {
    const data = imageData.data;
    for (let i = 0; i < data.length; i += 4) {
      const gray = 0.299 * data[i] + 0.587 * data[i+1] + 0.114 * data[i+2];
      data[i] = data[i+1] = data[i+2] = gray;
    }
    return imageData;
  },
  
  adaptiveThreshold(imageData, blockSize = 11, constant = 2) {
    // Implementation using local mean calculation
    // ...
  }
};
```

**Deliverables:**
- Preprocessing functions
- Visual comparison tool (original vs processed)
- Performance benchmarks

---

#### Step 4: Implement Grid Line Detection
**Estimated Time:** 12 hours

**Tasks:**
1. Implement Canny edge detection
2. Implement Hough line transform
3. Filter and group lines (horizontal/vertical)
4. Calculate grid intersections
5. Validate grid structure (regular spacing)
6. Handle edge cases (missing lines, extra noise)

**Algorithm Overview:**
```javascript
function detectGrid(imageData) {
  // 1. Edge detection
  const edges = cannyEdgeDetection(imageData);
  
  // 2. Line detection
  const lines = houghLineTransform(edges);
  
  // 3. Classify lines
  const horizontal = lines.filter(l => Math.abs(l.angle) < 10);
  const vertical = lines.filter(l => Math.abs(l.angle - 90) < 10);
  
  // 4. Find intersections
  const intersections = calculateIntersections(horizontal, vertical);
  
  // 5. Construct grid
  return buildGridStructure(intersections);
}
```

**Challenges:**
- Varying line thickness
- Incomplete or faint lines
- Perspective distortion
- Background noise

**Deliverables:**
- Grid detection function
- Debug visualization (overlay detected lines)
- Test suite with various puzzle images

---

#### Step 5: Implement Perspective Correction
**Estimated Time:** 6 hours

**Tasks:**
1. Detect outer grid corners
2. Calculate homography transformation matrix
3. Apply perspective warp
4. Crop to grid bounds

**Code Structure:**
```javascript
function perspectiveCorrect(imageData, corners) {
  // corners: [{x, y}, {x, y}, {x, y}, {x, y}] (TL, TR, BR, BL)
  
  // 1. Calculate destination dimensions
  const width = Math.max(
    distance(corners[0], corners[1]),
    distance(corners[3], corners[2])
  );
  const height = Math.max(
    distance(corners[0], corners[3]),
    distance(corners[1], corners[2])
  );
  
  // 2. Calculate homography matrix
  const H = calculateHomography(corners, [
    {x: 0, y: 0}, {x: width, y: 0},
    {x: width, y: height}, {x: 0, y: height}
  ]);
  
  // 3. Apply transformation
  return warpPerspective(imageData, H, width, height);
}
```

**Deliverables:**
- Perspective correction function
- Before/after visualization
- Testing with angled photos (15°, 30°, 45°)

---

#### Step 6: Implement Cage Boundary Detection
**Estimated Time:** 10 hours

**Tasks:**
1. Enhance edges within grid cells
2. Detect thick internal boundaries (cage separators)
3. Use flood fill to group cells into cages
4. Validate cage structure (connected regions)
5. Assign unique cage IDs

**Approach:**
- **Method 1:** Thickness-based (thick lines = cage boundaries)
- **Method 2:** Connected components after removing grid lines
- **Hybrid:** Combine both approaches

**Code Structure:**
```javascript
function detectCages(gridImage, gridInfo) {
  // 1. Remove grid lines (replace with white)
  const cleaned = removeGridLines(gridImage, gridInfo);
  
  // 2. Find remaining thick lines (cage boundaries)
  const cageBorders = detectThickLines(cleaned, threshold = 3);
  
  // 3. Flood fill to group cells
  const cageMap = floodFillCages(gridInfo, cageBorders);
  
  // 4. Validate and assign IDs
  return assignCageIds(cageMap);
}
```

**Deliverables:**
- Cage detection function
- Debug visualization (color-coded cages)
- Accuracy metrics on test dataset

---

#### Step 7: Implement OCR for Number Recognition
**Estimated Time:** 8 hours

**Tasks:**
1. Integrate Tesseract.js library
2. Extract individual cell regions
3. Preprocess cells for OCR (crop, enhance contrast)
4. Configure Tesseract for single digit recognition
5. Apply confidence filtering
6. Handle empty cells

**Code Structure:**
```javascript
async function recognizeNumbers(gridImage, gridInfo) {
  const cells = extractCells(gridImage, gridInfo);
  const results = [];
  
  for (const cell of cells) {
    const processed = preprocessCell(cell);
    
    const { data: { text, confidence } } = await Tesseract.recognize(
      processed,
      'eng',
      {
        tessedit_char_whitelist: '123456789',
        tessedit_pageseg_mode: Tesseract.PSM.SINGLE_CHAR
      }
    );
    
    results.push({
      position: cell.position,
      value: confidence > 70 ? parseInt(text) : null,
      confidence: confidence
    });
  }
  
  return results;
}
```

**Deliverables:**
- OCR integration
- Cell preprocessing pipeline
- Confidence threshold tuning report
- Support for printed and handwritten digits (separate models if needed)

---

#### Step 8: Build Camera Capture UI Integration
**Estimated Time:** 6 hours

**Tasks:**
1. Create camera mode toggle
2. Implement editable preview grid
3. Allow manual correction of detected values
4. Add confidence indicators (color-coded cells)
5. Implement "Confirm" → convert to puzzleState

**UI Features:**
- Low confidence cells highlighted in yellow
- Empty cells highlighted in gray
- Click cell to manually enter/change value
- Cage boundaries overlaid for verification

**Deliverables:**
- Complete camera capture workflow
- Manual correction interface
- User testing with 5+ test puzzles

---

#### Step 9: Implement Hint System
**Estimated Time:** 5 hours

**Tasks:**
1. Add "Hint" button to UI
2. Implement hint generation logic
3. Find cell with fewest valid options (Most Constrained Variable)
4. Highlight cell and show possible values
5. Add explanation for hint

**Code Structure:**
```javascript
function getSingleHint(puzzleState) {
  let bestCell = null;
  let minOptions = Infinity;
  
  // Find cell with fewest valid options
  for (let r = 0; r < puzzleState.height; r++) {
    for (let c = 0; c < puzzleState.width; c++) {
      if (puzzleState.grid[r][c] !== 0) continue;
      
      const cageId = puzzleState.cageGrid[r][c];
      const cage = puzzleState.cages[cageId];
      const validOptions = [];
      
      for (let num = 1; num <= cage.size; num++) {
        if (isValidPlacement(r, c, num, cage)) {
          validOptions.push(num);
        }
      }
      
      if (validOptions.length > 0 && validOptions.length < minOptions) {
        minOptions = validOptions.length;
        bestCell = {
          row: r,
          col: c,
          options: validOptions,
          reason: validOptions.length === 1 
            ? 'Only one valid number for this cell'
            : `${validOptions.length} possible numbers`
        };
      }
    }
  }
  
  return bestCell;
}
```

**Deliverables:**
- Hint button functionality
- Visual highlight for suggested cell
- Explanation text
- Option to show all possible values or just confirmation that hint exists

---

#### Step 10: Add Puzzle Validation for Partially Solved Puzzles
**Estimated Time:** 4 hours

**Tasks:**
1. Implement full puzzle validation function
2. Detect conflicts (rule violations)
3. Highlight conflicting cells in red
4. Add "Validate" button
5. Show validation results with specific error messages

**Code Structure:**
```javascript
function validatePuzzle(puzzleState) {
  const conflicts = [];
  
  for (let r = 0; r < puzzleState.height; r++) {
    for (let c = 0; c < puzzleState.width; c++) {
      const value = puzzleState.grid[r][c];
      if (value === 0) continue;
      
      // Check adjacent cells
      for (let dr = -1; dr <= 1; dr++) {
        for (let dc = -1; dc <= 1; dc++) {
          if (dr === 0 && dc === 0) continue;
          const nr = r + dr;
          const nc = c + dc;
          if (nr >= 0 && nr < puzzleState.height && 
              nc >= 0 && nc < puzzleState.width &&
              puzzleState.grid[nr][nc] === value) {
            conflicts.push({
              cells: [{r, c}, {r: nr, c: nc}],
              type: 'adjacent_duplicate',
              message: `Adjacent cells cannot have the same value (${value})`
            });
          }
        }
      }
      
      // Check cage duplicates
      const cageId = puzzleState.cageGrid[r][c];
      const cage = puzzleState.cages[cageId];
      for (const cell of cage.cells) {
        if (cell.r === r && cell.c === c) continue;
        if (puzzleState.grid[cell.r][cell.c] === value) {
          conflicts.push({
            cells: [{r, c}, cell],
            type: 'cage_duplicate',
            message: `Same cage cannot have duplicate values (${value})`
          });
        }
      }
    }
  }
  
  return {
    valid: conflicts.length === 0,
    conflicts: conflicts
  };
}
```

**Deliverables:**
- Validation function
- Conflict visualization
- Error message display
- Auto-validation option (on/off toggle)

---

#### Step 11: Implement Manual Grid Editing
**Estimated Time:** 4 hours

**Tasks:**
1. Add "Edit Mode" toggle
2. Make cells clickable in edit mode
3. Number input interface (1-9, 0 to clear)
4. Support keyboard input
5. Distinguish user-edited cells from original/scanned values
6. Add "Clear All Solved" button

**Code Structure:**
```javascript
// Track cell sources
const cellSources = {
  original: new Set(),  // From initial puzzle
  scanned: new Set(),   // From camera OCR
  manual: new Set(),    // User edited
  solved: new Set()     // Algorithm solved
};

function enableEditMode() {
  const cells = document.querySelectorAll('.grid-cell');
  cells.forEach((cell, index) => {
    const r = Math.floor(index / puzzleState.width);
    const c = index % puzzleState.width;
    
    cell.addEventListener('click', () => {
      if (cellSources.original.has(`${r},${c}`)) {
        // Don't allow editing original cells
        return;
      }
      showNumberPicker(r, c);
    });
  });
}
```

**Deliverables:**
- Edit mode functionality
- Visual distinction for different cell types
- Keyboard shortcuts
- Undo/redo capability

---

#### Step 12: Performance Optimization and Testing
**Estimated Time:** 6 hours

**Tasks:**
1. Profile solver performance on various grid sizes
2. Optimize image processing pipeline
3. Add Web Workers for heavy computation
4. Test with 20+ different puzzle images
5. Measure accuracy metrics
6. Cross-browser testing

**Optimization Techniques:**
- Use Web Workers for solver and image processing
- Implement constraint propagation to reduce backtracking
- Cache intermediate results
- Lazy rendering for large grids

**Testing Checklist:**
- [ ] 3×3 to 10×10 grid sizes
- [ ] Printed puzzles (clear text)
- [ ] Handwritten puzzles (legible)
- [ ] Various angles (0°-30° tilt)
- [ ] Different lighting conditions
- [ ] Partially solved puzzles
- [ ] Edge cases (irregular cage shapes)

**Deliverables:**
- Performance report
- Optimization implementation
- Test results documentation
- Bug fixes

---

### Phase 2: Android Application (v2.0)

#### Step 13: Android Project Setup
**Estimated Time:** 4 hours

**Tasks:**
1. Create new Android Studio project (Kotlin)
2. Set up MVVM architecture with Navigation Component
3. Add dependencies (CameraX, Room, ML Kit)
4. Configure build.gradle files
5. Set up Git repository structure

**Dependencies:**
```gradle
dependencies {
    // Core
    implementation "androidx.core:core-ktx:1.12.0"
    implementation "androidx.appcompat:appcompat:1.6.1"
    implementation "com.google.android.material:material:1.11.0"
    
    // Architecture
    implementation "androidx.lifecycle:lifecycle-viewmodel-ktx:2.7.0"
    implementation "androidx.navigation:navigation-fragment-ktx:2.7.6"
    
    // CameraX
    implementation "androidx.camera:camera-camera2:1.3.1"
    implementation "androidx.camera:camera-lifecycle:1.3.1"
    implementation "androidx.camera:camera-view:1.3.1"
    
    // ML Kit OCR
    implementation "com.google.mlkit:text-recognition:16.0.0"
    
    // Room Database
    implementation "androidx.room:room-runtime:2.6.1"
    kapt "androidx.room:room-compiler:2.6.1"
    
    // Image Processing
    implementation "org.opencv:opencv:4.8.0"
}
```

**Deliverables:**
- Android project structure
- Build configuration
- Architecture skeleton

---

#### Step 14: Port Core Solver Logic to Kotlin
**Estimated Time:** 8 hours

**Tasks:**
1. Create PuzzleState data class
2. Port backtracking solver algorithm
3. Port validation logic
4. Optimize for mobile performance
5. Unit tests for solver

**Kotlin Implementation:**
```kotlin
data class PuzzleState(
    val width: Int,
    val height: Int,
    val grid: Array<IntArray>,
    val cageGrid: Array<IntArray>,
    val cages: Map<Int, Cage>
) {
    data class Cage(
        val size: Int,
        val cells: List<Cell>
    )
    
    data class Cell(val row: Int, val col: Int)
}

class SuguruSolver(private val puzzle: PuzzleState) {
    fun solve(): Boolean {
        return backtrack()
    }
    
    private fun backtrack(): Boolean {
        val emptyCell = findEmptyCell() ?: return true
        val (r, c) = emptyCell
        val cageId = puzzle.cageGrid[r][c]
        val cage = puzzle.cages[cageId]!!
        
        for (num in 1..cage.size) {
            if (isValid(r, c, num, cage)) {
                puzzle.grid[r][c] = num
                if (backtrack()) return true
                puzzle.grid[r][c] = 0
            }
        }
        return false
    }
    
    private fun isValid(r: Int, c: Int, num: Int, cage: Cage): Boolean {
        // Check adjacent cells
        for (dr in -1..1) {
            for (dc in -1..1) {
                if (dr == 0 && dc == 0) continue
                val nr = r + dr
                val nc = c + dc
                if (nr in 0 until puzzle.height && 
                    nc in 0 until puzzle.width &&
                    puzzle.grid[nr][nc] == num) {
                    return false
                }
            }
        }
        
        // Check cage
        for (cell in cage.cells) {
            if (puzzle.grid[cell.row][cell.col] == num) {
                return false
            }
        }
        
        return true
    }
    
    private fun findEmptyCell(): Pair<Int, Int>? {
        for (r in 0 until puzzle.height) {
            for (c in 0 until puzzle.width) {
                if (puzzle.grid[r][c] == 0) {
                    return Pair(r, c)
                }
            }
        }
        return null
    }
}
```

**Deliverables:**
- Kotlin solver implementation
- Unit tests (JUnit)
- Performance benchmarks

---

#### Step 15: Implement CameraX Integration
**Estimated Time:** 10 hours

**Tasks:**
1. Create CameraActivity with preview
2. Implement camera permission handling
3. Add capture button and image preview
4. Implement preview overlay (grid guide)
5. Handle image rotation and orientation
6. Save captured image to temporary storage

**CameraX Implementation:**
```kotlin
class CameraActivity : AppCompatActivity() {
    private lateinit var cameraExecutor: ExecutorService
    private lateinit var imageCapture: ImageCapture
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_camera)
        
        cameraExecutor = Executors.newSingleThreadExecutor()
        
        if (allPermissionsGranted()) {
            startCamera()
        } else {
            requestPermissions()
        }
    }
    
    private fun startCamera() {
        val cameraProviderFuture = ProcessCameraProvider.getInstance(this)
        
        cameraProviderFuture.addListener({
            val cameraProvider = cameraProviderFuture.get()
            
            val preview = Preview.Builder()
                .build()
                .also {
                    it.setSurfaceProvider(viewBinding.previewView.surfaceProvider)
                }
            
            imageCapture = ImageCapture.Builder()
                .setCaptureMode(ImageCapture.CAPTURE_MODE_MAXIMIZE_QUALITY)
                .build()
            
            val cameraSelector = CameraSelector.DEFAULT_BACK_CAMERA
            
            try {
                cameraProvider.unbindAll()
                cameraProvider.bindToLifecycle(
                    this, cameraSelector, preview, imageCapture
                )
            } catch (e: Exception) {
                Log.e(TAG, "Camera binding failed", e)
            }
            
        }, ContextCompat.getMainExecutor(this))
    }
    
    private fun captureImage() {
        val imageCapture = imageCapture ?: return
        
        imageCapture.takePicture(
            cameraExecutor,
            object : ImageCapture.OnImageCapturedCallback() {
                override fun onCaptureSuccess(image: ImageProxy) {
                    processImage(image)
                    image.close()
                }
                
                override fun onError(exception: ImageCaptureException) {
                    Log.e(TAG, "Capture failed: ${exception.message}", exception)
                }
            }
        )
    }
}
```

**Deliverables:**
- CameraActivity implementation
- Permission handling
- Image capture functionality
- Preview overlay UI

---

#### Step 16: Implement Android Image Processing
**Estimated Time:** 12 hours

**Tasks:**
1. Integrate OpenCV for Android
2. Port grid detection algorithm
3. Port cage detection algorithm
4. Optimize for mobile performance
5. Add progress indicators

**OpenCV Integration:**
```kotlin
class ImageProcessor(private val context: Context) {
    
    init {
        if (!OpenCVLoader.initDebug()) {
            Log.e(TAG, "OpenCV initialization failed")
        }
    }
    
    suspend fun processImage(bitmap: Bitmap): PuzzleState? = withContext(Dispatchers.Default) {
        // Convert to Mat
        val mat = Mat()
        Utils.bitmapToMat(bitmap, mat)
        
        // Preprocessing
        val gray = preprocessImage(mat)
        
        // Grid detection
        val gridInfo = detectGrid(gray) ?: return@withContext null
        
        // Perspective correction
        val corrected = correctPerspective(gray, gridInfo)
        
        // Cage detection
        val cages = detectCages(corrected, gridInfo)
        
        // OCR
        val cells = recognizeNumbers(corrected, gridInfo)
        
        // Build puzzle state
        return@withContext buildPuzzleState(gridInfo, cages, cells)
    }
    
    private fun preprocessImage(mat: Mat): Mat {
        val gray = Mat()
        Imgproc.cvtColor(mat, gray, Imgproc.COLOR_BGR2GRAY)
        
        val blurred = Mat()
        Imgproc.GaussianBlur(gray, blurred, Size(5.0, 5.0), 0.0)
        
        val thresh = Mat()
        Imgproc.adaptiveThreshold(
            blurred, thresh, 255.0,
            Imgproc.ADAPTIVE_THRESH_GAUSSIAN_C,
            Imgproc.THRESH_BINARY, 11, 2.0
        )
        
        return thresh
    }
    
    private fun detectGrid(mat: Mat): GridInfo? {
        val edges = Mat()
        Imgproc.Canny(mat, edges, 50.0, 150.0)
        
        val lines = Mat()
        Imgproc.HoughLinesP(edges, lines, 1.0, Math.PI / 180, 100, 100.0, 10.0)
        
        // Process lines to find grid...
        // (Implementation similar to web version)
        
        return null // Return actual GridInfo
    }
}
```

**Deliverables:**
- Image processing module
- Grid detection for Android
- Performance optimization
- Progress tracking

---

#### Step 17: Implement ML Kit OCR Integration
**Estimated Time:** 6 hours

**Tasks:**
1. Integrate ML Kit Text Recognition
2. Extract cell regions from processed image
3. Configure OCR for digit recognition
4. Apply confidence filtering
5. Handle recognition errors gracefully

**ML Kit Implementation:**
```kotlin
class OCRProcessor(private val context: Context) {
    private val recognizer = TextRecognition.getClient(TextRecognizerOptions.DEFAULT_OPTIONS)
    
    suspend fun recognizeNumbers(
        bitmap: Bitmap,
        gridInfo: GridInfo
    ): List<CellData> = withContext(Dispatchers.Default) {
        val results = mutableListOf<CellData>()
        
        for (cell in gridInfo.cells) {
            val cellBitmap = extractCellBitmap(bitmap, cell)
            val inputImage = InputImage.fromBitmap(cellBitmap, 0)
            
            try {
                val result = recognizer.process(inputImage).await()
                val text = result.text.filter { it.isDigit() }
                
                if (text.isNotEmpty()) {
                    val digit = text.first().toString().toIntOrNull()
                    results.add(
                        CellData(
                            position = cell.position,
                            value = digit,
                            confidence = calculateConfidence(result)
                        )
                    )
                }
            } catch (e: Exception) {
                Log.e(TAG, "OCR failed for cell ${cell.position}", e)
            }
        }
        
        return@withContext results
    }
    
    private fun extractCellBitmap(bitmap: Bitmap, cell: CellInfo): Bitmap {
        return Bitmap.createBitmap(
            bitmap,
            cell.bounds.left,
            cell.bounds.top,
            cell.bounds.width(),
            cell.bounds.height()
        )
    }
}
```

**Deliverables:**
- OCR integration
- Cell extraction
- Confidence scoring
- Error handling

---

#### Step 18: Implement Room Database for Puzzle History
**Estimated Time:** 6 hours

**Tasks:**
1. Define database schema (Puzzle entity)
2. Create DAO (Data Access Object)
3. Implement Repository pattern
4. Add CRUD operations
5. Migrate to new schema if needed

**Database Schema:**
```kotlin
@Entity(tableName = "puzzles")
data class PuzzleEntity(
    @PrimaryKey(autoGenerate = true)
    val id: Long = 0,
    val width: Int,
    val height: Int,
    val puzzleString: String,
    val status: PuzzleStatus,
    val createdAt: Long,
    val lastModified: Long,
    val isFavorite: Boolean = false
)

enum class PuzzleStatus {
    UNSOLVED, IN_PROGRESS, SOLVED
}

@Dao
interface PuzzleDao {
    @Query("SELECT * FROM puzzles ORDER BY lastModified DESC")
    fun getAllPuzzles(): Flow<List<PuzzleEntity>>
    
    @Query("SELECT * FROM puzzles WHERE id = :id")
    suspend fun getPuzzleById(id: Long): PuzzleEntity?
    
    @Insert
    suspend fun insertPuzzle(puzzle: PuzzleEntity): Long
    
    @Update
    suspend fun updatePuzzle(puzzle: PuzzleEntity)
    
    @Delete
    suspend fun deletePuzzle(puzzle: PuzzleEntity)
    
    @Query("SELECT * FROM puzzles WHERE isFavorite = 1")
    fun getFavoritePuzzles(): Flow<List<PuzzleEntity>>
}

@Database(entities = [PuzzleEntity::class], version = 1)
abstract class PuzzleDatabase : RoomDatabase() {
    abstract fun puzzleDao(): PuzzleDao
}
```

**Deliverables:**
- Database implementation
- Repository layer
- ViewModels for data access
- Migration strategy

---

#### Step 19: Build Main UI Screens
**Estimated Time:** 12 hours

**Tasks:**
1. Implement MainActivity with puzzle list
2. Create SolverActivity with grid view
3. Implement SettingsActivity
4. Add navigation between screens
5. Implement RecyclerView for puzzle history
6. Add swipe-to-delete and favorite toggles

**MainActivity Layout:**
```xml
<!-- activity_main.xml -->
<androidx.coordinatorlayout.widget.CoordinatorLayout>
    <com.google.android.material.appbar.AppBarLayout>
        <com.google.android.material.appbar.MaterialToolbar
            android:id="@+id/toolbar"
            app:title="Suguru Solver" />
    </com.google.android.material.appbar.AppBarLayout>
    
    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/puzzleRecyclerView"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:layoutManager="androidx.recyclerview.widget.LinearLayoutManager" />
    
    <com.google.android.material.floatingactionbutton.FloatingActionButton
        android:id="@+id/fab"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="bottom|end"
        android:layout_margin="16dp"
        android:src="@drawable/ic_add"
        android:contentDescription="New Puzzle" />
</androidx.coordinatorlayout.widget.CoordinatorLayout>
```

**SolverActivity ViewModel:**
```kotlin
class SolverViewModel(
    private val repository: PuzzleRepository
) : ViewModel() {
    
    private val _puzzleState = MutableLiveData<PuzzleState>()
    val puzzleState: LiveData<PuzzleState> = _puzzleState
    
    private val _solveStatus = MutableLiveData<SolveStatus>()
    val solveStatus: LiveData<SolveStatus> = _solveStatus
    
    fun loadPuzzle(puzzleId: Long) {
        viewModelScope.launch {
            val puzzle = repository.getPuzzleById(puzzleId)
            if (puzzle != null) {
                _puzzleState.value = parsePuzzleString(puzzle.puzzleString)
            }
        }
    }
    
    fun solvePuzzle() {
        viewModelScope.launch(Dispatchers.Default) {
            _solveStatus.postValue(SolveStatus.SOLVING)
            
            val puzzle = _puzzleState.value ?: return@launch
            val solver = SuguruSolver(puzzle)
            
            val solved = solver.solve()
            
            if (solved) {
                _puzzleState.postValue(puzzle)
                _solveStatus.postValue(SolveStatus.SOLVED)
            } else {
                _solveStatus.postValue(SolveStatus.NO_SOLUTION)
            }
        }
    }
}
```

**Deliverables:**
- Main activity with puzzle list
- Solver activity with grid
- Settings screen
- Navigation implementation
- RecyclerView adapters

---

#### Step 20: Implement Custom Grid View for Android
**Estimated Time:** 10 hours

**Tasks:**
1. Create custom GridView component
2. Implement touch handling for cell editing
3. Draw cage boundaries
4. Apply visual styling (colors, fonts)
5. Optimize rendering performance
6. Add zoom/pan gestures for large grids

**Custom GridView:**
```kotlin
class PuzzleGridView @JvmOverloads constructor(
    context: Context,
    attrs: AttributeSet? = null,
    defStyleAttr: Int = 0
) : View(context, attrs, defStyleAttr) {
    
    private var puzzleState: PuzzleState? = null
    private val cellPaint = Paint(Paint.ANTI_ALIAS_FLAG)
    private val linePaint = Paint(Paint.ANTI_ALIAS_FLAG)
    private val textPaint = Paint(Paint.ANTI_ALIAS_FLAG)
    
    private var cellSize = 0f
    private var gridOffsetX = 0f
    private var gridOffsetY = 0f
    
    init {
        cellPaint.color = Color.WHITE
        cellPaint.style = Paint.Style.FILL
        
        linePaint.color = Color.BLACK
        linePaint.strokeWidth = 2f
        
        textPaint.color = Color.BLACK
        textPaint.textSize = 48f
        textPaint.textAlign = Paint.Align.CENTER
    }
    
    fun setPuzzle(puzzle: PuzzleState) {
        puzzleState = puzzle
        calculateDimensions()
        invalidate()
    }
    
    private fun calculateDimensions() {
        val puzzle = puzzleState ?: return
        
        val availableWidth = width - paddingLeft - paddingRight
        val availableHeight = height - paddingTop - paddingBottom
        
        cellSize = minOf(
            availableWidth / puzzle.width.toFloat(),
            availableHeight / puzzle.height.toFloat()
        )
        
        gridOffsetX = (availableWidth - cellSize * puzzle.width) / 2 + paddingLeft
        gridOffsetY = (availableHeight - cellSize * puzzle.height) / 2 + paddingTop
    }
    
    override fun onDraw(canvas: Canvas) {
        super.onDraw(canvas)
        
        val puzzle = puzzleState ?: return
        
        // Draw cells
        for (r in 0 until puzzle.height) {
            for (c in 0 until puzzle.width) {
                val left = gridOffsetX + c * cellSize
                val top = gridOffsetY + r * cellSize
                val right = left + cellSize
                val bottom = top + cellSize
                
                // Draw cell background
                canvas.drawRect(left, top, right, bottom, cellPaint)
                
                // Draw value
                val value = puzzle.grid[r][c]
                if (value != 0) {
                    val x = left + cellSize / 2
                    val y = top + cellSize / 2 - (textPaint.descent() + textPaint.ascent()) / 2
                    canvas.drawText(value.toString(), x, y, textPaint)
                }
                
                // Draw cage borders
                drawCageBorders(canvas, r, c, left, top, right, bottom)
            }
        }
        
        // Draw grid lines
        drawGridLines(canvas, puzzle)
    }
    
    private fun drawCageBorders(
        canvas: Canvas,
        r: Int, c: Int,
        left: Float, top: Float,
        right: Float, bottom: Float
    ) {
        val puzzle = puzzleState ?: return
        val cageId = puzzle.cageGrid[r][c]
        
        val thickPaint = Paint(linePaint).apply { strokeWidth = 6f }
        
        // Top border
        if (r == 0 || puzzle.cageGrid[r - 1][c] != cageId) {
            canvas.drawLine(left, top, right, top, thickPaint)
        }
        
        // Left border
        if (c == 0 || puzzle.cageGrid[r][c - 1] != cageId) {
            canvas.drawLine(left, top, left, bottom, thickPaint)
        }
        
        // Bottom border (only for last row)
        if (r == puzzle.height - 1) {
            canvas.drawLine(left, bottom, right, bottom, thickPaint)
        }
        
        // Right border (only for last column)
        if (c == puzzle.width - 1) {
            canvas.drawLine(right, top, right, bottom, thickPaint)
        }
    }
    
    private fun drawGridLines(canvas: Canvas, puzzle: PuzzleState) {
        // Draw thin grid lines
        for (i in 0..puzzle.width) {
            val x = gridOffsetX + i * cellSize
            canvas.drawLine(x, gridOffsetY, x, gridOffsetY + puzzle.height * cellSize, linePaint)
        }
        
        for (i in 0..puzzle.height) {
            val y = gridOffsetY + i * cellSize
            canvas.drawLine(gridOffsetX, y, gridOffsetX + puzzle.width * cellSize, y, linePaint)
        }
    }
    
    override fun onTouchEvent(event: MotionEvent): Boolean {
        if (event.action == MotionEvent.ACTION_DOWN) {
            val x = event.x - gridOffsetX
            val y = event.y - gridOffsetY
            
            if (x >= 0 && y >= 0) {
                val col = (x / cellSize).toInt()
                val row = (y / cellSize).toInt()
                
                val puzzle = puzzleState ?: return false
                if (row < puzzle.height && col < puzzle.width) {
                    onCellClickListener?.invoke(row, col)
                    return true
                }
            }
        }
        return super.onTouchEvent(event)
    }
    
    var onCellClickListener: ((row: Int, col: Int) -> Unit)? = null
}
```

**Deliverables:**
- Custom grid view
- Touch handling
- Visual styling
- Performance optimization

---

#### Step 21: Add Hint and Edit Features to Android
**Estimated Time:** 6 hours

**Tasks:**
1. Port hint system to Android
2. Implement edit mode toggle
3. Add number picker dialog
4. Implement validation with visual feedback
5. Add undo/redo functionality

**Deliverables:**
- Hint functionality
- Edit mode
- Number input UI
- Validation feedback

---

#### Step 22: Implement Settings and Preferences
**Estimated Time:** 4 hours

**Tasks:**
1. Create Settings screen with PreferenceFragment
2. Add solver timeout setting
3. Add theme selection (Light/Dark/System)
4. Add OCR confidence threshold setting
5. Add about/help section

**Settings Implementation:**
```kotlin
class SettingsFragment : PreferenceFragmentCompat() {
    override fun onCreatePreferences(savedInstanceState: Bundle?, rootKey: String?) {
        setPreferencesFromResource(R.xml.preferences, rootKey)
        
        findPreference<ListPreference>("theme")?.setOnPreferenceChangeListener { _, newValue ->
            applyTheme(newValue as String)
            true
        }
    }
    
    private fun applyTheme(theme: String) {
        when (theme) {
            "light" -> AppCompatDelegate.setDefaultNightMode(AppCompatDelegate.MODE_NIGHT_NO)
            "dark" -> AppCompatDelegate.setDefaultNightMode(AppCompatDelegate.MODE_NIGHT_YES)
            "system" -> AppCompatDelegate.setDefaultNightMode(AppCompatDelegate.MODE_NIGHT_FOLLOW_SYSTEM)
        }
    }
}
```

**Deliverables:**
- Settings screen
- Theme support
- Preference persistence

---

#### Step 23: Testing and QA
**Estimated Time:** 12 hours

**Tasks:**
1. Unit tests for solver and validation
2. Integration tests for image processing
3. UI tests with Espresso
4. Manual testing on multiple devices
5. Performance profiling
6. Memory leak detection
7. Bug fixing

**Test Coverage:**
- Solver correctness (100+ test puzzles)
- OCR accuracy (50+ images per category)
- UI responsiveness
- Edge cases (empty grids, invalid inputs)
- Permission handling
- Database operations

**Deliverables:**
- Test suite
- Test report
- Bug fixes
- Performance report

---

#### Step 24: Polish and Release Preparation
**Estimated Time:** 8 hours

**Tasks:**
1. Create app icon and splash screen
2. Write user guide/tutorial
3. Add onboarding flow
4. Implement analytics (optional, privacy-focused)
5. Prepare Play Store listing
6. Generate signed APK
7. Beta testing with external users

**Deliverables:**
- Polished UI
- User documentation
- Release build
- Play Store assets

---

## Summary Timeline

### Web Application (v1.1)
| Step | Description | Time | Dependencies |
|------|-------------|------|--------------|
| 1 | Camera Module Foundation | 4h | - |
| 2 | Image Capture | 3h | Step 1 |
| 3 | Image Preprocessing | 8h | Step 2 |
| 4 | Grid Line Detection | 12h | Step 3 |
| 5 | Perspective Correction | 6h | Step 4 |
| 6 | Cage Boundary Detection | 10h | Step 5 |
| 7 | OCR Integration | 8h | Step 6 |
| 8 | Camera UI Integration | 6h | Steps 1-7 |
| 9 | Hint System | 5h | - |
| 10 | Puzzle Validation | 4h | - |
| 11 | Manual Editing | 4h | - |
| 12 | Optimization & Testing | 6h | All above |
| **Total** | | **79h** | (~2 weeks) |

### Android Application (v2.0)
| Step | Description | Time | Dependencies |
|------|-------------|------|--------------|
| 13 | Android Project Setup | 4h | - |
| 14 | Port Solver to Kotlin | 8h | Step 13 |
| 15 | CameraX Integration | 10h | Step 13 |
| 16 | Android Image Processing | 12h | Step 15 |
| 17 | ML Kit OCR | 6h | Step 16 |
| 18 | Room Database | 6h | Step 13 |
| 19 | Main UI Screens | 12h | Steps 13, 18 |
| 20 | Custom Grid View | 10h | Steps 14, 19 |
| 21 | Hint & Edit Features | 6h | Steps 14, 20 |
| 22 | Settings & Preferences | 4h | Step 19 |
| 23 | Testing & QA | 12h | All above |
| 24 | Polish & Release Prep | 8h | All above |
| **Total** | | **98h** | (~2.5 weeks) |

**Grand Total: 177 hours (~4.5 weeks of development time)**

---