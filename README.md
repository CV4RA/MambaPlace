[![PWC](https://img.shields.io/endpoint.svg?url=https://paperswithcode.com/badge/mambaplace-text-to-point-cloud-cross-modal/visual-place-recognition-on-kitti360pose)](https://paperswithcode.com/sota/visual-place-recognition-on-kitti360pose?p=mambaplace-text-to-point-cloud-cross-modal) <a href="https://arxiv.org/pdf/2408.15740"><img src="https://img.shields.io/badge/Paper-pdf-<COLOR>.svg?style=flat-square" /></a> 

<p align="center">
<h1 align="center">MambaPlace: Text-to-Point-Cloud Cross-Modal Place Recognition with Attention Mamba Mechanisms</h1>
 <p align="center">
Tianyi Shang, Zhenyu Li*, Wenhao Pei, Pengjie Xu, ZhaoJun Deng, and Fanchen Kong
</p>

This repository is the official implementation of MambaPlace [paper](https://arxiv.org/pdf/2408.15740), also see another [implement](https://github.com/nuozimiaowu/MambaPlace/tree/main).  🔥🔥🔥

###  Introduction
  In future smart cities, autonomous vehicles, drones, and intelligent logistics systems will rely heavily on accurate localization from human language descriptions for effective path planning. Traditional visual place recognition (VPR) methods, which depend on cameras or radar to extract features from 2D images or point clouds, struggle with efficiency in human-computer interaction and lack precision under varying environmental conditions. A promising alternative is the text-to-point-cloud localization approach, which enables accurate localization without requiring proximity to the location and is resilient to changes in the natural environment. However, this method faces challenges such as ambiguous language descriptions and similar descriptions for different positions within the same region. Existing solutions, like Text2Pos and Text2loc, have made progress but still fall short in fully integrating multimodal data. To address these issues, we propose the Mamba model, a unified approach using Selective State Space Models (SSM) to enhance feature representation and improve localization accuracy.

###  Structure overview
![image](https://github.com/user-attachments/assets/b7949c7d-3481-4149-89b5-69ee873c9fac)

###  Experimental performance
![image](https://github.com/user-attachments/assets/e44eff5f-b26e-4b65-abf1-438e23c9f23e)

##  Installation
Create a conda environment and install basic dependencies:
```bash
git clone https://github.com/nuozimiaowu/MambaPlace
cd MambaPlace

conda create -n mambaplace python=3.10
conda activate mambaplace

# Install the according versions of torch and torchvision
conda install pytorch==1.11.0 torchvision==0.12.0 torchaudio==0.11.0 cudatoolkit=11.3 -c pytorch

# Install required dependencies
CC=/usr/bin/gcc-9 pip install -r requirements.txt
```

## Datasets & Backbone

The KITTI360Pose dataset is used in our implementation.

For training and evaluation, we need cells and poses from Kitti360Pose dataset.
The cells and poses folder can be downlowded from [HERE](https://cvg.cit.tum.de/webshare/g/text2pose/KITTI360Pose/k360_30-10_scG_pd10_pc4_spY_all/)  

In addtion, to successfully implement prototype-based map cloning, we need to know the neighbors of each cell. We use direction folder to store the adjacent cells in different directions. 
The direction folder can be downloaded from [HERE](https://drive.google.com/drive/folders/15nsTfN7oQ2uctghRIWo0UgVmJUURzNUZ?usp=sharing)  

If you want to train the model, you need to download the pretrained object backbone [HERE](https://drive.google.com/file/d/1j2q67tfpVfIbJtC1gOWm7j8zNGhw5J9R/view?usp=drive_link):

The KITTI360Pose and the pretrained object backbone is provided by Text2Pos ([paper](https://arxiv.org/abs/2203.15125), [code](https://github.com/mako443/Text2Pos-CVPR2022))

<!-- ```bash
mkdir checkpoints/k360_30-10_scG_pd10_pc4_spY_all/
wget https://cvg.cit.tum.de/webshare/g/text2pose/pretrained_models/pointnet_acc0.86_lr1_p256.pth
mv pointnet_acc0.86_lr1_p256.pth checkpoints/
``` -->

The final directory structure should be:
```
│Text2Loc/
├──dataloading/
├──datapreparation/
├──data/
│   ├──k360_30-10_scG_pd10_pc4_spY_all/
│       ├──cells/
│           ├──2013_05_28_drive_0000_sync.pkl
│           ├──2013_05_28_drive_0002_sync.pkl
│           ├──...
│       ├──poses/
│           ├──2013_05_28_drive_0000_sync.pkl
│           ├──2013_05_28_drive_0002_sync.pkl
│           ├──...
│       ├──direction/
│           ├──2013_05_28_drive_0000_sync.json
│           ├──2013_05_28_drive_0002_sync.json
│           ├──...
├──checkpoints/
│   ├──pointnet_acc0.86_lr1_p256.pth
├──...
```


## Train
After setting up the dependencies and dataset, our models can be trained using the following commands:

### Train Global Place Recognition (Coarse)

```bash
python -m training.coarse --batch_size 64 --coarse_embed_dim 256 --shuffle --base_path ./data/k360_30-10_scG_pd10_pc4_spY_all/   \
  --use_features "class"  "color"  "position"  "num" \
  --no_pc_augment \
  --fixed_embedding \
  --epochs 20 \
  --learning_rate 0.0005 \
  --lr_scheduler step \
  --lr_step 7 \
  --lr_gamma 0.4 \
  --temperature 0.1 \
  --ranking_loss contrastive \
  --hungging_model t5-large \
  --folder_name PATH_TO_COARSE
```

### Train Fine Localization

```bash
python -m training.fine --batch_size 32 --fine_embed_dim 128 --shuffle --base_path ./data/k360_30-10_scG_pd10_pc4_spY_all/ \
  --use_features "class"  "color"  "position"  "num" \
  --no_pc_augment \
  --fixed_embedding \
  --epochs 35 \
  --learning_rate 0.0003 \
  --fixed_embedding \
  --hungging_model t5-large \
  --regressor_cell all \
  --pmc_prob 0.5 \
  --folder_name PATH_TO_FINE
```

## Evaluation

### Evaluation on Val Dataset

```bash
python -m evaluation.pipeline --base_path ./data/k360_30-10_scG_pd10_pc4_spY_all/ \
    --use_features "class"  "color"  "position"  "num" \
    --no_pc_augment \
    --no_pc_augment_fine \
    --hungging_model t5-large \
    --fixed_embedding \
    --path_coarse ./checkpoints/{PATH_TO_COARSE}/{COARSE_MODEL_NAME} \
    --path_fine ./checkpoints/{PATH_TO_FINE}/{FINE_MODEL_NAME} 
```

### Evaluation on Test Dataset

```bash
python -m evaluation.pipeline --base_path ./data/k360_30-10_scG_pd10_pc4_spY_all/ \
    --use_features "class"  "color"  "position"  "num" \
    --use_test_set \
    --no_pc_augment \
    --no_pc_augment_fine \
    --hungging_model t5-large \
    --fixed_embedding \
    --path_coarse ./checkpoints/{PATH_TO_COARSE}/{COARSE_MODEL_NAME} \
    --path_fine ./checkpoints/{PATH_TO_FINE}/{FINE_MODEL_NAME} 
```
## Acknowledgemengt
We borrowed some code from [Textpos](https://openaccess.thecvf.com/content/CVPR2022/html/Kolmet_Text2Pos_Text-to-Point-Cloud_Cross-Modal_Localization_CVPR_2022_paper.html) and [Text2Loc](https://openaccess.thecvf.com/content/CVPR2024/html/Xia_Text2Loc_3D_Point_Cloud_Localization_from_Natural_Language_CVPR_2024_paper.html), and we would like to thank them for their help!

## Note
We encourage anyone to use our code for further research, but please cite [our paper](https://arxiv.org/pdf/2408.15740) when doing so. Thank you!🙇‍

@article{Mambaplace,
  title={MambaPlace: Text-to-Point-Cloud Cross-Modal Place Recognition with Attention Mamba Mechanisms},
  author={Shang, Tianyi, Li, Zhenyu*, Pei, Wenhao, Xu, Pengjie, Deng, ZhaoJun, Kong, Fanchen},
  journal={arXiv preprint arXiv:2408.15740},
  year={2024}
}