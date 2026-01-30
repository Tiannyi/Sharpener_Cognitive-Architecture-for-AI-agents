---
description: Convert GDS/GDSII layout files to Cadence SKILL code using gdspy or klayout.
---

# GDS-to-SKILL Conversion Sub-Skill

> Load this sub-skill when converting existing GDS/OASIS layouts to SKILL code, extracting hierarchy from GDS files, or mapping GDS layer numbers to PDK conventions.

## Purpose

Convert static GDS layouts into parameterizable SKILL code for layout automation and pcell development. This enables version control, parameterization, and integration with Cadence Virtuoso design flows.

## Workflow Overview

1. **Parse GDS with Python** (gdspy or klayout)
2. **Extract hierarchy** and cell dependencies
3. **Map layers** from GDS numbers to PDK layer/purpose pairs
4. **Generate SKILL code** using templates
5. **Validate** generated code in Virtuoso

## GDS Parsing with Python

### Using gdspy

```python
import gdspy

# Load GDS file
gdsii = gdspy.GdsLibrary()
gdsii.read_gds('input_layout.gds')

# Access cells
for cell_name, cell in gdsii.cells.items():
    print(f"Cell: {cell_name}")

    # Get all polygons
    polygons = cell.get_polygons(by_spec=True)
    for (layer, datatype), polys in polygons.items():
        print(f"  Layer {layer}/{datatype}: {len(polys)} polygons")
        for poly in polys:
            print(f"    Polygon points: {poly}")

    # Get all paths
    paths = cell.get_paths()
    for path in paths:
        print(f"  Path on layer {path.layers[0]}/{path.datatypes[0]}")
        print(f"    Width: {path.widths[0]}")
        print(f"    Points: {path.polygons[0]}")

    # Get all references (instances)
    for ref in cell.references:
        if isinstance(ref, gdspy.CellReference):
            print(f"  Instance: {ref.ref_cell.name}")
            print(f"    Origin: {ref.origin}")
            print(f"    Rotation: {ref.rotation}")
            print(f"    Magnification: {ref.magnification}")
        elif isinstance(ref, gdspy.CellArray):
            print(f"  Array: {ref.ref_cell.name}")
            print(f"    Columns: {ref.columns}, Rows: {ref.rows}")
            print(f"    Spacing: {ref.spacing}")
```

### Using klayout

```python
import klayout.db as db

# Load GDS file
layout = db.Layout()
layout.read("input_layout.gds")

# Iterate through cells
for cell in layout.each_cell():
    cell_obj = layout.cell(cell)
    print(f"Cell: {cell_obj.name}")

    # Iterate through layers
    for layer_info in layout.layer_indices():
        layer = layout.get_info(layer_info)
        print(f"  Layer {layer.layer}/{layer.datatype} ({layer.name})")

        # Get shapes on this layer
        shapes = cell_obj.shapes(layer_info)

        # Rectangles
        for shape in shapes.each(db.ShapeIterator.Boxes):
            box = shape.box()
            print(f"    Rect: ({box.left}, {box.bottom}) to ({box.right}, {box.top})")

        # Polygons
        for shape in shapes.each(db.ShapeIterator.Polygons):
            poly = shape.polygon()
            pts = [f"({p.x}, {p.y})" for p in poly.each_point()]
            print(f"    Polygon: {pts}")

        # Paths
        for shape in shapes.each(db.ShapeIterator.Paths):
            path = shape.path()
            print(f"    Path width: {path.width}")
            pts = [f"({p.x}, {p.y})" for p in path.each_point()]
            print(f"    Path points: {pts}")

    # Instances
    for inst in cell_obj.each_inst():
        print(f"  Instance: {inst.cell.name}")
        trans = inst.trans
        print(f"    Translation: ({trans.disp.x}, {trans.disp.y})")
        print(f"    Rotation: {trans.angle}")
        print(f"    Mirror: {trans.is_mirror()}")
```

## Layer Mapping

### Layer Mapping Table Structure

Create a mapping from GDS layer/datatype to PDK layer/purpose:

