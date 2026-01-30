---
description: Load when structuring SKILL code files, applying naming conventions, or organizing Cadence SKILL projects.
---

# SKILL Code Organization Sub-Skill

> Load this sub-skill when structuring SKILL files, establishing naming conventions, organizing code layout, or implementing team coding standards.

## Purpose

Establish consistent code organization patterns for maintainable, readable, and team-friendly SKILL code. Proper organization enables code reuse, simplifies debugging, and facilitates collaboration.

## File Naming Conventions

### Pcell Files

Use the pattern: `pcc_<device>_<variant>.il`

Examples:
- `pcc_nmos_core.il` - Core NMOS transistor pcell
- `pcc_nmos_io.il` - I/O NMOS transistor
- `pcc_resistor_poly.il` - Poly resistor
- `pcc_capacitor_mom.il` - Metal-oxide-metal capacitor
- `pcc_inductor_spiral.il` - Spiral inductor

### Utility/Helper Files

Use descriptive names: `<functionality>_utils.il`

Examples:
- `layout_utils.il` - General layout helper functions
- `drc_checks.il` - DRC checking functions
- `layer_map.il` - Layer mapping definitions
- `placement_utils.il` - Instance placement helpers

### Library Initialization Files

- `<libname>Init.il` - Library initialization (auto-loaded by Virtuoso)
- `<libname>Menu.il` - Custom menu definitions

### Version Control

Include version info in filename for major releases:
- `pcc_nmos_core_v2.il` - Major version change
- Use git tags for detailed version tracking

## Standard File Structure

Every SKILL pcell file should follow this template:

