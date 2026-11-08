# AI Application — Group 9


## Install
```bash
pip install -r requirements.txt
```

---

## Project structure
```text
primagaz-yolo/
├── notebooks/
│   ├── 00_data_preparation.ipynb
│   ├── 01_gas_bottle_baseline.ipynb
│   ├── 02_video_tracking.ipynb
│   ├── 03_gas_cylinder_training.ipynb
│   ├── 04_gas_cylinder_evaluation.ipynb
│   ├── 05_cvat_custom_dataset.ipynb
│   ├── 06_synchronised_tracking.ipynb
│   ├── 07_ocr_first_test.ipynb
│   ├── 08_ocr_data_preparation.ipynb
│   └── 09_ocr_primagaz.ipynb
├── datasets/
├── videos/
├── videos_1/
├── videos_2/
├── data/
│   ├── ocr_frames/
│   └── ocr_ws6/
├── results/
└── requirements.txt
```

---

## Notebooks

| Notebook | Workshop | Purpose |
|---|---|---|
| 00_data_preparation | WS2 + WS4 | Extracts frames from the raw videos for CVAT annotation and testing. |
| 01_gas_bottle_baseline | WS2 | Runs pretrained YOLO11n (COCO) on local LGP images. Shows the model has no cylinder class. |
| 02_video_tracking | WS2 | Runs ByteTrack and BoT-SORT on short videos. |
| 03_gas_cylinder_training | WS2 | Fine-tunes YOLO11n on the public Roboflow datasets. |
| 04_gas_cylinder_evaluation | WS2 | Re-evaluates each public dataset `best.pt` on validation and test splits. |
| 05_cvat_custom_dataset | WS4 | Merges the 3 CVAT batches, performs a scene-aware train/val split (to prevent data leakage), and trains the final YOLO11n model. |
| 06_synchronised_tracking | WS4 | Runs the final CVAT model on the optimized videos to assign global, monotonically increasing IDs to bottles across different camera angles. |
| 07_ocr_classification | WS5 | First OCR experiment. EasyOCR on 12 hand-cropped close-ups. |
| 08_ocr_data_preparation | WS6 | Extracts one shoulder crop per camera per bottle from the 4K source videos at each bottle's `.xlsx` timestamp. Writes `manifest.csv`. |
| 09_ocr_primagaz | WS6 | Runs EasyOCR on every crop, parses tare and year with regex, and votes per bottle with camera weights. |

*(Note: The raw video and dataset files must be downloaded or extracted into their respective folders before running the notebooks).*

---

## Datasets & Resources

**Roboflow Datasets (Public):**

| Dataset | Location | Classes | Link |
|---|---|---|---|
| CylinDeRS | `datasets/gas_bottles/` | `gas_cylinder` | https://universe.roboflow.com/klearchos-stavrothanasopoulos-konstantinos-gkountakos-6jwgj/cylinders-iaq6n/dataset/3 |
| Gas-cylinder detection | `datasets/gas_cylinder/` | `gas_cylinder`, `gas cylinder head` | https://universe.roboflow.com/obj-dect/gas-cylinder-detection/dataset/3 |

**Offline CVAT Datasets (Custom Labeled):**

- `datasets/cvat_batch_1a/`: NOK bottles provided by the lecturer.
- `datasets/cvat_batch_1b/`: Mixed OK/NOK bottles from the factory site.
- `datasets/cvat_batch_2/`: Extracted from YouTube reference videos.

**Reference Videos (videos_1):**

- Bolivia: FLEXSPEED for Side-Entry Valves (2014): https://www.youtube.com/watch?v=PtKnRpGvPM4
- Portugal: Petrogal LPG Cylinder Filling Plant (2013): https://www.youtube.com/watch?v=UH-u0g3B8Pw
- Automatic LPG Cylinder filling plant: https://www.youtube.com/shorts/sdo9Pg50GNY
- LPG CYLINDER FILLING CAROUSEL (ELECTRONIC 36 sec.): https://www.youtube.com/shorts/dvBFjJub6rE
- LPG CYLINDER FILLING CAROUSEL: https://www.youtube.com/shorts/dvBFjJub6rE
- LPG Cylinder Robotic Loading System: https://www.youtube.com/watch?v=TIVIFfRqbgU

**Primagaz Videos:**

- Original 4K footage in `videos/` (front, back left, back right views for both 14:43 and 14:55).
- Downscaled versions in `videos_2/` used for the WS4 synchronised tracking demos.

**Ground Truth:**

- `data/14_43_ground_truth_ai_app.xlsx` — sequence, time, tare, year, colour, status per bottle.
- `data/14_55_ground_truth_ai_app.xlsx` — same structure.

**Synchronised Tracking Demos (WS4):**

- Front Camera:
  - https://github.com/mahrukhtayyab23/AI-Application-Group-9/blob/main/results/synchronised_tracking/demos/sync_optimized_14_55_front_cropped.mp4
  - https://github.com/mahrukhtayyab23/AI-Application-Group-9/blob/main/results/synchronised_tracking/demos/sync_optimized_14_43_front_cropped.mp4
- Back Left Camera:
  - https://github.com/mahrukhtayyab23/AI-Application-Group-9/blob/main/results/synchronised_tracking/demos/sync_optimized_14_55_back_left_cropped.mp4
  - https://github.com/mahrukhtayyab23/AI-Application-Group-9/blob/main/results/synchronised_tracking/demos/sync_optimized_14_43_back_left_cropped.mp4
