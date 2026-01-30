# Pcell Patterns Sub-Skill

> Load this sub-skill when creating parameterized cells (pcells), defining CDF parameters, implementing callback functions, or working with common parameterization patterns in Cadence Virtuoso.

## Purpose

Provide templates and patterns for creating robust, parameterized cells that support dynamic updates, parameter validation, and design rule checking. Pcells enable reusable, configurable layout components.

## Pcell Architecture Overview

A complete pcell implementation consists of:

1. **Geometry Generation Function** - SKILL code that creates layout shapes
2. **CDF Parameters** - Component Description Format definitions for GUI
3. **Callback Functions** - Code triggered when parameters change
4. **Validation Logic** - Parameter checking and design rule verification
5. **Compilation Function** - Generates final layout from parameters

## SKILL++ Pcell Skeleton

SKILL++ provides a modern object-oriented approach to pcells:

```lisp
;; ============================================================================
;; SKILL++ Pcell Template
;; ============================================================================

; Load SKILL++ library
load(strcat(getShellEnvVar("CDSHOME") "/tools/dfII/samples/artist/leBindKeys.il"))

; Define the pcell class
pcDefinePCell(
    list(ddGetObj("myLib") "myDevice" "layout")  ; Library, cell, view

    ; Parameter list with types and defaults
    (
        (width          "float"  1.0)     ; Device width (um)
        (length         "float"  0.5)     ; Device length (um)
        (fingers        "int"    2)       ; Number of fingers
        (contactRows    "int"    2)       ; Contact rows per finger
        (guardRing      "boolean" nil)    ; Add guard ring
        (orientation    "string" "R0")    ; Orientation: R0, R90, R180, R270
    )

    ; Let block for local variables
    let((
        cvId            ; Cellview ID
        fingerWidth     ; Calculated finger width
        totalWidth      ; Total device width
        spacing         ; Finger spacing

        ; Layer definitions
        layerDiff
        layerPoly
        layerCont
        layerM1

        ; Design rules
        minWidth
        minLength
        minSpacing
        )

        ; Get cellview ID
        cvId = pcCellView

        ; Get layer IDs from tech file
        layerDiff = list("DIFF" "drawing")
        layerPoly = list("POLY" "drawing")
        layerCont = list("CONT" "drawing")
        layerM1   = list("M1" "drawing")

        ; Define design rules (from PDK)
        minWidth   = 0.150
        minLength  = 0.050
        minSpacing = 0.120

        ;; ------------------------------------------------
        ;; Parameter Validation
        ;; ------------------------------------------------

        ; Validate width
        when(width < minWidth
            error("Width %.3fum below minimum %.3fum" width minWidth)
        )

        ; Validate length
        when(length < minLength
            error("Length %.3fum below minimum %.3fum" length minLength)
        )

        ; Validate fingers
        when(fingers < 1
            error("Finger count must be >= 1")
        )

        ; Validate contact rows
        when(contactRows < 1
            error("Contact rows must be >= 1")
        )

        ;; ------------------------------------------------
        ;; Calculate Derived Parameters
        ;; ------------------------------------------------

        fingerWidth = width / fingers
        spacing = 0.200  ; 200nm spacing between fingers
        totalWidth = fingers * fingerWidth + (fingers - 1) * spacing

        ; Check calculated spacing
        when(spacing < minSpacing
            warn("Calculated spacing %.3fum below recommended %.3fum"
                 spacing minSpacing)
        )

        ;; ------------------------------------------------
        ;; Create Geometry
        ;; ------------------------------------------------

        ; Create diffusion fingers
        foreach(i (0 fingers-1)
            let((xPos xLeft xRight yBot yTop)
                ; Calculate position for this finger
                xPos = i * (fingerWidth + spacing) - totalWidth/2.0
                xLeft = xPos
                xRight = xPos + fingerWidth
                yBot = -length/2.0 - 0.3  ; Extension for contacts
                yTop = length/2.0 + 0.3

                ; Create diffusion rectangle
                dbCreateRect(cvId layerDiff
                    list(xLeft:yBot xRight:yTop))
            )
        )

        ; Create poly gate (crosses all fingers)
        let((polyLeft polyRight polyBot polyTop)
            polyLeft = -totalWidth/2.0 - 0.1
            polyRight = totalWidth/2.0 + 0.1
            polyBot = -length/2.0
            polyTop = length/2.0

            dbCreateRect(cvId layerPoly
                list(polyLeft:polyBot polyRight:polyTop))
        )

        ; Create contacts
        foreach(i (0 fingers-1)
            let((xPos)
                xPos = i * (fingerWidth + spacing) - totalWidth/2.0 + fingerWidth/2.0

                ; Source side contacts
                foreach(row (0 contactRows-1)
                    let((yPos)
                        yPos = -length/2.0 - 0.15 - row * 0.18
                        dbCreateRect(cvId layerCont
                            list((xPos-0.065):(yPos-0.065)
                                 (xPos+0.065):(yPos+0.065)))
                    )
                )

                ; Drain side contacts
                foreach(row (0 contactRows-1)
                    let((yPos)
                        yPos = length/2.0 + 0.15 + row * 0.18
                        dbCreateRect(cvId layerCont
                            list((xPos-0.065):(yPos-0.065)
                                 (xPos+0.065):(yPos+0.065)))
                    )
                )
            )
        )

        ; Create metal1 over contacts
        foreach(i (0 fingers-1)
            let((xPos xLeft xRight)
                xPos = i * (fingerWidth + spacing) - totalWidth/2.0 + fingerWidth/2.0
                xLeft = xPos - 0.1
                xRight = xPos + 0.1

                ; Source metal
                dbCreateRect(cvId layerM1
                    list(xLeft:(-length/2.0 - 0.4)
                         xRight:(-length/2.0 - 0.1)))

                ; Drain metal
                dbCreateRect(cvId layerM1
                    list(xLeft:(length/2.0 + 0.1)
                         xRight:(length/2.0 + 0.4)))
            )
        )

        ; Create pins
        dbCreateLabel(cvId list("M1" "pin")
            0.0:(-length/2.0 - 0.5)
            "S" "centerCenter" "R0" "roman" 0.1)

        dbCreateLabel(cvId list("M1" "pin")
            0.0:(length/2.0 + 0.5)
            "D" "centerCenter" "R0" "roman" 0.1)

        dbCreateLabel(cvId list("POLY" "pin")
            (totalWidth/2.0 + 0.2):0.0
            "G" "centerCenter" "R0" "roman" 0.1)

        ; Optional guard ring
        when(guardRing
            let((ringInset ringWidth)
                ringInset = 0.5
                ringWidth = 0.3

                ; Bottom rectangle
                dbCreateRect(cvId layerDiff
                    list((-totalWidth/2.0 - ringInset - ringWidth):(-length/2.0 - 0.6 - ringInset - ringWidth)
                         (totalWidth/2.0 + ringInset + ringWidth):(-length/2.0 - 0.6 - ringInset)))

                ; Top rectangle
                dbCreateRect(cvId layerDiff
                    list((-totalWidth/2.0 - ringInset - ringWidth):(length/2.0 + 0.6 + ringInset)
                         (totalWidth/2.0 + ringInset + ringWidth):(length/2.0 + 0.6 + ringInset + ringWidth)))

                ; Left rectangle
                dbCreateRect(cvId layerDiff
                    list((-totalWidth/2.0 - ringInset - ringWidth):(-length/2.0 - 0.6 - ringInset)
                         (-totalWidth/2.0 - ringInset):(length/2.0 + 0.6 + ringInset)))

                ; Right rectangle
                dbCreateRect(cvId layerDiff
                    list((totalWidth/2.0 + ringInset):(- length/2.0 - 0.6 - ringInset)
                         (totalWidth/2.0 + ringInset + ringWidth):(length/2.0 + 0.6 + ringInset)))
            )
        )

    ) ; End let
) ; End pcDefinePCell
```