```lisp
;; ============================================================================
;; File: pcc_device_variant.il
;; Description: Brief description of the pcell
;; Author: Your Name
;; Created: YYYY-MM-DD
;; Modified: YYYY-MM-DD
;; Version: 1.0
;;
;; Parameters:
;;   - param1: Description (units)
;;   - param2: Description (units)
;;
;; Dependencies:
;;   - Required library files
;;   - External utilities
;;
;; Change History:
;;   v1.0 (YYYY-MM-DD): Initial release
;; ============================================================================

;; ============================================================================
;; SECTION 1: GLOBAL VARIABLES AND CONSTANTS
;; ============================================================================

; Design rule constants (from PDK)
_deviceName_minWidth     = 0.150      ; Minimum device width (um)
_deviceName_minLength    = 0.050      ; Minimum device length (um)
_deviceName_minSpacing   = 0.100      ; Minimum spacing (um)
_deviceName_contactEnc   = 0.040      ; Contact enclosure (um)

; Layer definitions (from PDK)
_layerDiffusion = list("DIFF" "drawing")
_layerPoly      = list("POLY" "drawing")
_layerMetal1    = list("M1" "drawing")
_layerContact   = list("CONT" "drawing")

;; ============================================================================
;; SECTION 2: HELPER FUNCTIONS
;; ============================================================================

; Helper function: Calculate derived parameters
procedure(_deviceName_calcParams(width length fingers)
    let((totalWidth fingerWidth spacing)

        ; Calculations
        fingerWidth = width / fingers
        spacing = 0.200  ; Example
        totalWidth = fingers * fingerWidth + (fingers - 1) * spacing

        ; Return as list
        list(totalWidth fingerWidth spacing)
    )
)

; Helper function: Create guard ring
procedure(_deviceName_createGuardRing(cvId centerX centerY innerW innerH width layer)
    let((x1 y1 x2 y2)

        ; Calculate coordinates
        x1 = centerX - innerW/2.0 - width
        y1 = centerY - innerH/2.0 - width
        x2 = centerX + innerW/2.0 + width
        y2 = centerY + innerH/2.0 + width

        ; Create four rectangles forming ring
        ; Bottom
        dbCreateRect(cvId layer list(x1:y1 x2:(y1+width)))
        ; Top
        dbCreateRect(cvId layer list(x1:(y2-width) x2:y2))
        ; Left
        dbCreateRect(cvId layer list(x1:y1 (x1+width):y2))
        ; Right
        dbCreateRect(cvId layer list((x2-width):y1 x2:y2))

        t  ; Return success
    )
)

;; ============================================================================
;; SECTION 3: MAIN PCELL PROCEDURE
;; ============================================================================

procedure(deviceName_createLayout(cvId @key
    (width 1.0)           ; Device width (um)
    (length 0.5)          ; Device length (um)
    (fingers 1)           ; Number of fingers
    (guardRing nil)       ; Add guard ring (t/nil)
    (contactRows 2)       ; Number of contact rows
    )

    ; Let block: declare local variables
    let((
        ; Derived parameters
        totalWidth
        fingerWidth
        fingerSpacing
        deviceHeight

        ; Geometry variables
        x y
        rectId
        pinId

        ; Temporary storage
        tempList
        )

        ;; ------------------------------------------------
        ;; SUBSECTION 3.1: Parameter Validation
        ;; ------------------------------------------------

        ; Validate width
        when(width < _deviceName_minWidth
            error("Width %.3f is below minimum %.3f" width _deviceName_minWidth)
        )

        ; Validate length
        when(length < _deviceName_minLength
            error("Length %.3f is below minimum %.3f" length _deviceName_minLength)
        )

        ; Validate fingers
        when(fingers < 1
            error("Finger count must be >= 1")
        )

        ;; ------------------------------------------------
        ;; SUBSECTION 3.2: Calculate Derived Parameters
        ;; ------------------------------------------------

        tempList = _deviceName_calcParams(width length fingers)
        totalWidth = car(tempList)
        fingerWidth = cadr(tempList)
        fingerSpacing = caddr(tempList)

        deviceHeight = length + 2 * _deviceName_contactEnc

        ;; ------------------------------------------------
        ;; SUBSECTION 3.3: Create Device Geometry
        ;; ------------------------------------------------

        ; Origin at center for symmetry
        x = 0.0
        y = 0.0

        ; Create diffusion region
        foreach(i (0 fingers-1)
            let((xPos)
                xPos = x + i * (fingerWidth + fingerSpacing) - totalWidth/2.0

                dbCreateRect(cvId _layerDiffusion
                    list(xPos : (y - deviceHeight/2.0)
                         (xPos + fingerWidth) : (y + deviceHeight/2.0))
                )
            )
        )

        ; Create poly gate (spans all fingers)
        dbCreateRect(cvId _layerPoly
            list((x - totalWidth/2.0 - 0.1) : (y - length/2.0)
                 (x + totalWidth/2.0 + 0.1) : (y + length/2.0))
        )

        ; Create contacts
        foreach(i (0 fingers-1)
            let((xPos)
                xPos = x + i * (fingerWidth + fingerSpacing) - totalWidth/2.0 + fingerWidth/2.0

                ; Source contacts
                foreach(row (0 contactRows-1)
                    dbCreateVia(cvId
                        list(lib "CONT_STD")
                        (xPos : (y - length/2.0 - 0.1 - row * 0.2)))
                )

                ; Drain contacts
                foreach(row (0 contactRows-1)
                    dbCreateVia(cvId
                        list(lib "CONT_STD")
                        (xPos : (y + length/2.0 + 0.1 + row * 0.2)))
                )
            )
        )

        ;; ------------------------------------------------
        ;; SUBSECTION 3.4: Create Pins
        ;; ------------------------------------------------

        ; Source pin
        pinId = dbCreateLabel(cvId list("M1" "pin")
                             (x : (y - length/2.0 - 0.2))
                             "S" "centerCenter" "R0" "roman" 0.1)

        ; Drain pin
        pinId = dbCreateLabel(cvId list("M1" "pin")
                             (x : (y + length/2.0 + 0.2))
                             "D" "centerCenter" "R0" "roman" 0.1)

        ; Gate pin
        pinId = dbCreateLabel(cvId list("POLY" "pin")
                             ((x - totalWidth/2.0 - 0.2) : y)
                             "G" "centerCenter" "R0" "roman" 0.1)

        ;; ------------------------------------------------
        ;; SUBSECTION 3.5: Optional Guard Ring
        ;; ------------------------------------------------

        when(guardRing
            _deviceName_createGuardRing(cvId x y
                totalWidth deviceHeight
                0.5 _layerDiffusion)
        )

        ;; ------------------------------------------------
        ;; SUBSECTION 3.6: Design Rule Checks
        ;; ------------------------------------------------

        ; Check spacing between fingers
        when((fingerSpacing < _deviceName_minSpacing)
            warn("Finger spacing %.3f is below recommended %.3f"
                 fingerSpacing _deviceName_minSpacing)
        )

        ; Final save
        dbSave(cvId)

    ) ; End let
) ; End procedure

;; ============================================================================
;; SECTION 4: CDF CALLBACK FUNCTIONS
;; ============================================================================

; Callback: Update parameters when width changes
procedure(deviceName_widthCallback(param)
    let((width fingers fingerWidth)

        width = cdfgData->width->value
        fingers = cdfgData->fingers->value

        ; Recalculate finger width
        fingerWidth = width / fingers

        ; Update display
        cdfgData->fingerWidth->value = fingerWidth

        t
    )
)

; Callback: Validate parameters before generating
procedure(deviceName_validateCallback()
    let((width length fingers valid)

        width = cdfgData->width->value
        length = cdfgData->length->value
        fingers = cdfgData->fingers->value

        valid = t

        ; Check width
        when(width < _deviceName_minWidth
            cdfgData->width->error = sprintf(nil "Width below minimum %.3f" _deviceName_minWidth)
            valid = nil
        )

        ; Check length
        when(length < _deviceName_minLength
            cdfgData->length->error = sprintf(nil "Length below minimum %.3f" _deviceName_minLength)
            valid = nil
        )

        ; Check fingers
        when(fingers < 1
            cdfgData->fingers->error = "Must be >= 1"
            valid = nil
        )

        valid
    )
)

;; ============================================================================
;; SECTION 5: INITIALIZATION AND REGISTRATION
;; ============================================================================

; Register the pcell (typically done in library init file)
procedure(register_deviceName_pcell(libName)
    let((cellName cvId)

        cellName = "deviceName"

        ; Create the pcell master if it doesn't exist
        cvId = dbOpenCellView(libName cellName "layout" "maskLayout" "w")

        ; Register parameters and callbacks with CDF
        ; (CDF setup typically in separate file or GUI)

        dbClose(cvId)

        printf("Registered pcell: %s\n" cellName)
    )
)

;; ============================================================================
;; END OF FILE
;; ============================================================================
```