- Back Right Camera:
  - https://github.com/mahrukhtayyab23/AI-Application-Group-9/blob/main/results/synchronised_tracking/demos/sync_optimized_14_55_back_right_cropped.mp4
  - https://github.com/mahrukhtayyab23/AI-Application-Group-9/blob/main/results/synchronised_tracking/demos/sync_optimized_14_43_back_right_cropped.mp4

---

## Model & Training Settings

**Model:** YOLO11n (initialized from COCO weights).

**Settings for the final CVAT model (Notebook 05):**

| Field | Value |
|---|---|
| Epochs | 25 |
| Image size | 416 |
| Batch size | 2 (limited by CPU RAM) |
| Device | CPU |

*Note: Public dataset training (Notebook 03) used 5 epochs and a fraction of 0.25 due to the large size of the CylinDeRS dataset.*

---

## Results

**Baseline:** The pretrained COCO model has no cylinder class. It labels cylinder-shaped objects as vase, bottle, or fire hydrant, or misses them entirely.

**Tracking:** ByteTrack is faster than BoT-SORT.

**Public Datasets (Notebook 04):**

| Dataset | Split | Precision | Recall | mAP50 | mAP50-95 |
|---|---|---|---|---|---|
| gas_bottles | val | 0.9170 | 0.6864 | 0.8866 | 0.8421 |
| gas_bottles | test | 0.9321 | 0.6845 | 0.8952 | 0.8545 |
| gas_cylinder | val | 0.7896 | 0.3108 | 0.3449 | 0.1263 |
| gas_cylinder | test | 0.8344 | 0.3421 | 0.3775 | 0.1567 |

**Custom CVAT Dataset (Final Model for WS4):**

| Dataset | Split | Precision | Recall | mAP50 | mAP50-95 |
|---|---|---|---|---|---|
| cvat_merged | val | 0.8312 | 0.6765 | 0.7345 | 0.5320 |

Training on our site-specific CVAT data improved the mAP50 from ~34% (public dataset) to 73.4%, proving the necessity of domain-specific annotations.

---

## WS6 — OCR on Primagaz Bottles

**Goal:** read the tare weight and re-certification year from the printed shoulder of the bottle.

**Pipeline:**

1. `08_ocr_data_preparation.ipynb` — for each bottle in the `.xlsx`, seek each of the 3 synchronised videos to that bottle's timestamp, run the WS4 YOLO11n detector, crop the shoulder (20%–75% of the bbox), save to `data/ocr_frames/`. Output `manifest.csv`.
2. `09_ocr_primagaz.ipynb` — run EasyOCR on every crop, parse tare and year with regex, reject the net-weight sticker (12.5 / 10.5), vote per bottle with weights `back_right=3, front=2, back_left=1`. Output `data/ocr_ws6/`.

**Coverage:**

| Item | Value |
|---|---|
| Ground-truth bottles with readable tare+year | 182 (154 black, 28 teal) |
| Bottles with at least one shoulder crop | 182 (100%) |
| Bottles with all 3 cameras | 163 (89.6%) |
| Total crops extracted | 526 |

**OCR performance:**

| Metric | Value |
|---|---|
| Bottles with a crop of the correct bottle | 4 of 182 (2.2%) |
| Bottles where the correct crop gave the exact tare | 3 of 4 (75%) |
| **Tare exact match, end-to-end** | **3 of 182 (1.6%)** |
| **Year exact match, end-to-end** | **0 of 182 (0%)** |

**Diagnosis:**

- **Layer 1 — alignment.** The `.xlsx` gives wall-clock times of the filling process, not frame numbers in the video. The pipeline assumed `video_time = wall_clock − session_start`, which breaks when the 4K source videos are downscaled or trimmed. Result: 178 of 182 bottles have their crop landing on a neighbouring bottle.
- **Layer 2 — digit misreads.** On the 4 correctly-cropped bottles, EasyOCR gave the exact tare on 3 of 4. The one miss was `10.8` instead of `10.6` — a `6`/`8` confusion at 4K distance on shiny metal.
- **Layer 3 — year.** The tare and year are on the same printed line. OCR reads the year glued to the `KG` text as `KG-2037`, one digit off from GT `2036`. That is a `6`/`7` digit confusion, not a parsing failure.

**Recommended fixes (WS7):**

1. **Ask Primagaz for a per-camera video-start label** — data change only, no code change needed.
2. **Fixed close-range shoulder camera** — 40–60 cm from the bottle reproduces the WS5 close-up conditions.
3. **Contrast booster for teal bottles** — teal has readable print but the current CLAHE + upscale is tuned for white-on-black. A second pre-processing path (inverted contrast or adaptive thresholding) would recover teal bottles.

**Key insight:** the pipeline and the OCR engine both work. The single blocker is the timestamp-to-frame mapping. The OCR engine is 75% correct on the crops it is correctly given.

---

## Known limitations

- The pre-trained COCO model has no cylinder class.
- Models trained on public datasets (Roboflow) fail to generalize well to our specific factory videos.
- The CVAT dataset is currently imbalanced (more NOK than OK bottles), which will affect the future OK/NOK classification model.
- All training was done on CPU.