## CDF Parameter Definitions

CDF (Component Description Format) defines the GUI interface for pcells. While often created via GUI, you can script it:

```lisp
;; ============================================================================
;; CDF Parameter Setup for Pcell
;; ============================================================================

procedure(setupCDF_myDevice(libName cellName)
    let((cdfId paramList)

        ; Get or create CDF data
        cdfId = cdfGetBaseCellCDF(ddGetObj(libName cellName "layout"))

        ; If doesn't exist, create it
        when(cdfId == nil
            cdfId = cdfCreateBaseCellCDF(ddGetObj(libName cellName "layout"))
        )

        ;; ------------------------------------------------
        ;; Define Parameters
        ;; ------------------------------------------------

        ; Width parameter
        cdfCreateParam(cdfId
            ?name           "width"
            ?type           "float"
            ?defValue       1.0
            ?prompt         "Device Width"
            ?units          "lengthMetric"
            ?callback       "widthCallback()"
            ?display        "artParameterInToolDisplay('width)"
        )

        ; Length parameter
        cdfCreateParam(cdfId
            ?name           "length"
            ?type           "float"
            ?defValue       0.5
            ?prompt         "Device Length"
            ?units          "lengthMetric"
            ?callback       "lengthCallback()"
            ?display        "artParameterInToolDisplay('length)"
        )

        ; Fingers parameter (integer)
        cdfCreateParam(cdfId
            ?name           "fingers"
            ?type           "int"
            ?defValue       2
            ?prompt         "Number of Fingers"
            ?callback       "fingersCallback()"
            ?display        "artParameterInToolDisplay('fingers)"
        )

        ; Contact rows parameter
        cdfCreateParam(cdfId
            ?name           "contactRows"
            ?type           "int"
            ?defValue       2
            ?prompt         "Contact Rows"
        )

        ; Guard ring parameter (boolean)
        cdfCreateParam(cdfId
            ?name           "guardRing"
            ?type           "boolean"
            ?defValue       nil
            ?prompt         "Add Guard Ring"
            ?display        "artParameterInToolDisplay('guardRing)"
        )

        ; Orientation parameter (cyclic choice)
        cdfCreateParam(cdfId
            ?name           "orientation"
            ?type           "cyclic"
            ?defValue       "R0"
            ?choices        list("R0" "R90" "R180" "R270" "MX" "MY")
            ?prompt         "Orientation"
        )

        ; Calculated parameter (read-only, display only)
        cdfCreateParam(cdfId
            ?name           "totalWidth"
            ?type           "float"
            ?defValue       0.0
            ?prompt         "Total Width (calculated)"
            ?editable       nil
            ?display        "artParameterInToolDisplay('totalWidth)"
        )

        ;; ------------------------------------------------
        ;; Parameter Validation Rules
        ;; ------------------------------------------------

        ; Add validation for width
        cdfSaveParam(cdfId
            ?name           "width"
            ?parseAsNumber  t
            ?parseAsCEL     t
        )

        ; Save CDF
        cdfSaveCDF(cdfId)

        printf("CDF setup complete for %s/%s\n" libName cellName)
    )
)

; Example: Run CDF setup
; setupCDF_myDevice("myLib" "myDevice")
```

