# Programmatic Design & Structural FEA of an Industrial Manifold Flange Plate

Welcome to my portfolio project demonstrating the integration of programmatic 3D modeling and Finite Element Analysis (FEA) within **FreeCAD** using Python scripting and the **CalculiX** implicit static structural solver backend. 

In this project, I act as an external engineering consultant optimizing an industrial component for a high-pressure boiler system framework.

---

##  Project Architecture & Workflow
The repository contains the implementation of a full engineering pipeline from code to stress simulation verification:

1. **Geometry Automation:** A Python macro (`ParametricFlangePlate.py`) that constructs an exact solid square plate equipped with a central hollow cylinder hub and 4 corner bolt holes.
2. **FEM Mesh Generation:** Breaking down the 3D solid engine parts into parabolic tetrahedral element node grids using the **Gmsh** meshing engine framework.
3. **Boundary Condition Mapping:** Pinning the structural component's bolt holes securely (0 Degrees of Freedom) and introducing a heavy operational load vectors profile.
4. **Solver Processing:** Compiling structural geometry stiffness matrices dynamically inside the implicit structural calculation solver backend.
5. **Post-Processing & Optimization:** Visualizing Von Mises stress distributions to isolate local geometric failure zones and proposing material optimizations.

---

##  Source Code: `ParametricFlangePlate.py`

This script programmatically generates the 3D manifold model inside FreeCAD. By shifting design dimensions into variables, it allows for instant geometry rescaling based on operational pressure spikes.

```python
import FreeCAD as App
import Part

def generate_flange_plate():
    # Close any existing document with the same name to avoid collisions
    doc_name = "Industrial_Flange_Plate"
    try:
        App.closeDocument(doc_name)
    except:
        pass

    doc = App.newDocument(doc_name)
    
    # --- DESIGN VARIATION PARAMETERS ---
    plate_width = 200.0     # Square plate dimension (mm)
    plate_thickness = 12.0  # Main plate thickness (mm)
    
    hub_radius = 50.0       # Central pipe outer radius (mm)
    hub_inner_r = 40.0      # Central pipe inner radius (mm)
    hub_height = 45.0       # Extrusion height of the hub (mm)
    
    bolt_hole_r = 10.0      # Corner mounting hole radius (mm)
    hole_offset = 25.0      # Distance of bolt centers from edge (mm)

    print("[INFO] Fabricating main mounting base plate...")
    base_plate = Part.makeBox(plate_width, plate_width, plate_thickness)
    base_plate.translate(App.Vector(-plate_width/2.0, -plate_width/2.0, 0.0))

    print("[INFO] Constructing central manifold hub cylinder...")
    outer_hub = Part.makeCylinder(hub_radius, hub_height)
    inner_hub = Part.makeCylinder(hub_inner_r, hub_height)
    hollow_hub = outer_hub.cut(inner_hub)
    
    combined_body = base_plate.fuse(hollow_hub)

    print("[INFO] Boring corner structural bolt holes...")
    pos_x = (plate_width / 2.0) - hole_offset
    pos_y = (plate_width / 2.0) - hole_offset
    
    hole_coordinates = [
        App.Vector(pos_x, pos_y, -1.0),
        App.Vector(-pos_x, pos_y, -1.0),
        App.Vector(pos_x, -pos_y, -1.0),
        App.Vector(-pos_x, -pos_y, -1.0)
    ]

    for coords in hole_coordinates:
        drill_bit = Part.makeCylinder(bolt_hole_r, plate_thickness + 2.0)
        drill_bit.translate(coords)
        combined_body = combined_body.cut(drill_bit)

    structural_part = doc.addObject("Part::Feature", "Manifold_Flange_Plate")
    structural_part.Shape = combined_body

    doc.recompute()
    print("[SUCCESS] Parametric structure generated cleanly for structural analysis.")

generate_flange_plate()
```

---

##  Finite Element Analysis (FEA) Simulation Report

### 1. Mesh & Boundary Configurations
* **Material Properties:** Generic Structural Steel (Young's Modulus: 210 GPa, Poisson's Ratio: 0.30, Yield Strength: 250 MPa).
* **Mesh Discretization:** Generated via **Gmsh** with a maximum element scale constraint of `8.0 mm`.
* **Fixed Constraints:** Pinned all 4 interior cylindrical surfaces of the mounting bolt holes (0 DOFs) to simulate rigid bolted anchors.
* **Loading Profile:** Applied a static, uniform downward mechanical force of **25,000 Newtons** along the negative Z-axis across the top face of the cylinder hub.

### 2. Post-Processing Pipeline Metrics
When evaluating the model pipeline using a **VonMisesStress** color map overlay, structural stress flows reveal the following distribution profiles:

