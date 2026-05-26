# 🏷️ YOLO Bounding Box Annotator

<div align="center"> 

**A lightweight, keyboard-free image annotation tool built with pure OpenCV — draw boxes with your mouse, save YOLO `.txt` labels instantly.**

[![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)]()
[![OpenCV](https://img.shields.io/badge/OpenCV-5C3EE8?style=for-the-badge&logo=opencv&logoColor=white)]()
[![YOLO](https://img.shields.io/badge/YOLO-Format-FF6600?style=for-the-badge)]()

</div>

---

## 📖 What Is This?

A minimal manual annotation tool that lets you draw bounding boxes on images using only your mouse and output **YOLO-format `.txt` label files** — one annotation file per image, ready to drop into any YOLO training pipeline.

No external GUI libraries, no LabelImg, no Roboflow — just OpenCV, a mouse, and a coloured sidebar.

---

## 🖥️ Interface

The annotation window is split into two sections:

```
┌────────┬─────────────────────────────────────────┐
│  100px │              Image (800×800)             │
│        │                                          │
│ START  │  ← Draw bounding boxes here             │
│  red   │                                          │
│        │  Green rectangle appears while you drag │
│  NEXT  │  Released → box saved to .txt instantly  │
│ orange │                                          │
│        │                                          │
│RETAKE  │                                          │
│ yellow │                                          │
│        │                                          │
│  SKIP  │                                          │
│  blue  │                                          │
│        │                                          │
│ [rot]  │                                          │
│  pink  │                                          │
│        │                                          │
│ [spare]│                                          │
│  cyan  │                                          │
└────────┴─────────────────────────────────────────┘
```

> The sidebar (`option.png`) is a pre-made 100×800 image with coloured zones — clicking a zone triggers its action.

---

## 🕹️ Controls

All controls are **mouse-only** — click inside the left sidebar:

| Zone | Y range | Colour | Action |
|------|---------|--------|--------|
| **START** | 0–133 | 🔴 Red | Enable box-drawing mode |
| **NEXT** | 133–266 | 🟠 Orange | Save current labels and move to next image |
| **RETAKE** | 266–400 | 🟡 Yellow | Clear all annotations for this image and start over |
| **SKIP** | 400–533 | 🔵 Blue | Skip image — replaces it with a blank white frame |
| **ROTATE** | 533–666 | 🩷 Pink | Rotate image 90° clockwise and re-annotate |

### Drawing a Box

1. Click **START** first to enable drawing mode
2. **Click and drag** on the image — a live green rectangle appears as you drag
3. **Release** the mouse button — the box is immediately saved to the `.txt` file

You can draw **as many boxes as you want** on one image before pressing **NEXT**.

---

## 📐 YOLO Label Format

Each saved annotation line follows the standard YOLO format:

```
<class_id> <x_center> <y_center> <width> <height>
```

All values are **normalised** to `[0, 1]` relative to the image dimensions.

**Example output (`image001.txt`):**
```
0 0.26125 0.026875 0.095 0.05125
0 0.2825 0.139375 0.1125 0.07375
0 0.270625 0.2125 0.08125 0.055
```

### How the Normalisation Is Calculated

```python
# Mouse gives pixel coordinates (x_temp1, y_temp1) at press
# and (x_temp2, y_temp2) at release
# The image is offset 100px right (sidebar), so subtract 200 from x sum

x_center = (x_temp1 + x_temp2 - 200) / (2 * image_width)
y_center = (y_temp1 + y_temp2) / (2 * image_height)
width    = abs(x_temp1 - x_temp2) / image_width
height   = abs(y_temp1 - y_temp2) / image_height
```

> **Why `-200` on x?** The sidebar is 100px wide. The raw mouse x coordinates are in the full window space (sidebar + image). Subtracting 200 from the sum of both x values (i.e. subtracting 100 from each) corrects for the offset.

---

## ⚙️ How It Works

### Pipeline Per Image

```
Load image from input folder
        │
        ▼
Resize to 800×800
        │
        ▼
Attach sidebar (hstack option.png + image)
        │
        ▼
Write to delete.png (working copy)
        │
        ▼
Enter annotation loop:
  │
  ├── Mouse in sidebar (x < 100)?
  │       └── Dispatch action (START / NEXT / RETAKE / SKIP / ROTATE)
  │
  ├── START active + mouse drag?
  │       └── Draw live green rectangle on image
  │
  ├── Mouse released?
  │       └── Compute YOLO coords → append to label .txt file
  │
  └── Show updated image in window
        │
        ▼ (NEXT pressed)
Move to next file in folder
```

### File I/O

| File | Purpose |
|------|---------|
| `delete.png` | Working copy of current annotated image (overwritten each frame) |
| `option.png` | Sidebar button image (100×800 px, pre-made) |
| `<name>.txt` | YOLO label file written to output folder |
| `_.txt` | Temp file cleared on RETAKE |

**RETAKE** erases the `.txt` file and re-draws the sidebar so you can start from a clean slate:
```python
with open(name, "w") as file:
    file.close()   # truncate to empty
```

**SKIP** overwrites the original image with a blank white frame so it won't appear in future training runs:
```python
blank = np.ones((800, 800, 3)).astype(np.uint8)
cv2.imwrite(file_path, blank)
```

---

## 🚀 Setup & Installation

### Requirements

- Python 3.x
- Images to annotate

### Install Dependencies

```bash
pip install opencv-python numpy pandas
```

### Project Structure

```
yolo-annotator/
├── yolo_annotate.ipynb   # Main annotation notebook
├── option.png            # Sidebar button image (required)
├── delete.png            # Auto-generated working copy (do not edit)
└── README.md
```

### Configure Paths

In the second cell of the notebook, set your folders and class ID:

```python
infu   = r"D:\your\images\folder"    # Input: folder of images to annotate
onfu   = r"D:\your\labels\folder"    # Output: where .txt files are saved
annoti = "0"                          # Class ID written to every label line
```

- `infu` — any folder of `.jpg` / `.png` images
- `onfu` — must exist before running (create it manually)
- `annoti` — change to `"1"`, `"2"`, etc. for different classes. Re-run the notebook per class if you have multiple.

### Run

```bash
jupyter notebook yolo_annotate.ipynb
```

Run all cells. The annotation window opens with the first image. Work through each image, then close the window when done.

---

## 📁 Output Structure (YOLO-Ready)

After annotating, your dataset will look like:

```
dataset/
├── images/
│   ├── img001.jpg
│   ├── img002.jpg
│   └── ...
└── labels/
    ├── img001.txt     ← one line per box, YOLO format
    ├── img002.txt
    └── ...
```

This matches the standard YOLOv5 / YOLOv8 directory layout and can be used directly in a `data.yaml` training config.

---

## 🔮 Future Improvements

- [ ] **Multi-class support** — sidebar colour zones mapped to different class IDs (currently single-class per run)
- [ ] **Undo last box** — press a button to remove the most recently drawn annotation
- [ ] **Box display on reload** — show existing annotations when resuming a half-annotated image
- [ ] **Zoom** — scroll wheel to zoom in on small details
- [ ] **Auto-suggest boxes** — run a lightweight detector to pre-fill boxes, then manually confirm/reject

---

*Built with ❤️ using pure OpenCV — no annotation GUI required.*
