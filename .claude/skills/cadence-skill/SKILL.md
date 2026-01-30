---
description: Load when working with Cadence Virtuoso layout automation, SKILL/SKILL++ code, pcell creation, or GDS file processing.
---

# Cadence SKILL Programming Domain

> Load this skill when working with Cadence Virtuoso layout automation, SKILL code development, pcell creation, or GDS file processing tasks.

## Overview

This domain skill provides guidance for developing SKILL code in Cadence Virtuoso environments. SKILL is a LISP-based language used for layout automation, custom cell generation, and EDA tool customization.

## Core Capabilities

- **Layout Automation**: Programmatic creation of layout geometries (rectangles, paths, polygons, vias)
- **GDS Conversion**: Converting GDS/OASIS files to SKILL code for reproducible layout generation
- **Pcell Development**: Creating parameterized cells with dynamic geometry and DRC integration
- **Design Rule Checking**: Integrating DRC checks into automated layout flows
- **PDK Integration**: Working with foundry PDKs, layer mapping, and technology files

## Do NOT Build

The following components should use existing tools rather than custom implementations:

| Task | Use Instead | Reason |
|------|-------------|--------|
| GDS file parsing | gdspy (Python) or klayout (Python API) | Robust, well-tested libraries with complete GDS spec support |
| GDS hierarchy extraction | klayout.db module | Native support for cell hierarchy, references, and transformations |
| Layer number extraction | gdspy.GdsLibrary or klayout.Layout | Efficient access to layer/datatype information |
| Geometric operations | SKILL built-in functions (dbCreateRect, dbCreatePath, etc.) | Native to Cadence, optimized for database operations |
| Coordinate transformations | SKILL transformation functions (dbCreateTransform, etc.) | Integrated with Virtuoso coordinate system |
| File I/O for GDS | gdspy/klayout export functions | Handle binary format correctly |

## Do Build

Focus custom development on these team-specific and integration tasks:

1. **GDS-to-SKILL Conversion Scripts**
   - Python scripts that parse GDS using gdspy/klayout
   - Generate SKILL code templates from extracted geometry
   - Map GDS layer numbers to PDK layer/purpose pairs
   - Handle parameterization opportunities (repeated structures, scalable dimensions)

2. **Team-Specific Pcell Libraries**
   - Custom parameterized cells for project-specific devices
   - Standard cell variants with configurable options
   - Guard ring generators with corner stitching
   - Array generators with programmable pitch and alignment

3. **DRC Integration Wrappers**
   - SKILL functions that call Assura/Calibre/PVS from within pcells
   - Automated spacing checks during geometry generation
   - Design rule caching for performance
   - Error reporting integrated with CIW

4. **Layout Template Generators**
   - Code generation for repetitive patterns
   - Parameterized placement engines
   - Routing pattern libraries
   - Standard subcell instantiation patterns

5. **Testing and Validation Tools**
   - SKILL test harnesses for pcell validation
   - Geometry comparison utilities
   - Parameter sweep generators

## Sub-Skill Routing

Load additional sub-skills based on the specific task:

| Sub-Skill | Path | When to Load |
|-----------|------|--------------|
| GDS-to-SKILL Conversion | `gds-to-skill/SKILL.md` | Converting existing GDS layouts to SKILL code, extracting hierarchy, layer mapping |
| Code Organization | `code-organization/SKILL.md` | Structuring SKILL files, naming conventions, team coding standards |
| Pcell Patterns | `pcell-patterns/SKILL.md` | Creating parameterized cells, CDF definitions, callback functions |

## Typical Workflow Order

Follow this sequence for most SKILL development tasks:

### 1. Requirements Gathering
- Identify device/layout specifications
- Determine required parameters (widths, lengths, finger counts, etc.)
- Review applicable design rules from PDK
- Check for existing pcells or reference layouts

### 2. Reference Analysis (if converting existing layout)
- Load GDS file into Python (gdspy/klayout)
- Extract cell hierarchy and dependencies
- Identify layer usage and map to PDK layers
- Analyze geometry patterns for parameterization opportunities
- **Load sub-skill**: `gds-to-skill/SKILL.md`

### 3. Code Structure Setup
- Create appropriately named SKILL files
- Set up standard file structure (header, body, footer)
- Define parameter lists and default values
- Plan function organization
- **Load sub-skill**: `code-organization/SKILL.md`

### 4. Pcell Development
- Implement pcell skeleton with SKILL++/SKILL syntax
- Create CDF parameter definitions
- Implement callback functions for dynamic updates
- Add geometry generation logic
- Implement parameterization logic (scaling, arrays, conditionals)
- **Load sub-skill**: `pcell-patterns/SKILL.md`

