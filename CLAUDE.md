# CLAUDE.md - AI Assistant Guide for BurstCard/FlashABC

## Project Overview

**BurstCard** (also known as **FlashABC**) is a client-side web application for creating, managing, and learning with digital flashcards. It features interactive learning games and supports image-based cards with text labels.

**Key Characteristics:**
- Single-file monolithic architecture (all HTML, CSS, JS in `index.html`)
- No backend server required - runs entirely in the browser
- Mobile-first responsive design with iOS Web App support
- Offline-capable (except ZIP export which requires CDN)

---

## Repository Structure

```
/home/user/burstcard/
├── index.html              # Complete application (~6,000 lines)
├── PERFORMANCE_ANALYSIS.md # Detailed performance audit report
├── CLAUDE.md               # This file
└── Audio Files (8 total):
    ├── Shuffle_Chips.wav   # Card shuffle sound
    ├── flip2.wav           # Card flip sound
    ├── correct.wav         # Correct answer feedback
    ├── error.wav           # Wrong answer feedback
    ├── explosion.wav       # Bomb game effect
    ├── Winning_Guess_the_word.wav  # Victory sound
    ├── ope.wav             # Game sound
    └── wow.wav             # Game sound
```

---

## Technology Stack

| Technology | Purpose |
|------------|---------|
| HTML5 | Structure, file inputs, audio elements |
| CSS3 | Flexbox/Grid layouts, animations, responsive design |
| Vanilla JavaScript | All application logic (no frameworks) |
| IndexedDB | Persistent storage for Units/Quick Load |
| SessionStorage | Current session card storage |
| JSZip (CDN) | ZIP file creation/extraction |
| Google Fonts | Caveat Brush, Dancing Script, Great Vibes |

---

## Application Architecture

### Global State Variables (index.html:1610-1645)

```javascript
let cards = [];                    // Main card array [{frontImage, backText, backImage}]
let currentIndex = 0;              // Current card position
let isFlipped = false;             // Card flip state
let currentLoadedUnitId = null;    // Active unit tracking
let guessGameState = {};           // Guess the Word game session
let bombGameState = {};            // Bomb game session
let spellingGameState = {};        // Spelling game session
```

### DOM Cache (index.html:1645-1678)

The app uses a `domCache` object to cache frequently accessed DOM elements for performance:

```javascript
const domCache = {
    shuffleBtn: document.getElementById('shuffleBtn'),
    clearBtn: document.getElementById('clearBtn'),
    // ... other cached elements
};
```

### Storage Architecture

1. **SessionStorage** (`flashcards` key): Stores current cards as JSON
2. **IndexedDB** (`FlashcardUnitsDB`): Stores saved units and Quick Load configurations
   - Object Store: `units` - Contains unit metadata and card data

---

## Key Functional Modules

### Card Management
| Function | Location | Purpose |
|----------|----------|---------|
| `addCard()` | ~line 2231 | Add new cards with image compression |
| `deleteCard()` | ~line 2350 | Remove single card |
| `shuffleCards()` | ~line 2324 | Randomize card order (uses Fisher-Yates) |
| `flipCard()` | ~line 2400 | Toggle card front/back |
| `saveCards()` | ~line 2450 | Persist to sessionStorage |
| `loadCards()` | ~line 2480 | Retrieve from sessionStorage |

### Rendering
| Function | Location | Purpose |
|----------|----------|---------|
| `renderDisplay()` | ~line 2967 | Main card display (full HTML rebuild) |
| `renderImagePreviews()` | ~line 1363 | Upload preview grid |
| `scaleTextToFit()` | ~line 3037 | Dynamic text sizing |

### Game Modes
| Function | Purpose |
|----------|---------|
| `startGame(gameType)` | Initialize game mode |
| `renderGuessGame()` | Multiple choice game UI |
| `renderBombGame()` | Time-pressure elimination game |
| `renderSpellingGame()` | Letter unscrambling puzzle |

### Unit/Quick Load System
| Function | Purpose |
|----------|---------|
| `openUnitConfigModal()` | Unit configuration interface |
| `addNewUnit()` | Save unit to IndexedDB |
| `loadUnit()` | Load unit from IndexedDB |
| `deleteUnitFromDB()` | Remove unit |

### Utilities
| Function | Purpose |
|----------|---------|
| `playSound(soundId)` | Audio playback helper |
| `showToast()` | Notification display |
| `fisherYatesShuffle()` | Proper O(n) shuffle algorithm |
| `fastHash()` | Hash function for duplicate detection |
| `compressImage()` | Auto-compress images to 500KB |

---

## Code Conventions

### JavaScript Patterns

1. **Event Handlers**: Mix of inline handlers (in innerHTML) and addEventListener
   ```javascript
   // Inline in templates
   onclick="flipCard()"

   // addEventListener for complex handlers
   backTextElement.addEventListener('keydown', protectNumberedList);
   ```

2. **State Changes**: Most state changes trigger full re-renders via `renderDisplay()`

3. **Async Operations**: Uses callbacks for FileReader, Promises for IndexedDB

4. **Error Handling**: Try-catch blocks around DataTransfer API (Safari compatibility)

### CSS Patterns

1. **Mobile-First**: Base styles for mobile, media queries for larger screens
2. **Breakpoints**:
   - `@media (max-width: 480px)` - Small phones
   - `@media (max-width: 768px)` - Tablets portrait
   - `@media (min-width: 769px) and (max-width: 1024px)` - Tablets landscape
   - `@media (min-width: 1025px)` - Desktop

