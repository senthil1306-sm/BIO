# SmartBio - AI Model Integration Directory

This directory provides the infrastructure for integrating deep-learning object detection and semantic segmentation models (such as YOLOv8-seg, YOLOv9, Mask R-CNN, or SegNet) into the SmartBio system.

## Directory Structure

```
model/
├── README.md           # Instructions for training, format, and deployment
└── weights/            # Place your trained .pt, .onnx, or .tflite weights here
    └── (drop weights here, e.g. yolov8x-seg_hull_fouling.pt)
```

## How Dual Analysis Mode Operates

SmartBio features an intelligent dual-mode architecture:

1. **Demo Mode (Computer Vision)**:
   - Always functional without external AI weight downloads.
   - Leverages OpenCV HSV colorimetry, CLAHE contrast enhancement, Laplacian texture filters, morphological clustering, and connected components.
   - Outputs: Suspected biofouling coverage %, highlighted overlay, and binary mask.
   - Clearly labeled: **"Demo Computer-Vision Estimate"**.

2. **AI Model Mode**:
   - The backend service `AIModelAnalyzer` automatically scans `model/weights/` for `.pt`, `.onnx`, or `.engine` files.
   - When trained weights are present and `mode="AI"` is requested, the neural network segments biofouling organisms (barnacles, macro-algae, tubeworms, bryozoans).
   - **Automatic Fallback**: If no weight file is present, or if dependencies are missing, SmartBio gracefully falls back to Demo Mode without crashing, returning an alert notification (`warning_message: "AI model unavailable — switched to Demo Mode."`).

## Integrating a Trained YOLO Segmentation Model

### Step 1: Place Weights in `model/weights/`
Copy your trained PyTorch or ONNX weights into this folder:
```bash
cp path/to/best.pt model/weights/yolov8_hull_fouling.pt
```

### Step 2: Install Ultralytics in Backend Environment
```bash
backend\venv\Scripts\pip.exe install ultralytics
```

### Step 3: Enable Model Loading in `backend/app/analyzers/ai_analyzer.py`
Uncomment and configure the inference call inside `AIModelAnalyzer.analyze()`:
```python
from ultralytics import YOLO

model = YOLO(str(self.weights_path))
results = model(str(image_path))
# Extract segmentation masks and compute hull coverage %
```