### 5. Geometry Implementation
- Use SKILL built-ins for shape creation:
  - `dbCreateRect()` for rectangles
  - `dbCreatePath()` for paths
  - `dbCreatePolygon()` for complex shapes
  - `dbCreateInst()` for instances
  - `dbCreateVia()` for vias
- Apply transformations for placement
- Handle coordinate systems (microns vs database units)

### 6. DRC Integration
- Add inline design rule checks
- Implement spacing verification
- Add enclosure/overlap checks
- Provide meaningful error messages

### 7. Testing and Validation
- Test with parameter extremes (min/max values)
- Verify DRC cleanliness
- Check connectivity (LVS)
- Test in multiple process corners if applicable

### 8. Documentation
- Add header comments with usage examples
- Document parameters and valid ranges
- Include design rule references
- Add change history

## SKILL Language Essentials

### Basic Syntax Patterns

```lisp
; Variable assignment
let( (varName value)
    ; code using varName
)

; Conditionals
if( condition
    then-expression
    else-expression
)

; Iteration
foreach( item list
    ; process item
)

; Function definition
procedure( functionName(arg1 arg2)
    ; function body
    returnValue
)
```

### Common Database Operations

```lisp
; Create rectangle
dbCreateRect(cvId layer list(xmin ymin xmax ymax))

; Create path
dbCreatePath(cvId layer pathPoints width)

; Create instance
dbCreateInst(cvId masterCellId name xy orient)

; Create via
dbCreateVia(cvId viaDefId xy)

; Get layer-purpose pair
list(layer purpose)  ; e.g., list("M1" "drawing")
```

### Coordinate Handling

```lisp
; Points
list(x y)  ; or x:y in some contexts

; Bounding boxes
list(list(xmin ymin) list(xmax ymax))

; Transformations
dbCreateTransform(dx dy orientation magnification)
```

## PDK Integration

### Layer Mapping
- GDS layer numbers must map to PDK layer/purpose pairs
- Typical mapping: `GDS (layer, datatype)` → `PDK (layerName, purpose)`
- Common purposes: `"drawing"`, `"pin"`, `"label"`, `"boundary"`
- Store mappings in a centralized lookup table

### Technology File References
- Access technology file: `techGetTechFile(libId)`
- Get layer info: `techFindLayer(techFileId layerName)`
- Verify layer exists before use

## Best Practices

1. **Parameterization Philosophy**
   - Expose high-level parameters (width, length, fingers)
   - Hide low-level details (individual rectangle coordinates)
   - Provide sensible defaults
   - Validate parameters before use

2. **Error Handling**
   - Check parameter validity at function entry
   - Provide informative error messages
   - Use `error()` or `warn()` appropriately
   - Return nil on failure

3. **Performance**
   - Minimize database queries
   - Cache repeated calculations
   - Use batch operations when available
   - Profile code for bottlenecks

4. **Maintainability**
   - Follow team naming conventions
   - Comment non-obvious logic
   - Keep functions focused and short
   - Use meaningful variable names

5. **Version Control**
   - Track SKILL files in git
   - Include version info in file headers
   - Document breaking changes
   - Tag stable releases

## Common Pitfalls

- **Coordinate system confusion**: SKILL uses microns by default, verify units
- **Layer name typos**: Layer names are case-sensitive strings
- **Missing DRC checks**: Always verify spacing/enclosure rules
- **Hardcoded dimensions**: Parameterize everything that might change
- **Instance placement without checking**: Verify master cell exists
- **Forgetting to save cvId**: Always capture the cellview ID
- **Mixing list operations**: `list()` vs `quote()` behavior differs

## Resources and References

- SKILL Language Reference (Cadence documentation)
- SKILL++ Programming Guide
- PDK-specific layer documentation
- Team code review guidelines
- Example pcell libraries in `/examples` directory

## Integration with Other Tools

- **Python**: Use for GDS parsing, preprocessing, external calculations
- **gdspy**: Python GDS library, excellent for reading/analysis
- **klayout**: Python API for complex GDS operations, DRC, layer operations
- **Calibre/Assura**: DRC/LVS tool integration via SKILL APIs
- **Ocean**: For simulation setup and post-processing

## Notes

- SKILL is case-sensitive
- Parentheses must be balanced (LISP heritage)
- Comments start with semicolon `;`
- File extension is typically `.il` for SKILL code
- Load files with `load("filename.il")` or via CDS.lib setup
