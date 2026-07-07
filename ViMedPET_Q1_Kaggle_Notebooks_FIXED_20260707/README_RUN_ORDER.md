# ViMedPET Q1 Kaggle Notebooks Fixed 2026-07-07

Run order and inputs:

1. 00_extract_clean_manifest.ipynb
   - Accelerator: None
   - Inputs: three raw no-autoextract Kaggle datasets for Part 1, Part 2, Part 3
   - Main checks: 2,757 cases, zero split overlap, Findings coverage, Impression coverage >= 90%

2. 01_build_25d_cache.ipynb
   - Accelerator: None
   - Inputs: output of notebook 00 plus three raw no-autoextract Kaggle datasets
   - Main checks: cache index, 2.5D memmap, model target contains IMPRESSION >= 90%

3. 02_retrieval_baseline.ipynb
   - Accelerator: None
   - Inputs: output of notebook 01

4. 03_25d_qwen_sft_train.ipynb
   - Accelerator: GPU T4 x2
   - Inputs: output of notebook 01 plus Qwen/Hugging Face model access
   - Do not use old cache outputs created before 2026-07-07.

5. 04_25d_qwen_eval.ipynb
   - Accelerator: GPU T4 x2
   - Inputs: output of notebook 01 plus output of notebook 03

6. 05_3d_dual_stream_train.ipynb
   - Accelerator: GPU T4 x2
   - Inputs: run only after 2.5D pipeline is stable and a 3D patch cache exists

7. 06_final_eval_ablation.ipynb
   - Accelerator: None
   - Inputs: outputs of notebooks 02, 03/04, and 05 when available

Parser note:
Notebook 00 uses normalized Vietnamese JSON keys, so the parser reads keys such as `M? t? h?nh ?nh` and `Nh?n ??nh k?t qu?` without depending on source-file Unicode rendering.