## Coding Style Rules

### Indentation

- **Use 4 spaces per indentation level** (not tabs)
- Align closing parentheses with the opening statement
- Indent continuation lines by 4 additional spaces

```lisp
; Correct
procedure(myFunction(arg1 arg2)
    let((var1 var2)
        var1 = arg1 + arg2
        var2 = arg1 * arg2
        var1 + var2
    )
)

; Incorrect - inconsistent indentation
procedure(myFunction(arg1 arg2)
  let((var1 var2)
      var1 = arg1 + arg2
    var2 = arg1 * arg2
  var1 + var2
  )
)
```

### Spacing

- Space after commas in lists: `list(1 2 3)` not `list(1,2,3)`
- Space around operators: `x + y` not `x+y`
- No space between function name and parenthesis: `func(arg)` not `func (arg)`
- Space after keywords: `if( condition` not `if(condition`

### Line Length

- Maximum 100 characters per line
- Break long expressions across multiple lines

```lisp
; Break long function calls
dbCreateRect(cvId
    list("M1" "drawing")
    list(xmin:ymin xmax:ymax))

; Break long calculations
totalArea = fingerWidth * deviceLength +
            contactWidth * contactHeight +
            guardRingArea
```

## Commenting Conventions

### File Header Comments

Always include at top of file (see template above):
- File name
- Description
- Author
- Dates (created, modified)
- Version
- Parameter list
- Dependencies
- Change history

### Section Comments

Use large comment blocks to divide file into sections:

```lisp
;; ============================================================================
;; SECTION NAME IN CAPS
;; ============================================================================
```

### Subsection Comments

```lisp
;; ------------------------------------------------
;; Subsection Name
;; ------------------------------------------------
```

### Inline Comments

```lisp
; Single-line comment explaining the next statement
statement

; Multi-line comment explaining complex logic:
; Line 1 of explanation
; Line 2 of explanation
complex_statement
```

### Function Documentation

Document every function:

```lisp
; Function: calculateSpacing
; Description: Calculates minimum spacing between devices based on DRC rules
; Arguments:
;   deviceType - String: "nmos", "pmos", "resistor", etc.
;   width1 - Float: Width of first device (um)
;   width2 - Float: Width of second device (um)
; Returns:
;   Float: Minimum spacing in microns
; Example:
;   spacing = calculateSpacing("nmos" 1.0 1.5)
procedure(calculateSpacing(deviceType width1 width2)
    ; Implementation
)
```

