## ⚙️ Usage
### 🛠️ Preprocessing

#### 📌 Labeled Dataset
Organize labeled dataset as:
```text
nnUNet_raw/
└── Dataset501_MyDataset/
    ├── imagesTr/      # training images
    ├── labelsTr/      # training labels
    ├── imagesTs/      # test images (optional)
    └── dataset.json
``````
#### 📌 Unlabeled Dataset
Organize the unlabeled dataset using the same nnU-Net folder structure.  
Since these cases do not have ground-truth labels, you should create **blank NIfTI files** in the `labelsTr` folder to maintain compatibility. 
```text
nnUNet_raw/
└── Dataset502_Unlabeled/
    ├── imagesTr/      # unlabeled images
    ├── labelsTr/      # blank NIfTI files (same shape as images)
    └── dataset.json
``````
### ⚡ Run Preprocessing

Once you have organized both the **labeled** and **unlabeled** datasets into the proper nnU-Net format, run the preprocessing step for each dataset separately.

#### ✅ Labeled Dataset
Example for dataset `Dataset501_MyDataset`:

```bash
nnUNetv2_plan_and_preprocess -d 501 --verify_dataset_integrity
``````
#### ✅ Unlabeled Dataset
For unlabeled data, **do not run planning again**.  
Instead, copy the planning from the labeled dataset, rename it to match the unlabeled dataset ID, and then only run the preprocessing step.
You should be able to find the plan file here: 
"nnUNet_preprocessed/Dataset501_MyDataset/nnUNetPlans.json"

You will need to **manually create a preprocessed folder** for your unlabeled dataset, for example:
"nnUNet_preprocessed/Dataset502_Unlabeled/"

Copy this file into your unlabeled dataset’s preprocessed folder (e.g., `Dataset502_Unlabeled/`) and rename accordingly.  

Then run preprocessing using the same plan:

Example (using the plan from dataset 502):
```bash
nnUNetv2_preprocess -d 502 -c 2d
``````

### 📓 Run Training (Jupyter Notebook version)

You can also launch training directly inside a Jupyter Notebook for interactive experiments.  
Below is a minimal example using **nnFilterMatch**:

```python
import os
import torch
from batchgenerators.utilities.file_and_folder_operations import join, load_json
# from nnunetv2.training.nnUNetTrainer.nnUNetTrainer_nnfiltermatch import nnUNetTrainer_nnfiltermatch

# 1. Set nnU-Net environment variables
# os.environ['nnUNet_raw'] = '...'
# os.environ['nnUNet_preprocessed'] = '...'
# os.environ['nnUNet_results'] = '...'

# 2. Load nnU-Net plans and dataset.json (replace Dataset344 with your dataset ID)
preprocessed_dataset_folder_base = os.environ['nnUNet_preprocessed']
plans_file = join(preprocessed_dataset_folder_base, 'Dataset344_...', 'nnUNetPlans.json')
dataset_json_file = join(preprocessed_dataset_folder_base, 'Dataset344_...', 'dataset.json')

plans = load_json(plans_file)
dataset_json = load_json(dataset_json_file)

# 3. Initialize the nnFilterMatch trainer
trainer = nnUNetTrainer_nnfiltermatch(
    plans=plans,
    configuration='3d_fullres',   # use '2d' if running 2D experiments
    fold=0,
    dataset_json=dataset_json,
    device=torch.device('cuda')
)

# 4. Specify unlabeled dataset (preprocessed folder)
trainer.unlabeled_preprocessed_folder = '.../nnUNet_preprocessed/Dataset345_Unlabeled/nnUNetPlans_3d_fullres'

# 5. (Optional) Resume from checkpoint
# checkpoint_file = '.../checkpoint_best.pth'
# trainer.load_checkpoint(checkpoint_file)

# 6. Start training
trainer.run_training()

⚠️ Notes:

# Replace Dataset344_... with your labeled dataset ID.

# Replace Dataset345_Unlabeled with your unlabeled dataset folder.

# You can swap between 3d_fullres and 2d configurations depending on your setup.

# Use checkpoint_best.pth or checkpoint_final.pth if resuming training.
``````

### 🔍 Run Inference (Jupyter Notebook version)

⚠️ **Important**: Before running inference, make sure you have created  
`nnUNetTrainer_nnfiltermatch.py` inside the folder:
This ensures nnU-Net recognizes the custom trainer class.

---

You can then perform inference directly inside a Jupyter Notebook using **nnFilterMatch**.  

```python
import torch
import os
from nnunetv2.inference.predict_from_raw_data import nnUNetPredictor

# 1. Set nnU-Net environment variables
os.environ['nnUNet_raw'] = '...'
os.environ['nnUNet_preprocessed'] = '...'
os.environ['nnUNet_results'] = '...'

# 2. Initialize predictor
predictor = nnUNetPredictor(device=torch.device('cuda'))

# 3. Load trained nnFilterMatch model
predictor.initialize_from_trained_model_folder(
    '.../nnUNet_results/DatasetXXX/nnUNetTrainer__nnUNetPlans__2d',  # path to your trained model
    use_folds=(1,),                       # which folds to use
    checkpoint_name='checkpoint_best.pth' # or 'checkpoint_final.pth'
)

print("Model class:", predictor.trainer_name)  # Should print nnUNetTrainer_nnfiltermatch
print("Model loaded successfully.")

# 4. Run inference
input_folder = '...'   # must contain *_0000.nii.gz files
output_folder = '...'  # folder where predictions will be saved

predictor.predict_from_files(
    list_of_lists_or_source_folder=input_folder,
    output_folder_or_list_of_truncated_output_files=output_folder,
    save_probabilities=False,
    overwrite=True,
    num_processes_preprocessing=2,
    num_processes_segmentation_export=2,
    folder_with_segs_from_prev_stage=None,
    num_parts=1,
    part_id=0
)

print("✅ Prediction finished. Check results in:", output_folder)
``````
### 📊 Run Evaluation (Jupyter Notebook version)

After training and inference, you can evaluate your nnFilterMatch model against ground truth labels.  
Below is an example evaluation script:

```python
import os
from nnunetv2.evaluation.evaluate_predictions import compute_metrics_on_folder2

# 1. Set nnU-Net environment variables
os.environ['nnUNet_raw'] = '/path/to/nnUNet_raw'
os.environ['nnUNet_preprocessed'] = '/path/to/nnUNet_preprocessed'
os.environ['nnUNet_results'] = '/path/to/nnUNet_results'

# 2. Define paths
folder_ref = '/path/to/ground_truth_labels'  
# Ground truth labels (e.g., labelsTr/ or a subset)

folder_pred = '/path/to/predicted_segmentations'  
# Predicted segmentations (e.g., nnUNet_results/DatasetXXX/fold0)

dataset_json_file = '/path/to/dataset.json'  
# Dataset configuration file (from nnUNet dataset folder)

plans_file = '/path/to/plans.json'  
# Model plans file (from trained model folder)

output_file = '/path/to/output/summary.json'  
# Optional summary file (created if not exists)

# 3. Run evaluation
compute_metrics_on_folder2(
    folder_ref,
    folder_pred,
    dataset_json_file,
    plans_file,
    output_file,
    num_processes=4,
    chill=True   # Set to False to raise error if files mismatch
)