```python
# Layer mapping dictionary
LAYER_MAP = {
    # GDS (layer, datatype): (PDK_layer, PDK_purpose)
    (1, 0): ("DIFF", "drawing"),
    (1, 1): ("DIFF", "pin"),
    (1, 2): ("DIFF", "label"),
    (2, 0): ("POLY", "drawing"),
    (2, 1): ("POLY", "pin"),
    (3, 0): ("CONT", "drawing"),
    (10, 0): ("M1", "drawing"),
    (10, 1): ("M1", "pin"),
    (10, 2): ("M1", "label"),
    (11, 0): ("VIA1", "drawing"),
    (20, 0): ("M2", "drawing"),
    (20, 1): ("M2", "pin"),
    (30, 0): ("M3", "drawing"),
    (40, 0): ("M4", "drawing"),
    (50, 0): ("M5", "drawing"),
}

def map_layer(gds_layer, gds_datatype):
    """Map GDS layer/datatype to PDK layer/purpose."""
    key = (gds_layer, gds_datatype)
    if key in LAYER_MAP:
        return LAYER_MAP[key]
    else:
        # Default behavior for unmapped layers
        return (f"GDS_{gds_layer}", f"dt_{gds_datatype}")
```

### Loading Layer Map from File

```python
import csv

def load_layer_map(csv_file):
    """Load layer mapping from CSV file.

    CSV format:
    gds_layer,gds_datatype,pdk_layer,pdk_purpose
    1,0,DIFF,drawing
    1,1,DIFF,pin
    """
    layer_map = {}
    with open(csv_file, 'r') as f:
        reader = csv.DictReader(f)
        for row in reader:
            gds_key = (int(row['gds_layer']), int(row['gds_datatype']))
            pdk_value = (row['pdk_layer'], row['pdk_purpose'])
            layer_map[gds_key] = pdk_value
    return layer_map
```

## SKILL Code Generation Templates

### Rectangle Template

```python
def generate_rect_skill(layer, purpose, bbox, indent=4):
    """Generate SKILL code for a rectangle.

    Args:
        layer: PDK layer name (string)
        purpose: PDK purpose (string)
        bbox: Bounding box as ((xmin, ymin), (xmax, ymax))
        indent: Indentation level in spaces
    """
    xmin, ymin = bbox[0]
    xmax, ymax = bbox[1]

    # Convert to microns if needed (assuming input is in database units)
    # GDS is typically in nanometers or database units
    xmin_um = xmin * 1e-3  # Convert nm to um
    ymin_um = ymin * 1e-3
    xmax_um = xmax * 1e-3
    ymax_um = ymax * 1e-3

    indent_str = " " * indent

    skill_code = f'{indent_str}dbCreateRect(cvId list("{layer}" "{purpose}") '
    skill_code += f'list({xmin_um:.4f}:{ymin_um:.4f} {xmax_um:.4f}:{ymax_um:.4f}))'

    return skill_code

# Example usage
rect_code = generate_rect_skill("M1", "drawing", ((0, 0), (1000, 500)))
print(rect_code)
# Output: dbCreateRect(cvId list("M1" "drawing") list(0.0000:0.0000 1.0000:0.5000))
```

### Path Template

```python
def generate_path_skill(layer, purpose, points, width, indent=4):
    """Generate SKILL code for a path.

    Args:
        layer: PDK layer name
        purpose: PDK purpose
        points: List of (x, y) tuples defining path centerline
        width: Path width in database units
        indent: Indentation level
    """
    indent_str = " " * indent

    # Convert points to microns
    points_um = [(x * 1e-3, y * 1e-3) for x, y in points]
    width_um = width * 1e-3

    # Format point list
    points_str = " ".join([f"{x:.4f}:{y:.4f}" for x, y in points_um])

    skill_code = f'{indent_str}dbCreatePath(cvId list("{layer}" "{purpose}") '
    skill_code += f'list({points_str}) {width_um:.4f})'

    return skill_code

# Example usage
path_code = generate_path_skill("M2", "drawing",
                                [(0, 0), (1000, 0), (1000, 2000)],
                                200)
print(path_code)
```

### Polygon Template

