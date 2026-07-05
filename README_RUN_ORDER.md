# ViMed-PET Q1 Kaggle Notebook Pipeline

Recommended run order:

1. `00_extract_clean_manifest.ipynb`
2. `01_build_25d_cache.ipynb`
3. `02_retrieval_baseline.ipynb`
4. `03_25d_qwen_sft_train.ipynb`
5. `04_25d_qwen_eval.ipynb`
6. `05_3d_dual_stream_train.ipynb`
7. `06_final_eval_ablation.ipynb`

Do not run everything in one Kaggle notebook. Save Version after every stage
and attach the previous stage output dataset to the next notebook.

Raw archive inputs are needed only for notebooks 00 and 01. Notebook 05 needs
the optional 3D patch cache from notebook 01. Notebooks 02, 03, 04, and 06 use
clean manifest/cache/checkpoint outputs.

Kaggle input paths expected by notebook 00/01:

- `/kaggle/input/datasets/tundng111/vimed-pet-ct-part1-raw-no-autoextract-v3`
- `/kaggle/input/datasets/tundng111/vimed-pet-ct-part2-raw-no-autoextract-v1`
- `/kaggle/input/datasets/hannhu4002/vimed-pet-part-3-archive`

If Kaggle mounts them under slug paths instead, the notebooks search
`/kaggle/input` recursively for `PETCT_*.zipraw` / `PETCT_*.zip`.
