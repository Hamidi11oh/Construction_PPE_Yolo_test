# Construction-PPE YOLOv8n Baseline — Reproducibility Manual

This README records the Construction-PPE YOLOv8n baseline experiment so that another lab member can reproduce the environment, training, validation, and result handling.

## 1. Experiment Summary

- Task: Object Detection
- Dataset: Construction-PPE
- Model: YOLOv8n
- Framework: Ultralytics
- Training input: 640 × 640
- Baseline training: 100 epochs
- GPU: NVIDIA GeForce RTX 3060 12 GB

### Reported baseline result

The completed baseline result was reported from the saved `best.pt` checkpoint.

| Metric | Result |
|---|---:|
| Precision | 65.3% |
| Recall | 55.8% |
| mAP@0.5 | 59.0% |
| mAP@0.5:0.95 | 27.1% |
| Parameters | 3.01 M |
| GFLOPs | 8.1 |
| Inference | ~3.8 ms/image |

For strict hardware-speed benchmarking, rerun inference/validation on the same machine.

## 2. Environment

Recorded from the actual experiment environment:

```text
OS          Windows 10
Python      3.10.21
Ultralytics 8.4.144
PyTorch     2.5.1+cu121
CUDA        12.1
GPU         NVIDIA GeForce RTX 3060 (12 GB)
CPU         12th Gen Intel Core i9-12900K
CPU cores   24
RAM         31.8 GB
```

Environment:

```text
C:\Users\JOOYOUNGBOK\anaconda3\envs\yolo_env
```

Activate:

```bat
conda activate yolo_env
```

Check:

```bat
yolo checks
```

The check should show CUDA/GPU availability.

## 3. Project Directory

```text
D:\Hamin\Yolo\Construction_PPE_Yolo_test
```

Recommended repository structure:

```text
Construction_PPE_Yolo_test/
├── README.md
├── .gitignore
├── configs/
│   └── construction-ppe.yaml
├── results/
│   ├── results.csv
│   ├── results.png
│   ├── confusion_matrix.png
│   ├── PR_curve.png
│   ├── F1_curve.png
│   └── selected_predictions/
├── weights/
│   └── best.pt
└── runs/                  # normally kept local / gitignored
```

`README.md` should stay in the project root, not inside `runs`.

## 4. Dataset

The dataset was downloaded automatically by Ultralytics during setup.

Observed local path:

```text
C:\Users\JOOYOUNGBOK\datasets\construction-ppe
```

Observed split used in training:

```text
Train: 1132 images
Val:    143 images
```

Classes:

```text
0  helmet
1  gloves
2  vest
3  boots
4  goggles
5  none
6  Person
7  no_helmet
8  no_goggle
9  no_gloves
10 no_boots
```

### Important reproducibility note

The original training command used:

```text
data=construction-ppe.yaml
```

For better cross-machine reproducibility, keep a project-local copy in:

```text
configs/construction-ppe.yaml
```

and point the training command to that copy.

## 5. Baseline Training

Run from:

```text
D:\Hamin\Yolo\Construction_PPE_Yolo_test
```

The recorded successful baseline command was:

```bat
yolo detect train model=yolov8n.pt data=construction-ppe.yaml epochs=100 imgsz=640 batch=16 device=0 name=baseline_yolov8n
```

Main settings:

```text
Model      yolov8n.pt
Dataset    construction-ppe.yaml
Epochs     100
Image size 640
Batch      16
Device     GPU 0
Run name   baseline_yolov8n
```

Output:

```text
D:\Hamin\Yolo\Construction_PPE_Yolo_test\runs\detect\baseline_yolov8n
```

Important result files:

```text
results.csv
results.png
confusion_matrix.png
PR_curve.png
F1_curve.png
weights\best.pt
weights\last.pt
```

Use `best.pt` when referring to the best saved model checkpoint.

## 6. Validation / Re-validation

Training performs validation automatically.

For an independent re-validation of the saved model:

```bat
yolo detect val model="D:\Hamin\Yolo\Construction_PPE_Yolo_test\runs\detect\baseline_yolov8n\weights\best.pt" data=construction-ppe.yaml imgsz=640 device=0 workers=0
```

Expected validation scale:

```text
143 validation images
11 classes
```

## 7. Prediction / Inference

For an independent image/folder test:

```bat
yolo detect predict model="D:\Hamin\Yolo\Construction_PPE_Yolo_test\runs\detect\baseline_yolov8n\weights\best.pt" source="PATH_TO_IMAGE_OR_FOLDER" imgsz=640 device=0
```

Example:

```bat
yolo detect predict model="D:\Hamin\Yolo\Construction_PPE_Yolo_test\runs\detect\baseline_yolov8n\weights\best.pt" source="D:\Hamin\Yolo\Test_Images" imgsz=640 device=0
```

The prediction output is written to an Ultralytics `runs\detect\predict*` directory.

Note: prediction is a reproducibility/inference check; it was not the command used to calculate the reported training metrics.

## 8. Class-level Baseline Results

Reported mAP@0.5 by class:

| Class | mAP@0.5 |
|---|---:|
| helmet | 81.1% |
| gloves | 78.1% |
| vest | 81.0% |
| boots | 79.6% |
| goggles | 80.9% |
| none | 53.5% |
| Person | 91.9% |
| no_helmet | 21.5% |
| no_goggle | 9.5% |
| no_gloves | 21.9% |
| no_boots | 49.5% |

Main observation:

- Several standard PPE classes were relatively strong.
- Several violation classes were substantially weaker.
- `no_boots` has very few validation instances, so its class metric should be interpreted cautiously.
- This is baseline evidence for later failure-case analysis, not a final research contribution.

## 9. Training Issue Encountered

An earlier training attempt stopped around epoch 29 with:

```text
RuntimeError: Couldn't open shared file mapping
```

and:

```text
RuntimeError: DataLoader worker ... exited unexpectedly
```

That run used multiple DataLoader workers.

For later Windows experiments, using:

```text
workers=0
```

was the stable workaround.

Example:

```bat
yolo detect train ... workers=0
```

## 10. Invalid Run to Ignore

A later run named:

```text
runs\detect\train-2
```

must NOT be used as a Construction-PPE result.

Its validation log showed:

```text
4 images
17 instances
person
dog
horse
elephant
umbrella
potted plant
```

This was a COCO8 run, not Construction-PPE.

Therefore:

```text
train-2 → invalid for Construction-PPE reporting
```

Keep it only as a local troubleshooting record if needed.

## 11. Recommended Experiment Archive

For every experiment, preserve:

```text
results.csv
results.png
confusion_matrix.png
PR_curve.png
F1_curve.png
best.pt
training command
environment versions
dataset path/version
```

Also record:

```text
Experiment ID:
Date:
Model:
Dataset:
Epochs:
Image size:
Batch:
GPU:
Workers:
Best epoch:
Precision:
Recall:
mAP@0.5:
mAP@0.5:0.95:
Main observation:
```

This makes baseline-to-improvement comparisons easier.

## 12. GitHub Strategy

Yes, the experiment should be version-controlled.

For this project, the Git repository can be the project directory itself:

```text
D:\Hamin\Yolo\Construction_PPE_Yolo_test
```

### Commit to GitHub

Recommended:

```text
README.md
configs/
results/results.csv
results/results.png
results/confusion_matrix.png
results/PR_curve.png
results/F1_curve.png
selected prediction images
scripts/
```

### Normally keep out of Git

```text
datasets/
*.cache
runs/
temporary files
```

Do not upload the dataset itself.

For `best.pt`, keep one clearly identified checkpoint. If the file is large, use Git LFS or your lab's approved storage rather than committing many checkpoints.

## 13. Minimal Git Commands

From the project directory:

```bat
cd /d D:\Hamin\Yolo\Construction_PPE_Yolo_test
git init
git add README.md .gitignore configs results
git commit -m "Add Construction-PPE YOLOv8n baseline"
```

Then connect the local repository to the lab GitHub repository and push.

## 14. Current Status

```text
[✓] YOLO environment verified
[✓] Construction-PPE dataset prepared
[✓] YOLOv8n baseline trained
[✓] Validation completed
[✓] Class-level results recorded
[✓] Initial limitation observed
[ ] Organize reproducibility files
[ ] Commit experiment to GitHub
```

## 15. Baseline Command — Quick Reference

```bat
yolo detect train model=yolov8n.pt data=construction-ppe.yaml epochs=100 imgsz=640 batch=16 device=0 name=baseline_yolov8n
```

Baseline output:

```text
runs\detect\baseline_yolov8n
```