```python
def generate_polygon_skill(layer, purpose, points, indent=4):
    """Generate SKILL code for a polygon.

    Args:
        layer: PDK layer name
        purpose: PDK purpose
        points: List of (x, y) tuples defining polygon vertices
        indent: Indentation level
    """
    indent_str = " " * indent

    # Convert points to microns
    points_um = [(x * 1e-3, y * 1e-3) for x, y in points]

    # Format point list
    points_str = " ".join([f"{x:.4f}:{y:.4f}" for x, y in points_um])

    skill_code = f'{indent_str}dbCreatePolygon(cvId list("{layer}" "{purpose}") '
    skill_code += f'list({points_str}))'

    return skill_code
```

### Via/Contact Template

```python
def generate_via_skill(via_name, position, indent=4):
    """Generate SKILL code for a via instance.

    Args:
        via_name: Via definition name (e.g., "VIA12")
        position: (x, y) tuple for via placement
        indent: Indentation level
    """
    indent_str = " " * indent

    x_um = position[0] * 1e-3
    y_um = position[1] * 1e-3

    skill_code = f'{indent_str}dbCreateVia(cvId '
    skill_code += f'list(lib "{via_name}") {x_um:.4f}:{y_um:.4f})'

    return skill_code
```

### Instance (Cell Reference) Template

```python
def generate_instance_skill(master_cell, inst_name, position,
                           rotation=0, mirror=False, magnification=1.0,
                           indent=4):
    """Generate SKILL code for a cell instance.

    Args:
        master_cell: Name of master cell to instantiate
        inst_name: Instance name
        position: (x, y) tuple for placement
        rotation: Rotation angle in degrees (0, 90, 180, 270)
        mirror: Boolean for X-axis mirroring
        magnification: Scale factor
        indent: Indentation level
    """
    indent_str = " " * indent

    x_um = position[0] * 1e-3
    y_um = position[1] * 1e-3

    # Determine orientation string
    orient_map = {
        (0, False): "R0",
        (90, False): "R90",
        (180, False): "R180",
        (270, False): "R270",
        (0, True): "MX",
        (90, True): "MXR90",
        (180, True): "MY",
        (270, True): "MYR90",
    }
    orient = orient_map.get((rotation, mirror), "R0")

    skill_code = f'{indent_str}dbCreateInst(cvId '
    skill_code += f'dbOpenCellView(lib "{master_cell}" "layout" "layout") '
    skill_code += f'"{inst_name}" {x_um:.4f}:{y_um:.4f} "{orient}")'

    return skill_code
```

### Array Reference Template

```python
def generate_array_skill(master_cell, base_name, origin,
                        rows, cols, row_pitch, col_pitch,
                        rotation=0, mirror=False, indent=4):
    """Generate SKILL code for an array of instances.

    Args:
        master_cell: Name of master cell
        base_name: Base name for instances (will append _R#C#)
        origin: (x, y) origin of array
        rows: Number of rows
        cols: Number of columns
        row_pitch: Spacing between rows in database units
        col_pitch: Spacing between columns in database units
        rotation: Rotation angle
        mirror: Mirror flag
        indent: Indentation level
    """
    indent_str = " " * indent
    x_orig_um = origin[0] * 1e-3
    y_orig_um = origin[1] * 1e-3
    row_pitch_um = row_pitch * 1e-3
    col_pitch_um = col_pitch * 1e-3

    orient_map = {
        (0, False): "R0",
        (90, False): "R90",
        (180, False): "R180",
        (270, False): "R270",
    }
    orient = orient_map.get((rotation, mirror), "R0")

    skill_lines = []
    skill_lines.append(f"{indent_str}; Array of {master_cell}: {rows}x{cols}")
    skill_lines.append(f"{indent_str}foreach(row list(0 {rows-1})")
    skill_lines.append(f"{indent_str}    foreach(col list(0 {cols-1})")

    inst_line = f'{indent_str}        dbCreateInst(cvId '
    inst_line += f'dbOpenCellView(lib "{master_cell}" "layout" "layout") '
    inst_line += f'sprintf(instName "{base_name}_R%dC%d" row col) '
    inst_line += f'{x_orig_um:.4f}+col*{col_pitch_um:.4f}:'
    inst_line += f'{y_orig_um:.4f}+row*{row_pitch_um:.4f} "{orient}")'
    skill_lines.append(inst_line)

    skill_lines.append(f"{indent_str}    )")
    skill_lines.append(f"{indent_str})")

    return "\n".join(skill_lines)
```

## Complete GDS-to-SKILL Converter

