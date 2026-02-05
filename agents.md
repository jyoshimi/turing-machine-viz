# Agent Documentation

This file contains information to help AI agents and developers quickly locate and modify commonly-changed parts of the codebase.

## Arrow Styling (State Diagram)

**Location:** `src/state-diagram/StateViz.css`

### Key CSS Classes

- `.edgepath` - Controls the arrow lines/paths between states
  - `stroke`: Color of the arrow lines
  - `stroke-width`: Thickness of the arrow lines
  
- `#arrowhead`, `#reversed-arrowhead` - Controls the arrowhead markers
  - `stroke`: Color of the arrowhead outline
  - `fill`: Color of the arrowhead fill

- `#active-arrowhead`, `#reversed-active-arrowhead` - Controls active/highlighted arrowheads
  - Currently set to `lightskyblue` for active transitions

- `.edgelabel` - Controls the labels on arrows (transition conditions)
  - `font-size`: Size of the edge label text
  - `fill`: Color of the edge label text

### Recent History of Arrow Styling Changes

1. **commit 76d9016** (Oct 2015) - Initial addition of arrowheads to edges
   - Added arrow markers to state diagram
   - Initial color: `#ccc` (light gray)

2. **commit b249aea** (Jul 2016) - Moved active arrow color to CSS
   - Added `.edgepath.active-edge` class with `lightskyblue` color
   - Fixed IE compatibility issue

3. **commit b74478c** (Feb 2026) - Made arrows and edge labels darker for better visibility
   - Changed `.edgepath` stroke from `#ccc` to `#666`
   - Changed arrowhead colors from `#ccc` to `#666`
   - Changed `.edgelabel` fill from `#aaa` to `#333`

4. **commit 0b60019** (Feb 2026) - Increased fonts for visibility
   - Changed `.edgelabel` font-size from `10px` to `11px`

5. **Latest change** (Feb 2026) - Made arrows even darker and thicker
   - Changed `.edgepath` stroke from `#666` to `black`
   - Added `stroke-width: 2px` to `.edgepath`
   - Changed arrowhead colors from `#666` to `black`

### Other Related Styling

- **Active edge animation:** `src/TMViz.js` contains the `pulseEdge()` function that animates edges during transitions
  - Temporarily increases `stroke-width` from `1px` to `3px` during pulse animation
  - Uses `.active-edge` class to change color

## Font Visibility

Multiple commits have adjusted font sizes for better visibility. See `.edgelabel` and `.nodelabel` classes in `src/state-diagram/StateViz.css`.

## Parsing and Instruction Format

**Location:** `src/parser.js`

### Current Instruction Format (as of Feb 2026)

Instructions use **explicit key-value pairs** with three required keys:
- `write`: The symbol to write (single character or empty string)
- `move`: The direction to move (`L` or `R`)
- `nextState`: The next state to transition to

**Example:**
```yaml
{write: '1', move: R, nextState: accept}
```

### Shorthand Forms

1. **Direction-only:** Just `L` or `R` (no write, no state change)
2. **Synonyms:** Custom abbreviations defined in the `synonyms` section

### History of Instruction Format Changes

#### 1. **commit eb7149b** (Jan 2016) - Initial YAML parser implementation
- Introduced YAML-based format replacing previous format
- Original concise notation: `L`, `R`, `{L: q5}`, `{write: x, R: q3}`
- Direction (`L` or `R`) was used as the key in instruction objects
- Supported synonyms for common patterns
- Designed with specific and helpful error messages

**Old format example:**
```yaml
table:
  state1:
    '0': {write: 1, R: state2}
    '1': {L: state3}
```

#### 2. **commit 5c14e4c** (Jan 2016) - Parser convenience fixes
- Auto-convert tape symbols to strings (e.g., `0` to `'0'`)
- Fixed halting states (properly handle `null` values)
- Fixed swapped movement directions bug

#### 3. **commit 02517c9** (Jan 2016) - Improved parser error messages
- Made error messages more descriptive with examples
- Added `.problemValue` field to separate values from error headers
- Added check for typos in instruction keys
- Fixed error handling for malformed synonym definitions
- Better formatting of error messages in HTML

**Key improvements:**
- Errors now show: `<strong>Error reason problemValue</strong> location`
- Added suggestions for common mistakes
- Better context in error messages (state, symbol, synonym)

#### 4. **commit bbc7180** (Feb 2026) - Changed instruction format to explicit key-value pairs
**MAJOR BREAKING CHANGE** - This fundamentally changed the instruction format

**Old format (direction as key):**
```yaml
{write: 0, L: carry}
{R: accept}
```

**New format (explicit keys):**
```yaml
{write: 0, move: L, nextState: carry}
{write: '', move: R, nextState: accept}
```

**Rationale:** Makes instructions more explicit and easier for students to understand. All three keys (`write`, `move`, `nextState`) are now required for every instruction object.

**Parser changes:**
- `parseInstructionObject()` completely rewritten
- Now checks for all three required keys
- Rejects old format with clear error messages
- `write` can now be empty string (length 0) to indicate no write
- Direction is now a value (`move: L`) instead of a key (`L: state`)

#### 5. **commit 4680c12** (Feb 2026) - Updated all examples to new format
- Converted all 15 built-in example files to new instruction format
- Updated documentation in `index.html` with new format examples
- Updated template file (`_template.yaml`)

### Parser Architecture

**Main parsing flow:**
1. `parseSpec(str)` - Entry point, parses YAML and validates structure
2. `parseSynonyms(val, table)` - Parses synonym definitions
3. `parseTable(synonyms, val)` - Parses transition table
4. `parseInstruction(synonyms, table, val)` - Dispatches to string or object parser
5. `parseInstructionString(synonyms, val)` - Handles `L`, `R`, or synonym strings
6. `parseInstructionObject(val)` - Handles `{write, move, nextState}` objects
7. `checkTarget(table, instruct)` - Validates that target states exist

### Error Handling

The parser uses `TMSpecError` for machine specification errors (distinct from YAML syntax errors which throw `YAMLException`).

**Error message structure:**
- `reason`: Short error code (e.g., "Missing required key")
- `details`: Object with context:
  - `problemValue`: The problematic value
  - `state`, `symbol`, `synonym`: Location context
  - `info`: Detailed explanation
  - `suggestion`: How to fix it

**Common validation checks:**
- Required fields: `blank`, `start state`, `table`
- Blank symbol must be single character
- Start state must exist in table
- All target states must be declared
- Instruction objects must have all three keys
- No typos in instruction keys
- Write value must be 0 or 1 character

### Related Files

- `src/examples/*.yaml` - Example Turing machines using current format
- `src/examples/_template.yaml` - Template for new machines
- `index.html` - Contains user-facing documentation of the instruction format
