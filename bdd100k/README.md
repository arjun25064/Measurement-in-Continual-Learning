# BDD100K: Determining the Best Teacher Model Through Annotations

> Evaluating and selecting optimal teacher-student model pairs for knowledge distillation in edge video analytics.

**Team:** Arjun Prasad (MT25064), Videh Sharma, Anand  
**Mentor:** Shubham Chaudhary  
**Professor:** Dr. Arani Bhattacharya  
**Institute:** Indraprastha Institute of Information Technology, Delhi

---

## Overview

This experiment evaluates a set of pretrained and fine-tuned object detection models on the **BDD100K** driving dataset to identify the best teacher model for knowledge distillation pipelines (such as RECL and Ekya). The goal is to find models that maximize detection accuracy and parameter efficiency when serving as teachers for lightweight student models deployed on edge devices.

---

## Dataset: BDD100K

BDD100K is a large-scale, diverse driving video dataset collected across different US locations.

| Property | Details |
|---|---|
| Videos | 100,000 (~40 sec each, 720p, 30 FPS) |
| Total duration | 1,100+ hours |
| Conditions | Day/night, sunny, rainy, overcast |
| Annotation types | Object detection, lane markings, drivable areas |
| Object classes | person, rider, car, truck, bus, train, motor, bike, traffic light, traffic sign |

> **Note:** The dataset is heavily car-dominated (~714K car instances vs. 136 train instances), which significantly affects per-class AP scores.

---

## Models Evaluated

### Teacher Models (Large)
- `yolo26x`
- `yolov8x`
- `yolo11x`
- `yolov10x`
- `rtdetr-x`

### Student Models (Lightweight)
- `yolo26n`
- `yolov8n`
- `yolo11n`
- `yolov10n`
- `rtdetr-l`

---

## Key Results

### Pretrained (COCO weights) — Poor Domain Transfer

All pretrained models performed poorly out-of-the-box on BDD100K due to the domain gap between COCO and driving-specific data.

| Model Type | mAP50 Range |
|---|---|
| Teacher (pretrained) | 0.123 – 0.140 |
| Student (pretrained) | 0.113 – 0.121 |

Per-class analysis showed that only `person` and `car` were reliably detected; the majority of classes (truck, bus, train, motor, bike, traffic light, traffic sign) had near-zero AP.

---

### Fine-tuned on BDD100K — Major Improvement

Fine-tuning on BDD100K dramatically improved performance across all models.

| Model Type | mAP50 Range |
|---|---|
| Teacher (fine-tuned) | 0.551 – 0.609 |
| Student (fine-tuned) | 0.355 – 0.493 |

---

### Teacher Model Rankings (Fine-tuned)

| Rank | Model | mAP50 | Efficiency (mAP50/M params) |
|---|---|---|---|
| Best accuracy | `yolov8x` | 0.609 | 8.87 |
| Best efficiency | `yolov10x` | 0.594 | **18.87** |
| 3rd | `rtdetr-x` | 0.593 | 8.60 |
| 4th | `yolo11x` | 0.577 | — |
| 5th | `yolo26x` | 0.551 | 8.83 |

**`yolov10x` is the recommended teacher** — it achieves near top-1 accuracy while being more than 2× more parameter-efficient than all other teachers.

---

### Student Model Rankings (Fine-tuned)

| Rank | Model | mAP50 | Efficiency (mAP50/M params) |
|---|---|---|---|
| Best efficiency | `yolov10n` | 0.476 | **175.78** |
| 2nd | `yolo26n` | 0.486 | 172.59 |
| 3rd | `yolov8n` | 0.481 | 135.72 |
| 4th | `rtdetr-l` | 0.493 | 17.16 |
| Lowest accuracy | `yolo11n` | 0.355 | — |

**`yolov10n` is the recommended student** — best efficiency score and strong per-class coverage.

---

### Recommended Pair

| Role | Model | mAP50 |
|---|---|---|
| Teacher | `yolov10x` | 0.594 |
| Student | `yolov10n` | 0.476 |

The `yolov10x → yolov10n` pairing offers the best balance of accuracy, efficiency, and architectural consistency for knowledge distillation.

---

## Per-Class AP Observations

- **`train`** class: Zero AP across all models (teacher and student, pretrained and fine-tuned) — virtually absent in BDD100K driving scenarios.
- **`car` and `person`**: Consistently the best-detected classes, with fine-tuned AP50 values reaching 0.81+.
- **`rider`, `motor`, `bike`**: Improved significantly after fine-tuning but remain challenging due to low instance counts.
- **`traffic light` and `traffic sign`**: Moderate performance after fine-tuning (~0.5–0.7 AP50); challenging due to small object size.

---

## Repository Structure

```
.
├── data/                          # BDD100K dataset scripts / splits
├── models/                        # Model configs and weight loading
├── train/                         # Fine-tuning scripts
├── eval/                          # Evaluation and metric scripts
├── results/                       # mAP plots, per-class AP grids
├── BDD100K_Determining_the_best_teacher_model.pdf   # Presentation slides
└── README.md
```

---

## Context

This work is part of a broader research effort on **continual learning for edge video analytics**, building on systems like [Ekya](https://www.usenix.org/conference/nsdi22/presentation/bhardwaj) and RECL. Selecting an appropriate teacher model is a prerequisite for effective knowledge distillation in resource-constrained edge deployments.

Related presentations in this series:
- `Ekya_Arjun_Prasad.pdf` — Review of the Ekya continuous learning system
- `RECL_Presentation.pdf` — Responsive Resource-Efficient Continuous Learning

---

## Citation

If you use BDD100K in your work:

```
@InProceedings{yu2020bdd100k,
  title={BDD100K: A Diverse Driving Dataset for Heterogeneous Multitask Learning},
  author={Yu, Fisher and Chen, Haofeng and Wang, Xin and Xian, Wenqi and Chen,
          Yingying and Liu, Fangchen and Madhavan, Vashisht and Darrell, Trevor},
  booktitle={CVPR},
  year={2020}
}
```