```python
import gdspy

class GDStoSKILL:
    """Convert GDS cell to SKILL code."""

    def __init__(self, layer_map, unit_scale=1e-3):
        """Initialize converter.

        Args:
            layer_map: Dictionary mapping (gds_layer, datatype) to (pdk_layer, purpose)
            unit_scale: Scale factor from GDS units to microns (default 1e-3 for nm to um)
        """
        self.layer_map = layer_map
        self.unit_scale = unit_scale
        self.skill_lines = []

    def convert_cell(self, cell, lib_name="myLib", cell_view_name="layout"):
        """Convert a gdspy Cell to SKILL code.

        Args:
            cell: gdspy.Cell object
            lib_name: Library name for the SKILL code
            cell_view_name: Cell view name (typically "layout")

        Returns:
            String containing SKILL code
        """
        self.skill_lines = []

        # Header
        self.skill_lines.append(f'; SKILL code for cell: {cell.name}')
        self.skill_lines.append(f'; Auto-generated from GDS')
        self.skill_lines.append('')
        self.skill_lines.append(f'procedure(create_{cell.name}(lib)')
        self.skill_lines.append(f'    let((cvId)')
        self.skill_lines.append('')
        self.skill_lines.append(f'        ; Open/create cellview')
        self.skill_lines.append(f'        cvId = dbOpenCellView(lib "{cell.name}" "{cell_view_name}" "maskLayout" "w")')
        self.skill_lines.append('')

        # Process polygons
        polygons = cell.get_polygons(by_spec=True)
        for (layer, datatype), polys in polygons.items():
            if (layer, datatype) in self.layer_map:
                pdk_layer, purpose = self.layer_map[(layer, datatype)]
                self.skill_lines.append(f'        ; Layer {layer}/{datatype} -> {pdk_layer}/{purpose}')

                for poly in polys:
                    # Check if it's a rectangle (4 points, axis-aligned)
                    if len(poly) == 4 or len(poly) == 5:  # 5 if closed
                        xs = [p[0] for p in poly[:4]]
                        ys = [p[1] for p in poly[:4]]
                        if len(set(xs)) == 2 and len(set(ys)) == 2:
                            # It's a rectangle
                            xmin, xmax = min(xs) * self.unit_scale, max(xs) * self.unit_scale
                            ymin, ymax = min(ys) * self.unit_scale, max(ys) * self.unit_scale
                            self.skill_lines.append(
                                f'        dbCreateRect(cvId list("{pdk_layer}" "{purpose}") '
                                f'list({xmin:.4f}:{ymin:.4f} {xmax:.4f}:{ymax:.4f}))'
                            )
                        else:
                            # General polygon
                            pts = " ".join([f"{p[0]*self.unit_scale:.4f}:{p[1]*self.unit_scale:.4f}"
                                          for p in poly])
                            self.skill_lines.append(
                                f'        dbCreatePolygon(cvId list("{pdk_layer}" "{purpose}") '
                                f'list({pts}))'
                            )
                    else:
                        # General polygon
                        pts = " ".join([f"{p[0]*self.unit_scale:.4f}:{p[1]*self.unit_scale:.4f}"
                                      for p in poly])
                        self.skill_lines.append(
                            f'        dbCreatePolygon(cvId list("{pdk_layer}" "{purpose}") '
                            f'list({pts}))'
                        )
                self.skill_lines.append('')

        # Process paths
        for path in cell.get_paths():
            layer, datatype = path.layers[0], path.datatypes[0]
            if (layer, datatype) in self.layer_map:
                pdk_layer, purpose = self.layer_map[(layer, datatype)]
                width = path.widths[0] * self.unit_scale
                points = path.polygons[0]  # Get centerline points
                pts_str = " ".join([f"{p[0]*self.unit_scale:.4f}:{p[1]*self.unit_scale:.4f}"
                                   for p in points])
                self.skill_lines.append(f'        ; Path on {pdk_layer}/{purpose}')
                self.skill_lines.append(
                    f'        dbCreatePath(cvId list("{pdk_layer}" "{purpose}") '
                    f'list({pts_str}) {width:.4f})'
                )
                self.skill_lines.append('')

        # Process references
        for ref in cell.references:
            if isinstance(ref, gdspy.CellReference):
                x, y = ref.origin
                x_um, y_um = x * self.unit_scale, y * self.unit_scale
                rot = ref.rotation if ref.rotation else 0
                mag = ref.magnification if ref.magnification else 1.0

                self.skill_lines.append(f'        ; Instance of {ref.ref_cell.name}')
                self.skill_lines.append(
                    f'        dbCreateInst(cvId '
                    f'dbOpenCellView(lib "{ref.ref_cell.name}" "{cell_view_name}") '
                    f'"I_{ref.ref_cell.name}" {x_um:.4f}:{y_um:.4f} "R0")'
                )
                self.skill_lines.append('')

        # Footer
        self.skill_lines.append('        ; Save and close')
        self.skill_lines.append('        dbSave(cvId)')
        self.skill_lines.append('        dbClose(cvId)')
        self.skill_lines.append('    )')
        self.skill_lines.append(')')

        return "\n".join(self.skill_lines)

# Example usage
if __name__ == "__main__":
    # Define layer map
    layer_map = {
        (1, 0): ("DIFF", "drawing"),
        (10, 0): ("M1", "drawing"),
        (20, 0): ("M2", "drawing"),
    }

    # Load GDS
    gdsii = gdspy.GdsLibrary()
    gdsii.read_gds('input.gds')

    # Convert cells
    converter = GDStoSKILL(layer_map)
    for cell_name, cell in gdsii.cells.items():
        skill_code = converter.convert_cell(cell)

        # Write to file
        with open(f'{cell_name}.il', 'w') as f:
            f.write(skill_code)

        print(f"Generated {cell_name}.il")
```

