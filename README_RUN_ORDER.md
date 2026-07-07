# ViMed-PET Q1 Kaggle Notebook Pipeline

Recommended run order:

1. `00_archive_listing_manifest.ipynb` or `00_extract_clean_manifest.ipynb`
2. `01_build_25d_cache_streaming.ipynb` or `01_build_25d_cache.ipynb`
3. `02_retrieval_baseline.ipynb`
4. `03_25d_qwen_sft_train.ipynb`
5. `04_25d_qwen_eval.ipynb`
6. `05_3d_dual_stream_train.ipynb`
7. `06_final_eval_ablation.ipynb`

Do not run everything in one Kaggle notebook. Save Version after every stage
and attach the previous stage output dataset to the next notebook.

Accelerator and output guide:

| Step | Notebook | Accelerator | Required inputs | Main output to save/attach next |
|---|---|---|---|---|
| 00 | `00_archive_listing_manifest.ipynb` | None | 3 raw no-autoextract datasets | `vimedpet_q1_outputs/00_manifest` |
| 01 | `01_build_25d_cache_streaming.ipynb` | None | Step 00 output + 3 raw no-autoextract datasets | `vimedpet_q1_outputs/01_cache` |
| 02 | `02_retrieval_baseline.ipynb` | None | Step 01 output | `vimedpet_q1_outputs/02_retrieval` |
| 03 | `03_25d_qwen_sft_train.ipynb` | GPU T4 x2 | Step 01 output + Qwen model input/HF access | `vimedpet_q1_outputs/03_qwen_sft` |
| 04 | `04_25d_qwen_eval.ipynb` | GPU T4 x2 | Step 01 output + Step 03 adapter output | `vimedpet_q1_outputs/04_qwen_eval` |
| 05 | `05_3d_dual_stream_train.ipynb` | GPU T4 x2 | Step 01 output with optional 3D patch cache | `vimedpet_q1_outputs/05_3d_dual` |
| 06 | `06_final_eval_ablation.ipynb` | None | Step 02/04/05 outputs | `vimedpet_q1_outputs/06_final_eval` |

Notebook 00 now builds the manifest from archive listings and report JSON files
without extracting full CT/PET volumes, which avoids Kaggle temp-disk failure.

Notebook 01 now builds the 2.5D cache by extracting small CT/PET batches from
the archives. It can resume from an existing `q1_cache_index.csv` if the Kaggle
session stops before all cases are cached.

Raw archive inputs are needed only for notebooks 00 and 01. Notebook 05 needs
the optional 3D patch cache from notebook 01. Notebooks 02, 03, 04, and 06 use
clean manifest/cache/checkpoint outputs.

Kaggle input paths expected by notebook 00/01:

- `/kaggle/input/datasets/tundng111/vimed-pet-ct-part1-raw-no-autoextract-v3`
- `/kaggle/input/datasets/tundng111/vimed-pet-ct-part2-raw-no-autoextract-v1`
- `/kaggle/input/datasets/tundng111/vimed-pet-ct-part3-raw-no-autoextract-v1`

If Kaggle mounts them under slug paths instead, the notebooks search
`/kaggle/input` recursively for `PETCT_*.zipraw` / `PETCT_*.zip`.