## Callback Function Patterns

Callbacks execute when parameters change, enabling dynamic updates:

### Basic Parameter Update Callback

```lisp
; Callback triggered when width changes
procedure(widthCallback()
    let((width fingers totalWidth fingerWidth)

        ; Get current parameter values from CDF form
        width = cdfgData->width->value
        fingers = cdfgData->fingers->value

        ; Calculate derived values
        fingerWidth = width / fingers
        totalWidth = fingers * fingerWidth + (fingers - 1) * 0.2

        ; Update read-only display parameters
        cdfgData->fingerWidth->value = fingerWidth
        cdfgData->totalWidth->value = totalWidth

        ; Return t for success
        t
    )
)
```

### Parameter Dependency Callback

```lisp
; When finger count changes, update derived parameters
procedure(fingersCallback()
    let((width length fingers fingerWidth maxFingers)

        width = cdfgData->width->value
        length = cdfgData->length->value
        fingers = cdfgData->fingers->value

        ; Calculate finger width
        fingerWidth = width / fingers

        ; Check if finger width is too small
        when(fingerWidth < 0.15
            ; Adjust fingers to maintain minimum width
            maxFingers = floor(width / 0.15)
            cdfgData->fingers->value = maxFingers
            fingers = maxFingers

            warn("Finger width too small, reduced fingers to %d" maxFingers)
        )

        ; Update display
        cdfgData->fingerWidth->value = fingerWidth

        t
    )
)
```

### Validation Callback (Pre-check)

