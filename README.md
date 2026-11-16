# Fairness Analysis of Retinal Lesion Segmentation on AMD-SD

A pilot study on subgroup performance disparities of a DINOv2-based segmentation model for wet AMD biomarkers (PED and SRF).

## Research question

Do segmentation models for retinal lesions perform equally well across patient subgroups (sex, age)? And does a good global Dice score hide disparities between subgroups?

This work is motivated by recent findings showing that bias mitigation in LLMs can be *superficial* (Pearman et al., 2026), leaving bias anchored in internal representations. We ask whether a similar phenomenon may occur in medical image segmentation: an apparently good global performance may mask subgroup-level disparities.

## Data

- **Dataset:** AMD-SD (Wet AMD OCT Segmentation Dataset), 3049 B-scans from 138 patients.
- **Biomarkers:** PED (Pigment Epithelial Detachment) and SRF (Subretinal Fluid).
- **Subset used in this pilot:** 119 test images, 7 eyes, 7 patients (split by patient to avoid data leakage).
- **Demographics:** age and sex provided per eye.

## Method

- **Model:** DINOv2-base backbone (`facebook/dinov2-base`) with a lightweight convolutional decoder for binary segmentation.
- **Preprocessing:** images resized to 224×224, normalized to [0, 1]; masks binarized at 0.5.
- **Training:** BCEWithLogitsLoss with positive class weighting, Adam optimizer, learning rate 1e-5, data augmentation.
- **Evaluation:** Dice per B-scan, then **aggregated per eye** (mean across B-scans of the same eye) to avoid treating correlated B-scans as independent samples.
- **Fairness analysis:** Dice compared across sex and age groups; disparity measured as the Dice gap between subgroups.

## Results

**Global performance (test set, per eye):**
- PED: Dice = 0.699 (7 eyes, 7 patients)
- SRF: Dice = 0.807 (7 eyes, 7 patients)

**Dice by sex:**
- PED — Female: 0.641 ± 0.306 (n=2) | Male: 0.722 ± 0.107 (n=5) | Gap (F−M): **−0.081**
- SRF — Female: 0.765 ± 0.332 (n=2) | Male: 0.823 ± 0.227 (n=5) | Gap (F−M): **−0.058**

**Dice by age group:**
- PED — 50–59: 0.857 (n=1) | 60–69: 0.767 ± 0.120 (n=3) | 70–79: 0.577 ± 0.135 (n=3)
- SRF — 50–59: 1.000 (n=1) | 60–69: 0.784 ± 0.312 (n=3) | 70–79: 0.765 ± 0.203 (n=3)

## Key observations

1. **Consistent direction across biomarkers.** Female patients obtain a lower Dice score than male patients for *both* PED and SRF. This consistency suggests the question is worth investigating — but does not constitute a conclusion.
2. **Age may also matter.** The 70–79 group shows a notably lower PED Dice (0.577) than the 60–69 group (0.767).
3. **Global scores hide subgroup variation.** The gap in PED (~0.08 Dice) is clinically non-trivial and would be invisible in a global metric alone.

## Limitations

This is a **pilot study**, and its results must be interpreted with caution:

- **Small sample size:** only 7 patients in the test split (2 female, 5 male). No statistical conclusion can be drawn.
- **Single population:** AMD-SD contains only Asian participants, so **ethnic fairness cannot be assessed**.
- **No confidence intervals:** with 7 patients, bootstrap estimates would be unstable.
- **The observed gaps are signals, not findings.** They justify a larger study, not a conclusion.

## What this work contributes

- A **reproducible methodology** for fairness auditing in medical image segmentation: per-eye aggregation, subgroup comparison, transparent reporting of small-sample limits.
- A **preliminary signal** (consistent female disadvantage across two biomarkers) that motivates further work on the full AMD-SD dataset.

## Future work

- Apply the same protocol to the **full AMD-SD dataset** (138 patients, 156 eyes), with a patient-stratified split.
- Add **bootstrap confidence intervals by patient**.
- Extend the analysis to **intersectional subgroups** (sex × age).
- Investigate **explainability** (Grad-CAM, attention maps).

## Reference

Pearman, E., Osborne, S., Kandlikar-Bloch, M., Arzaghi, M., Carichon, F., & Farnadi, G. (2026). *Mechanisms of Bias and Reasoning: Interpreting the Impact of Chain-of-Thought on Gender Bias in LLMs.* arXiv:2605.20410.

## Notes

- Images are not included in this repository (size). The dataset can be downloaded from the official AMD-SD source.
- The test split is patient-level (no patient appears in both train and test), which avoids data leakage.