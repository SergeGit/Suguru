## 1. Product Requirements Document (PRD)

### 1.1 Project Overview

**Product Name:** Suguru Puzzle Solver  
**Version:** 1.0 (Web) → 2.0 (Android)  
**Purpose:** Enable users to input, visualize, and solve Suguru puzzles through multiple input methods (string, camera) on both web browsers and Android devices.

### 1.2 Background

Suguru is a logic puzzle where:
- Numbers fill a grid divided into regions (cages)
- Each region of N cells contains numbers 1 through N
- No two adjacent cells (including diagonals) can contain the same number
- Users often encounter puzzles in books, newspapers, or apps and need solving assistance

[For a refresher on Suguru rules, see Appendix A](#appendix-a-suguru-rules-reference)

### 1.3 Target Users

- **Primary:** Puzzle enthusiasts who want to verify solutions or get unstuck
- **Secondary:** Educators demonstrating constraint satisfaction algorithms
- **Tertiary:** Developers learning computer vision and backtracking algorithms

### 1.4 Success Metrics

- Solve 95%+ of valid puzzles correctly
- Camera recognition accuracy >85% for printed puzzles, >75% for handwritten
- Web solver response time <2 seconds for grids up to 10×10
- Android app solve time <5 seconds for grids up to 10×10

### 1.5 Core Features

#### Phase 1: Web Application (v1.0 - v1.1)

**v1.0 Features (Implemented):**
- ✅ String-based puzzle import with format validation
- ✅ Dynamic grid visualization with cage boundaries
- ✅ Backtracking solver algorithm
- ✅ Reset functionality
- ✅ Visual distinction between initial and solved values

**v1.1 Features (Planned):**
- 📷 Camera-based puzzle capture
- 🔍 Grid line and cage boundary detection
- 🔢 OCR for number recognition (printed and handwritten)
- ✏️ Manual editing of detected cells
- ✔️ Validation of partially solved puzzles
- 💡 Hint system (single cell suggestion)

#### Phase 2: Android Application (v2.0)

- 📱 Native Android camera integration
- 🎯 Real-time preview with grid overlay
- 💾 Puzzle history and favorites
- 📤 Share puzzles and solutions
- ⚡ Optimized mobile solver performance
- 🌙 Dark mode support

### 1.6 Non-Functional Requirements

**Performance:**
- Web: Support grids up to 12×12
- Android: Support grids up to 10×10
- Solver timeout: 30 seconds maximum

**Usability:**
- Responsive design for tablets and phones
- Accessible UI (WCAG 2.1 AA compliance)
- Offline capability for Android app

**Compatibility:**
- Web: Chrome 90+, Firefox 88+, Safari 14+, Edge 90+
- Android: API Level 24+ (Android 7.0+)

**Security:**
- No personal data collection
- Camera permissions requested only when needed
- Local processing (no server uploads)

### 1.7 Constraints and Assumptions

**Constraints:**
- Camera quality affects recognition accuracy
- Handwritten puzzles require clear, legible digits
- Complex cage shapes may challenge boundary detection
- Limited device storage for Android app

**Assumptions:**
- Users photograph puzzles from reasonable angles (<30° tilt)
- Puzzles have visible cage boundaries
- Single puzzle per image
- Adequate lighting conditions

---