```lisp
; Called before layout generation to validate all parameters
procedure(validateParamsCallback()
    let((width length fingers valid errorMsg)

        width = cdfgData->width->value
        length = cdfgData->length->value
        fingers = cdfgData->fingers->value

        valid = t
        errorMsg = ""

        ; Check width against minimum
        when(width < 0.15
            errorMsg = strcat(errorMsg "Width below minimum 0.15um\n")
            cdfgData->width->error = "Below minimum"
            valid = nil
        )

        ; Check length against minimum
        when(length < 0.05
            errorMsg = strcat(errorMsg "Length below minimum 0.05um\n")
            cdfgData->length->error = "Below minimum"
            valid = nil
        )

        ; Check fingers
        when(fingers < 1
            errorMsg = strcat(errorMsg "Fingers must be >= 1\n")
            cdfgData->fingers->error = "Invalid count"
            valid = nil
        )

        ; Check finger width
        when((width / fingers) < 0.15
            errorMsg = strcat(errorMsg "Finger width too small\n")
            cdfgData->fingers->error = "Too many fingers"
            valid = nil
        )

        ; Display errors if any
        unless(valid
            error(errorMsg)
        )

        valid
    )
)
```

### Choice-Based Callback

```lisp
; Update available options based on another parameter
procedure(deviceTypeCallback()
    let((deviceType widthChoices)

        deviceType = cdfgData->deviceType->value

        ; Different width options for different device types
        case(deviceType
            ("nmos_core"
                widthChoices = list(0.5 1.0 2.0 5.0 10.0)
            )
            ("nmos_io"
                widthChoices = list(2.0 5.0 10.0 20.0)
            )
            ("pmos_core"
                widthChoices = list(0.5 1.0 2.0 5.0 10.0)
            )
        )

        ; Update width parameter choices
        cdfgData->width->choices = widthChoices
        cdfgData->width->value = car(widthChoices)  ; Set to first choice

        t
    )
)
```

## Common Parameterization Patterns

### Pattern 1: Finger Count Scaling

```lisp
; Scale device by increasing number of parallel fingers
procedure(createMultiFingerDevice(cvId width length fingers)
    let((fingerWidth spacing totalWidth)

        ; Calculate per-finger width
        fingerWidth = width / fingers
        spacing = 0.2  ; Fixed spacing
        totalWidth = fingers * fingerWidth + (fingers - 1) * spacing

        ; Create each finger
        foreach(i (0 fingers-1)
            let((xPos)
                xPos = i * (fingerWidth + spacing) - totalWidth/2.0

                ; Create finger geometry at xPos
                createSingleFinger(cvId xPos fingerWidth length)
            )
        )

        ; Connect fingers in parallel
        connectFingersParallel(cvId fingers fingerWidth spacing length)
    )
)
```

### Pattern 2: Width Scaling with Fixed Pitch

```lisp
; Scale device maintaining fixed contact/via pitch
procedure(createScalableWidth(cvId width length)
    let((contactPitch numContacts contactWidth)

        contactPitch = 0.5      ; Fixed pitch for contacts
        contactWidth = 0.13     ; Contact size

        ; Determine how many contacts fit
        numContacts = floor(width / contactPitch)

        ; Create device body
        dbCreateRect(cvId list("DIFF" "drawing")
            list(0.0:0.0 width:length))

        ; Place contacts at regular intervals
        foreach(i (0 numContacts-1)
            let((xPos)
                xPos = i * contactPitch + contactPitch/2.0

                ; Create contact
                dbCreateRect(cvId list("CONT" "drawing")
                    list((xPos - contactWidth/2.0):(length/2.0 - contactWidth/2.0)
                         (xPos + contactWidth/2.0):(length/2.0 + contactWidth/2.0)))
            )
        )
    )
)
```

### Pattern 3: Guard Ring with Corner Stitching

