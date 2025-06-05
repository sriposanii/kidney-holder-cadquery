# kidney-holder-cadquery
A Python + CadQuery script to generate a customizable 3D kidney holder tray for holding multiple mouse kidneys, based on their unique dimensions.
import cadquery as cq
import os
import math

OUTPUT_DIR = "outputs"
os.makedirs(OUTPUT_DIR, exist_ok=True)
OUTPUT_FILE = os.path.join(OUTPUT_DIR, "multi_kidney_holder.stl")

kidneys = [
    (12, 6, 5),
    (14, 7, 6),
    (10, 5, 4),
    (13, 6.5, 5.5)
]

SPACING = 25
LAYOUT = "row"  # change to 'carousel' if preferred

def kidney_slot(length, width, height):
    padding = 2
    wall = 2
    L = length + padding + wall
    W = width + padding + wall
    H = height + 1 + wall

    block = cq.Workplane("XY").box(L, W, H)
    cavity = (
        cq.Workplane("XY")
        .moveTo(0, 0)
        .ellipse(length/2, width/2)
        .extrude(height + 1)
        .translate((0, 0, wall/2))
    )
    return block.cut(cavity)

def build_holder(kidney_list, layout="row"):
    tray = cq.Workplane("XY")
    for i, (L, W, H) in enumerate(kidney_list):
        slot = kidney_slot(L, W, H)
        if layout == "row":
            x = i * SPACING
            y = 0
        elif layout == "carousel":
            angle = i * (360 / len(kidney_list))
            x = 50 * math.cos(math.radians(angle))
            y = 50 * math.sin(math.radians(angle))
        else:
            raise ValueError("Invalid layout type.")
        tray = tray.union(slot.translate((x, y, 0)))
    return tray

holder = build_holder(kidneys, layout=LAYOUT)
cq.exporters.export(holder, OUTPUT_FILE)
print(f"✅ Exported to {OUTPUT_FILE}")
