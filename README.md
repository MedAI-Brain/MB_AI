# nnUNet for Medulloblastoma Segmentation

## Quick Start

For inference based on trained model

- Install PyTorch and nnUNet

  ```shell
  pip install torch
  git clone https://github.com/MIC-DKFZ/nnUNet.git
  cd nnUNet && git checkout nnunetv1 && pip install -e .
  ```

  Please notice that we use nnUNet-version-1 instead of current version-2.

- Create and set workdirs for nnUNet

  Create `nnUNet_workdir` and three subfolders, please read the documentation
  of [nnUNetv1](https://github.com/MIC-DKFZ/nnUNet/tree/nnunetv1) for details. If you only want to test the model, you
  needn't create any files or folds under `nnUNet_raw_data` and `nnUNet_preprocessed`. We provide `RESULTS_FOLDER` which
  contains trained model's checkpoints.

  ```shell
  nnUNetData
  	|__  nnUNet_raw_data
  	|  |__  Task504_NativeTumor
  	|    |__  imagesTr
  	|    |__  imagesTs(optional)
  	|    |__  labelsTr
  	|    |__  dataset.json
  	|__  nnUNet_preprocessed
  	|__  RESULTS_FOLDER
  ```

  Set the environment variables for nnUNet

  ```shell
  export nnUNet_raw_data_base="nnUNetData"
  export nnUNet_preprocessed="nnUNetData/nnUNet_preprocessed"
  export RESULTS_FOLDER="nnUNetData/RESULTS_FOLDER"
  ```

- Download the checkpoints

  Please download trained checkpoints via this link. It is a zip file of `RESULTS_FOLDER` , providing 5-fold trained
  checkpoints and plans of model architecture and preprocessing.

- Dataset Conversion

  The modalities of the data should be `T1-enhanced` and `T2`. And the two modalities of one subjects should be aligned,
  with same resolutions, origins and directions. Please refer nnUNetv1 to rename your nifti files. This is an example.

  ```shell
  imagesTsYOURS
  	|__  Native_001_0000.nii.gz (T1-enhanced)
  	|__  Native_001_0001.nii.gz (T2)
  	|__  Native_002_0000.nii.gz
  	|__  ...
  ```

- Predict segmentation masks

  We use the `3d_fullres` architecture of nnUNet. It is a 3d convolution network.

  ```shell
  nnUNet_predict -i [PATH OF TEST DATASET] -o [OUTPUT PATH] -t 504 -m 3d_fullres
  ```

​ The parameter `504` is th ID of our experiment.

## Training and Testing Pipeline

#### Dataset and Experiment preparation

Please refer [nnUNetv1](https://github.com/MIC-DKFZ/nnUNet/tree/nnunetv1) for details.

#### Training

- Installation and experiment variables are same with **Quick Start**.

- Please
  refer [generate_dataset_json](https://github.com/MIC-DKFZ/nnUNet/blob/nnunetv1/nnunet/dataset_conversion/utils.py) to
  create `dataset.json` for training dataset.

- Start five-fold training (3D model: 3d_fullres; 2D model: 2d)

  ```shell
  CUDA_VISIBLE_DEVICES=0 nnUNet_train [3d_fullres/2d] nnUNetTrainerV2 TaskXXX_name 0 --npz
  CUDA_VISIBLE_DEVICES=1 nnUNet_train [3d_fullres/2d] nnUNetTrainerV2 TaskXXX_name 1 --npz
  CUDA_VISIBLE_DEVICES=2 nnUNet_train [3d_fullres/2d] nnUNetTrainerV2 TaskXXX_name 2 --npz
  CUDA_VISIBLE_DEVICES=3 nnUNet_train [3d_fullres/2d] nnUNetTrainerV2 TaskXXX_name 3 --npz
  CUDA_VISIBLE_DEVICES=4 nnUNet_train [3d_fullres/2d] nnUNetTrainerV2 TaskXXX_name 4 --npz
  ```

- Find best configuration of five-fold cross training (Optional)

  ```shell
  nnUNet_find_best_configuration -m 2d -t XXX –strict
  ```

#### Testing

```shell
nnUNet_predict -i [PATH OF TEST DATASET] -o [OUTPUT PATH] -tr nnUNetTrainerV2 -ctr nnUNetTrainerV2CascadeFullRes -m [3d_fullres/2d] -p nnUNetPlansv2.1 -t XXX
```

Please notice that you don't have to make `dataset.json` for testing dataset.