```lisp
; Create guard ring around device with proper corner handling
procedure(createGuardRingComplete(cvId centerX centerY innerWidth innerHeight
                                  ringWidth guardRingLayer)
    let((x1 y1 x2 y2 cornerSize)

        ; Calculate outer coordinates
        x1 = centerX - innerWidth/2.0 - ringWidth
        y1 = centerY - innerHeight/2.0 - ringWidth
        x2 = centerX + innerWidth/2.0 + ringWidth
        y2 = centerY + innerHeight/2.0 + ringWidth

        cornerSize = ringWidth  ; Square corners

        ; Bottom rail
        dbCreateRect(cvId guardRingLayer
            list(x1:y1 x2:(y1 + ringWidth)))

        ; Top rail
        dbCreateRect(cvId guardRingLayer
            list(x1:(y2 - ringWidth) x2:y2))

        ; Left rail
        dbCreateRect(cvId guardRingLayer
            list(x1:y1 (x1 + ringWidth):y2))

        ; Right rail
        dbCreateRect(cvId guardRingLayer
            list((x2 - ringWidth):y1 x2:y2))

        ; Add corner contacts for better connectivity
        ; Bottom-left corner
        dbCreateRect(cvId list("CONT" "drawing")
            list((x1 + 0.05):(y1 + 0.05)
                 (x1 + 0.18):(y1 + 0.18)))

        ; Bottom-right corner
        dbCreateRect(cvId list("CONT" "drawing")
            list((x2 - 0.18):(y1 + 0.05)
                 (x2 - 0.05):(y1 + 0.18)))

        ; Top-left corner
        dbCreateRect(cvId list("CONT" "drawing")
            list((x1 + 0.05):(y2 - 0.18)
                 (x1 + 0.18):(y2 - 0.05)))

        ; Top-right corner
        dbCreateRect(cvId list("CONT" "drawing")
            list((x2 - 0.18):(y2 - 0.18)
                 (x2 - 0.05):(y2 - 0.05)))

        t
    )
)
```

### Pattern 4: Array with Programmable Pitch

```lisp
; Create array of instances with custom pitch in X and Y
procedure(createDeviceArray(cvId masterCell rows cols pitchX pitchY originX originY)
    let((instName xPos yPos)

        foreach(row (0 rows-1)
            foreach(col (0 cols-1)
                ; Calculate position
                xPos = originX + col * pitchX
                yPos = originY + row * pitchY

                ; Generate unique instance name
                instName = sprintf(nil "%s_R%dC%d" masterCell row col)

                ; Create instance
                dbCreateInst(cvId
                    dbOpenCellView(lib masterCell "layout" "r")
                    instName
                    xPos:yPos
                    "R0")
            )
        )

        printf("Created %dx%d array of %s\n" rows cols masterCell)
        t
    )
)
```

### Pattern 5: Conditional Geometry (Options)

```lisp
; Create device with optional features based on parameters
procedure(createConfigurableDevice(cvId width length
                                   ?addGuardRing nil
                                   ?addDummyFill t
                                   ?routingStyle "direct"
                                   ?contactStyle "single")
    let((deviceBBox)

        ; Create core device
        deviceBBox = createCoreDevice(cvId width length)

        ; Optional guard ring
        when(addGuardRing
            createGuardRingComplete(cvId 0.0 0.0 width length 0.5
                                   list("DIFF" "drawing"))
        )

        ; Optional dummy fill
        when(addDummyFill
            addDummyPolyFill(cvId deviceBBox 0.3)  ; 0.3um dummy width
        )

        ; Routing style
        case(routingStyle
            ("direct"
                ; Direct metal connection
                createDirectRouting(cvId deviceBBox)
            )
            ("bussed"
                ; Bus-style routing with vias
                createBussedRouting(cvId deviceBBox)
            )
            ("custom"
                ; User-defined routing
                warn("Custom routing not implemented, using direct")
                createDirectRouting(cvId deviceBBox)
            )
        )

        ; Contact style
        case(contactStyle
            ("single"
                addSingleContacts(cvId deviceBBox)
            )
            ("array"
                addContactArray(cvId deviceBBox 0.18)  ; 0.18um pitch
            )
            ("maximum"
                addMaximumContacts(cvId deviceBBox)
            )
        )

        t
    )
)
```

### Pattern 6: Design Rule Aware Sizing

```lisp
; Automatically adjust sizes to meet design rules
procedure(createDRCAwareDevice(cvId nominalWidth nominalLength)
    let((width length minWidth minLength minSpacing snapGrid
        actualWidth actualLength)

        ; Get design rules from PDK
        minWidth = 0.15
        minLength = 0.05
        minSpacing = 0.12
        snapGrid = 0.005  ; 5nm grid

        ; Snap to grid
        width = ceiling(nominalWidth / snapGrid) * snapGrid
        length = ceiling(nominalLength / snapGrid) * snapGrid

        ; Enforce minimums
        when(width < minWidth
            width = minWidth
            warn("Width adjusted from %.3f to minimum %.3f" nominalWidth width)
        )

        when(length < minLength
            length = minLength
            warn("Length adjusted from %.3f to minimum %.3f" nominalLength length)
        )

        ; Create with validated dimensions
        actualWidth = width
        actualLength = length

        dbCreateRect(cvId list("DIFF" "drawing")
            list(0.0:0.0 actualWidth:actualLength))

        ; Return actual dimensions for caller
        list(actualWidth actualLength)
    )
)
```