## Handling Parameterization Opportunities

When converting GDS to SKILL, look for patterns that could be parameterized:

### Identifying Repeated Structures

```python
def find_repeated_patterns(cell):
    """Identify repeated geometry patterns for parameterization."""

    # Group polygons by layer
    polygons = cell.get_polygons(by_spec=True)

    # Look for arrays (repeated instances at regular intervals)
    for ref in cell.references:
        if isinstance(ref, gdspy.CellArray):
            print(f"Array found: {ref.ref_cell.name}")
            print(f"  Rows: {ref.rows}, Cols: {ref.columns}")
            print(f"  Could parameterize: row_count, col_count, pitch_x, pitch_y")

    # Look for repeated widths (could parameterize width)
    widths = set()
    for path in cell.get_paths():
        widths.add(path.widths[0])
    if len(widths) < 5:  # Few unique widths suggests parameterization opportunity
        print(f"Found {len(widths)} unique widths: {widths}")
        print("  Could parameterize: device_width")
```

## Validation

After generating SKILL code, validate in Virtuoso:

1. Load the SKILL file: `load("generated_cell.il")`
2. Execute the procedure: `create_cell_name(lib)`
3. Open the cellview and verify geometry
4. Run DRC to check for errors
5. Compare with original GDS using XOR

```lisp
; Validation helper in SKILL
procedure(validateGeneratedCell(lib cellName)
    let((cvId)
        cvId = dbOpenCellView(lib cellName "layout" "r")
        if(cvId then
            printf("Cell %s opened successfully\n" cellName)
            printf("Number of shapes: %d\n" length(cvId~>shapes))
            dbClose(cvId)
        else
            error("Failed to open cell %s\n" cellName)
        )
    )
)
```

## Best Practices

1. **Preserve hierarchy**: Convert bottom-up (leaf cells first)
2. **Verify layer mapping**: Check all GDS layers have PDK equivalents
3. **Handle units carefully**: GDS database units vary by foundry
4. **Add comments**: Include original GDS layer info in comments
5. **Test incrementally**: Validate small sections before full conversion
6. **Parameterize early**: Identify parameterization opportunities during extraction
7. **Version control**: Track both GDS source and generated SKILL code

## Common Issues

- **Unit mismatch**: Verify GDS units (nm, um, etc.) match SKILL expectations
- **Layer not found**: Unmapped GDS layers need manual intervention
- **Complex polygons**: Very complex shapes may need simplification
- **Rotated instances**: Handle all 8 orientations (R0, R90, R180, R270, MX, MY, etc.)
- **Array references**: GDS array parameters differ from SKILL array logic
