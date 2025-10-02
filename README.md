Please read the guide for usage


# 🗂️ Main

The **Main** folder contains the core implementation of **nnFilterMatch** in two forms:

- **📓 Jupyter Notebook version**  
  Located in `notebooks/nnFilterMatch_demo.ipynb`.  
  This is a detail version for quickly trying out nnFilterMatch.  

- **🐍 Python package version**  
  Located in `Main/nnFilterMatch/`.  
  This includes the complete implementation of the nnFilterMatch framework as a training class (`nnUNetTrainer_nnfiltermatch.py`) that integrates directly with [nnU-Net v2](https://github.com/MIC-DKFZ/nnUNet).
  

## 📂 Benchmark Datasets

Other folders in this repository contain **benchmark dataset information** used in our experiments. 
These include dataset splits and configuration details. 
- **ACDC** – Automated Cardiac Diagnosis Challenge  
- **LA** – Left Atrium segmentation dataset  
- **SegTHOR** – Thoracic organ segmentation
- **Prostate** – Prostate segmentation 


## 📚 Citation
1. B. Zhao, C. Wang, and S. Ding, “CrossMatch: Enhance Semi-Supervised Medical Image Segmentation with Perturbation Strategies and Knowledge Distillation,” IEEE Journal of Biomedical and Health Informatics, pp. 1–13, Jan. 2024, doi: 10.1109/jbhi.2024.3463711.
2. Y. Wang, Z. Li, L. Qi, Q. Yu, Y. Shi, and Y. Gao, “Balancing Multi-Target Semi-Supervised Medical Image Segmentation with Collaborative Generalist and Specialists,” IEEE Transactions on Medical Imaging, p. 1, Jan. 2025, doi: 10.1109/tmi.2025.3557537.
3.  X. Liu, W. Li, and Y. Yuan, “DifFRECT: Latent diffusion label rectification for semi-supervised medical image segmentation,” in Lecture notes in computer science, 2024, pp. 56–66. doi: 10.1007/978-3-031-72390-2_6.


## 🙏 Acknowledgments

This project makes use of several **publicly available datasets**.  
We gratefully acknowledge the data providers and research communities for making these resources accessible and supporting open science.  

Our implementation is built upon the **[nnU-Net](https://github.com/MIC-DKFZ/nnUNet)** framework  
(Isensee et al., *Nature Methods*, 2021). We sincerely thank the authors for releasing their code to the community,  
which has been instrumental in the development of this work.