## Complete Working Example: MOS Transistor Pcell

```lisp
;; ============================================================================
;; Complete NMOS Transistor Pcell Example
;; ============================================================================

pcDefinePCell(
    list(ddGetObj("analogLib") "nmos4" "layout")

    ; Parameters
    ((w "float" 1.0)          ; Width (um)
     (l "float" 0.18)         ; Length (um)
     (nf "int" 1)             ; Number of fingers
     (m "int" 1)              ; Multiplier (instances in parallel)
     (gr "boolean" nil)       ; Guard ring
    )

    let((cvId fw tw sp)

        cvId = pcCellView

        ; Validate
        when(w < 0.15 error("W < 0.15um"))
        when(l < 0.05 error("L < 0.05um"))
        when(nf < 1 error("NF < 1"))
        when(m < 1 error("M < 1"))

        ; Calculate
        fw = w / nf            ; Finger width
        sp = 0.2               ; Spacing
        tw = nf * fw + (nf - 1) * sp

        ; Create geometry
        foreach(f (0 nf-1)
            let((xp)
                xp = f * (fw + sp) - tw/2.0

                ; Diffusion
                dbCreateRect(cvId list("DIFF" "drawing")
                    list(xp:(-l/2.0 - 0.4) (xp + fw):(l/2.0 + 0.4)))
            )
        )

        ; Poly gate
        dbCreateRect(cvId list("POLY" "drawing")
            list((-tw/2.0 - 0.1):(-l/2.0) (tw/2.0 + 0.1):(l/2.0)))

        ; Guard ring if requested
        when(gr
            let((x1 y1 x2 y2 rw)
                rw = 0.5
                x1 = -tw/2.0 - 1.0
                y1 = -l/2.0 - 1.0
                x2 = tw/2.0 + 1.0
                y2 = l/2.0 + 1.0

                dbCreateRect(cvId list("DIFF" "drawing") list(x1:y1 x2:(y1+rw)))
                dbCreateRect(cvId list("DIFF" "drawing") list(x1:(y2-rw) x2:y2))
                dbCreateRect(cvId list("DIFF" "drawing") list(x1:y1 (x1+rw):y2))
                dbCreateRect(cvId list("DIFF" "drawing") list((x2-rw):y1 x2:y2))
            )
        )

    ) ; End let
) ; End pcDefinePCell
```

## Testing Pcells

```lisp
; Test function for pcell
procedure(testPcell(libName cellName)
    let((cvId testLib testCell)

        testLib = "testLib"
        testCell = "test_instance"

        ; Create test cellview
        cvId = dbOpenCellView(testLib testCell "layout" "maskLayout" "w")

        ; Create instance of pcell with various parameters
        dbCreateParamInst(cvId
            ddGetObj(libName cellName "layout")
            "I0"
            0.0:0.0
            "R0"
            1.0  ; Magnification
            list(list("width" "float" 2.0)
                 list("length" "float" 0.18)
                 list("fingers" "int" 4)
                 list("guardRing" "boolean" t)
            )
        )

        ; Save and close
        dbSave(cvId)
        dbClose(cvId)

        printf("Test pcell created in %s/%s\n" testLib testCell)
    )
)
```

## Best Practices

1. **Always validate parameters** before creating geometry
2. **Use design rules from PDK**, don't hardcode
3. **Provide sensible defaults** for all parameters
4. **Calculate derived parameters** in let block before geometry creation
5. **Comment sections** clearly (validation, calculation, geometry, pins)
6. **Test edge cases**: minimum sizes, maximum fingers, extreme ratios
7. **Add warnings** for suboptimal but legal parameters
8. **Create helper functions** for repeated patterns
9. **Use consistent naming**: width/length not w/l in user-facing params
10. **Document parameters** in CDF prompt strings

## Common Pitfalls

- Forgetting to validate parameters leads to DRC errors
- Hardcoded layer names instead of reading from tech file
- Not accounting for manufacturing grid (snap to grid)
- Missing corner cases in finger calculations
- Forgetting to update derived parameters in callbacks
- Creating geometry without checking cellview ID validity
- Not testing with parameter extremes
