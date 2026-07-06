Đã tạo bộ notebook Q1 mới tại:

[ViMedPET_Q1_Kaggle_Notebooks](C:\Users\ADMIN\Downloads\DSP391m\ViMedPET_Q1_Kaggle_Notebooks)

Zip:

[ViMedPET_Q1_Kaggle_Notebooks.zip](C:\Users\ADMIN\Downloads\DSP391m\ViMedPET_Q1_Kaggle_Notebooks.zip)

Tôi đã compile toàn bộ code cell, không lỗi cú pháp. Tôi không chạy training local vì phần đó phải chạy trên Kaggle GPU và raw archive input.

**Chiến lược chốt**
Không chạy 1 notebook tổng. Chạy 7 notebook tách stage là hợp lý nhất vì raw PET/CT rất nặng, Kaggle `/working` giới hạn, session 12h/30h có thể ngắt, và cache/checkpoint cần được Save Version từng bước.

**Run Order Và Input**

| Notebook | Input cần attach | Output chính |
|---|---|---|
| `00_extract_clean_manifest.ipynb` | 3 raw datasets Part 1/2/3 | `q1_clean_manifest.csv`, `q1_bad_records.csv`, split QA |
| `01_build_25d_cache.ipynb` | 3 raw datasets + output notebook 00 | 2.5D cache memmap, preview ảnh, cache index |
| `02_retrieval_baseline.ipynb` | output notebook 01 | retrieval metrics, retrieved reports |
| `03_25d_qwen_sft_train.ipynb` | output notebook 01 + Qwen2.5-VL model | LoRA adapter checkpoint |
| `04_25d_qwen_eval.ipynb` | output notebook 01 + output notebook 03 | generated reports + BLEU/ROUGE/clinical/SUV metrics |
| `05_3d_dual_stream_train.ipynb` | output notebook 01 với `BUILD_3D_PATCH_CACHE=True` | 3D dual-stream checkpoint + val loss |
| `06_final_eval_ablation.ipynb` | outputs notebook 02/04/05 | final ablation summary table |

Raw inputs bạn đưa đã được đưa vào notebook 00/01:

```text
/kaggle/input/datasets/tundng111/vimed-pet-ct-part1-raw-no-autoextract-v3
/kaggle/input/datasets/tundng111/vimed-pet-ct-part2-raw-no-autoextract-v1
/kaggle/input/datasets/tundng111/vimed-pet-ct-part3-raw-no-autoextract-v1
```

Notebook cũng tự search `/kaggle/input` nếu Kaggle mount bằng slug khác.

**Cách Chạy Trên Kaggle**
1. Chạy `00`, Save Version.
2. Notebook `01` attach output của `00` + raw 3 part, chạy cache, Save Version.
3. Từ `02` trở đi không attach raw nữa, chỉ attach output cache/checkpoint.
4. `03` bản mới mặc định chạy `RUN_MODE="full"` để train QLoRA đầy đủ trên T4 x2; nếu chỉ muốn test nhanh, sửa `RUN_MODE="smoke"` trong notebook.
5. Nếu Kaggle không có internet, add Qwen model làm Kaggle input hoặc set `QWEN_MODEL_PATH`.

**Gate Tối Thiểu**
- `00`: patient overlap = `0`, val/test đủ CT/PET/report.
- `01`: val/test cache `100%`.
- `02`: Recall@5 `>= 0.20`, MRR `>= 0.08`, retrieved ROUGE-L `>= 0.18`.
- `03`: full run phải có `adapter_final`, `training_summary.json`, `trainer_log_history.csv`, và `validation_generation_samples.csv`; loss phải giảm ổn định và không OOM.
- `04`: Q1 tối thiểu nên hướng tới ROUGE-L `>= 0.30`, BLEU-4 `>= 0.08`, clinical keyword F1 `>= 0.30`, SUV consistency `>= 0.60`.
- `05`: val loss giảm ổn định, không OOM; 3D nên cải thiện clinical/SUV so với 2.5D.
- `06`: có bảng ablation cuối cùng.

Repo ViPET-ReportGen có ích, nhất là cấu trúc stage: finetune vision encoder, VLM train/inference, clinical evaluation. Tôi đã đưa hướng đó vào pipeline nhưng chỉnh lại cho thực tế T4: cache 2.5D trước, retrieval baseline trước, Qwen LoRA sau, 3D dual-stream là stage riêng. Không nên claim “bằng hoặc hơn paper” nếu chưa cùng split/metric/compute; paper dùng pipeline lớn hơn và clinical metric riêng.

Nguồn đối chiếu: [ViPET-ReportGen GitHub](https://github.com/AIoT-Lab-AI4LIFE/ViPET-ReportGen).