## Variable Naming

### Naming Conventions

| Type | Convention | Example |
|------|-----------|---------|
| Local variables | camelCase | `fingerWidth`, `totalArea` |
| Global variables | _underscorePrefix | `_libName`, `_minWidth` |
| Constants | _CAPS_WITH_UNDERSCORES | `_PI`, `_MAX_FINGERS` |
| Parameters | camelCase | `deviceWidth`, `fingerCount` |
| Functions | camelCase or prefix_name | `calculateArea`, `nmos_create` |
| Private helpers | _prefix_camelCase | `_nmos_calcParams` |

### Descriptive Names

Use descriptive names that convey meaning:

```lisp
; Good
fingerWidth = deviceWidth / fingerCount
totalLength = deviceLength + 2 * contactExtension

; Bad
w = dw / fc
tl = dl + 2 * ce
```

### Avoid Ambiguity

```lisp
; Ambiguous
x = 1.0
y = 2.0

; Clear
deviceCenterX = 1.0
deviceCenterY = 2.0
```

## Function Organization

### Order Functions Logically

1. Helper functions first (private, prefixed with `_`)
2. Main functions
3. Callback functions
4. Initialization/registration functions

### Keep Functions Small

- Each function should do one thing
- Aim for < 50 lines per function
- Extract complex logic into helper functions

```lisp
; Instead of one large function
procedure(createDevice(cvId width length)
    ; 200 lines of mixed logic
)

; Break into focused functions
procedure(createDevice(cvId width length)
    _createDiffusion(cvId width length)
    _createPoly(cvId width length)
    _createContacts(cvId width length)
    _createPins(cvId)
)
```

## Error Handling

### Parameter Validation

Always validate parameters at function entry:

```lisp
procedure(createDevice(width length)
    ; Validate width
    when(width < 0.1
        error("Width %.3f below minimum 0.1um" width)
    )

    ; Validate length
    when(length < 0.05
        error("Length %.3f below minimum 0.05um" length)
    )

    ; Validate length is numeric
    unless(numberp(length)
        error("Length must be numeric, got %L" length)
    )

    ; Continue with creation
    ...
)
```

### Use warn() for Non-Fatal Issues

```lisp
; Warning for suboptimal but legal values
when(fingerCount > 20
    warn("Finger count %d is very high, may impact performance" fingerCount)
)
```

### Return Values

- Return `t` for success, `nil` for failure
- Return meaningful values (calculated results, created objects)
- Document return values in function header

```lisp
procedure(createOptionalFeature(cvId enable)
    if(enable then
        ; Create feature
        t  ; Success
    else
        nil  ; Feature not created
    )
)
```

## Best Practices Summary

1. **Consistency**: Follow the established patterns throughout all files
2. **Documentation**: Comment the "why", not the "what"
3. **Modularity**: Break complex tasks into small functions
4. **Validation**: Check parameters early and fail fast
5. **Readability**: Write code for humans first, computers second
6. **Version Control**: Use git, write meaningful commit messages
7. **Testing**: Test edge cases and parameter limits
8. **Collaboration**: Follow team conventions for easier code review

## File Organization in Repository

```
project/
├── skill/
│   ├── pcells/
│   │   ├── pcc_nmos_core.il
│   │   ├── pcc_pmos_core.il
│   │   ├── pcc_resistor_poly.il
│   │   └── ...
│   ├── utils/
│   │   ├── layout_utils.il
│   │   ├── drc_checks.il
│   │   ├── layer_map.il
│   │   └── ...
│   ├── init/
│   │   ├── myLibInit.il
│   │   └── myLibMenu.il
│   └── tests/
│       ├── test_nmos.il
│       └── test_utils.il
└── docs/
    ├── pcell_guide.md
    └── design_rules.md
```

## Loading and Testing

```lisp
; Load utility files first
load("utils/layer_map.il")
load("utils/layout_utils.il")

; Load pcells
load("pcells/pcc_nmos_core.il")

; Test the pcell
let((cvId lib)
    lib = "testLib"
    cvId = dbOpenCellView(lib "test_nmos" "layout" "maskLayout" "w")

    deviceName_createLayout(cvId
        ?width 2.0
        ?length 0.18
        ?fingers 4
        ?guardRing t)

    dbClose(cvId)
)
```
