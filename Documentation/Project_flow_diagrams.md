## 1. Flow Diagrams
### 1.1 Overall System Flow

```mermaid
graph TD
    Start([User]) --> InputSelect[Input Method Selection<br/>- String<br/>- Camera v1.1+]
    
    InputSelect --> StringInput[String Input]
    InputSelect --> CameraInput[Camera Capture + OCR]
    
    StringInput --> Parse[Parse/Validate Puzzle]
    CameraInput --> Parse
    
    Parse --> Render[Render Grid]
    
    Render --> EditOrSolve{User Action}
    EditOrSolve --> Edit[Edit Mode]
    EditOrSolve --> Solve[Solve]
    
    Edit --> ShowResult[Show Solution/Hint]
    Solve --> ShowResult
    
    ShowResult --> End([End])
    
    style Start fill:#e1f5ff
    style End fill:#e1f5ff
    style InputSelect fill:#fff4e6
    style Parse fill:#f3e5f5
    style Render fill:#e8f5e9
    style ShowResult fill:#fff9c4
```
---

### 1.2 String Input Flow (v1.0)
```mermaid
graph TD
    A[User enters string] --> B[parsePuzzleString]
    B --> C{Valid?}
    C -->|Invalid| D[Show error message]
    C -->|Valid| E[Extract dimensions]
    E --> F[Parse cage data<br/>column-major]
    F --> G[Parse value data<br/>column-major]
    G --> H[Build cages metadata]
    H --> I[Update puzzleState]
    I --> J[renderGrid]
    J --> K[Enable Solve/Reset buttons]
    
    style A fill:#e1f5ff
    style D fill:#ffebee
    style K fill:#e8f5e9
```
---

### 1.3 Camera Capture Flow (v1.1)

```mermaid
graph TD
    A[User clicks Camera button] --> B[Request camera permission]
    B --> C{Permission?}
    C -->|Denied| D[Show error + manual input option]
    C -->|Granted| E[Initialize camera stream]
    E --> F[Display live preview<br/>with grid overlay]
    F --> G[User aligns puzzle]
    G --> H[User clicks Capture]
    H --> I[Stop camera stream]
    I --> J[Image Processing Pipeline]
    
    J --> J1[1. Grayscale conversion]
    J1 --> J2[2. Adaptive thresholding]
    J2 --> J3[3. Edge detection Canny]
    J3 --> J4[4. Hough line transform]
    J4 --> J5[5. Grid intersection calc]
    J5 --> J6[6. Perspective correction]
    J6 --> J7[7. Cage boundary detection]
    J7 --> J8[8. Cell extraction]
    J8 --> J9[9. OCR per cell]
    
    J9 --> K[Present detected puzzle<br/>editable preview]
    K --> L[User reviews/corrects]
    L --> M{Action?}
    M -->|Retry| A
    M -->|Confirm| N[Convert to puzzleState]
    N --> O[renderGrid]
    
    style A fill:#e1f5ff
    style D fill:#ffebee
    style J fill:#fff4e6
    style O fill:#e8f5e9
```
---

### 1.4 Solver Flow (Backtracking Algorithm)

```mermaid
graph TD
    A[START: backtrackingSolver] --> B[findEmptyCell]
    B --> C{Cell found?}
    C -->|null| D[RETURN TRUE<br/>Solved!]
    C -->|Found r,c| E[Get cage for cell]
    E --> F[FOR num = 1 to cage.size]
    F --> G{isValidPlacement<br/>r, c, num?}
    G -->|False| H[Try next num]
    H --> F
    G -->|True| I[Place num in grid r c]
    I --> J[Recursive call:<br/>backtrackingSolver]
    J --> K{Result?}
    K -->|True| L[RETURN TRUE]
    K -->|False| M[Remove num<br/>backtrack]
    M --> H
    F --> N{All nums tried?}
    N -->|Yes| O[RETURN FALSE]
    
    style A fill:#e1f5ff
    style D fill:#c8e6c9
    style L fill:#c8e6c9
    style O fill:#ffccbc
```
---

### 1.5 Validation Flow

```mermaid
graph TD
    A[validateCellValue<br/>r, c, num, state] --> B[Check 8 adjacent cells<br/>orthogonal + diagonal]
    B --> C{num found<br/>in adjacent?}
    C -->|Yes| D[RETURN FALSE]
    C -->|No| E[Get cage for cell r,c]
    E --> F[FOR each cell in cage]
    F --> G{cell value<br/>== num?}
    G -->|Yes| H[RETURN FALSE]
    G -->|No| I{More cells?}
    I -->|Yes| F
    I -->|No| J[All checks passed]
    J --> K[RETURN TRUE]
    
    style A fill:#e1f5ff
    style D fill:#ffccbc
    style H fill:#ffccbc
    style K fill:#c8e6c9
```
---

### 1.6 Android App Navigation Flow

```mermaid
graph TD
    A[App Launch] --> B[Main Screen<br/>Recent List]
    B --> C{User Action}
    C -->|New Puzzle| D[Input Activity<br/>Camera/String]
    C -->|Continue Existing| E[Solver Activity]
    D --> E
    E --> F[Solver Activity]
    
    style A fill:#e1f5ff
    style B fill:#fff4e6
    style D fill:#f3e5f5
    style E fill:#e8f5e9
    style F fill:#e8f5e9
```

---