3. **Colors**:
   - Primary green: `#35654D`, `#2d5542`
   - Accent orange: `#f5a623`, `#f39c12`
   - Accent blue: `#667eea`

4. **Animations**: CSS transitions for hover states, transforms for card flips

### Naming Conventions

- **Functions**: camelCase, verb-first (`addCard`, `renderDisplay`, `handleDrop`)
- **Variables**: camelCase (`currentIndex`, `isFlipped`, `gameTimeout`)
- **DOM IDs**: camelCase (`createModal`, `frontUpload`, `backText`)
- **CSS Classes**: kebab-case (`card-container`, `nav-btn`, `modal-content`)

---

## Common Development Tasks

### Adding a New Feature

1. Add HTML structure in appropriate section of `index.html`
2. Add CSS styles in the `<style>` block (~lines 22-1600)
3. Add JavaScript logic in the `<script>` block (~lines 1600-6000)
4. If state-related, update relevant global variables
5. If UI-related, modify `renderDisplay()` or create new render function

### Modifying Card Behavior

1. Card data structure: `{frontImage: base64, backText: string, backImage: null}`
2. Cards stored in global `cards` array
3. Changes must call `saveCards()` to persist
4. UI updates require `renderDisplay()` call

### Adding a New Game Mode

1. Create game state object (follow `guessGameState` pattern)
2. Add game initialization in `startGame()`
3. Create render function (`renderNewGame()`)
4. Add game button in game menu modal
5. Handle game completion and scoring

### Working with IndexedDB Units

```javascript
// Opening database
const request = indexedDB.open('FlashcardUnitsDB', 1);

// Creating object store (on upgrade)
db.createObjectStore('units', { keyPath: 'id', autoIncrement: true });

// CRUD operations use transaction/objectStore pattern
```

---

## Known Issues & Performance Considerations

See `PERFORMANCE_ANALYSIS.md` for detailed analysis. Key issues:

### Critical
1. **Excessive DOM manipulation**: 100+ queries per render cycle
2. **N+1 FileReader pattern**: Up to 30 simultaneous FileReaders
3. **Full innerHTML re-renders**: Destroys/recreates entire DOM subtrees

### High Priority
4. **Text scaling layout thrashing**: Synchronous reflows in loop
5. **Event listener overhead**: Repeated attach/detach cycles

### Memory Concerns
- Base64 images increase size by 33%
- 30 large images can consume 600MB+ RAM
- SessionStorage has 5-10MB limit (will fail with large images)

---

## Testing Recommendations

### Manual Testing Checklist

1. **Card Operations**
   - [ ] Create card with single image
   - [ ] Create multiple cards (batch upload)
   - [ ] Delete single card
   - [ ] Clear all cards (double-click confirmation)
   - [ ] Shuffle cards
   - [ ] Navigate (next/prev, swipe gestures)
   - [ ] Flip card (click, tap)

2. **File Import/Export**
   - [ ] JSON export/import
   - [ ] ZIP export/import
   - [ ] URL-based Quick Load
   - [ ] Image compression (>500KB images)

3. **Game Modes**
   - [ ] Guess the Word (correct/wrong answers)
   - [ ] Bomb Game (timer, elimination)
   - [ ] Spelling Game (letter arrangement)

4. **Cross-Platform**
   - [ ] Desktop (Chrome, Firefox, Safari)
   - [ ] Mobile (iOS Safari, Android Chrome)
   - [ ] Tablet (iPad, Android tablet)

### Performance Testing

Use Chrome DevTools:
1. **Performance tab**: Record card navigation, game play
2. **Memory tab**: Monitor heap during image uploads
3. **Network tab**: Verify CDN loading (JSZip)

---

## Git Workflow

- Main branch: typically `main` or `master`
- Feature branches: `claude/feature-name-sessionid`
- Commit messages: Descriptive, present tense ("Fix card flip on desktop")

Recent development focus:
- Card flip functionality fixes
- Quick Load dropdown improvements
- Mobile/iPad display optimizations
- Audio playback debugging

---

## External Dependencies

| Dependency | Version | CDN URL | Purpose |
|------------|---------|---------|---------|
| JSZip | 3.10.1 | cdnjs.cloudflare.com | ZIP file handling |
| Google Fonts | - | fonts.googleapis.com | Custom fonts |

**Note**: Application works offline except for:
- Initial font loading
- ZIP export functionality (requires JSZip CDN)

---

## Quick Reference

### File Locations in index.html

| Section | Line Range |
|---------|------------|
| Meta/Head | 1-21 |
| CSS Styles | 22-1600 |
| HTML Body | 1600-1610 |
| JavaScript | 1610-6067 |
| Global Variables | 1610-1678 |
| Event Setup | 1680-2100 |
| Core Functions | 2100-3100 |
| Game Logic | 3100-4500 |
| UI Modals | 4500-5500 |
| Initialization | 5500-6067 |

### Common Debug Points

- Card not displaying: Check `renderDisplay()` and `cards` array
- Flip not working: Check `flipCard()` and `isFlipped` state
- Audio not playing: Check `playSound()` and audio element preload
- Storage issues: Check sessionStorage quota, IndexedDB permissions
- Mobile gestures: Check `setupSwipeGestures()` touch handlers
