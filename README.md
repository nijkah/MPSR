# Multi-scale Positive Sample Refinement for Few-shot Object Detection in PyTorch 1.1.0

Our code is based on  [https://github.com/facebookresearch/maskrcnn-benchmark](https://github.com/facebookresearch/maskrcnn-benchmark) and developed with Python 3.6.5 & PyTorch 1.1.0.

## Todo:
1. ~~Detection core of MPSR: 07/08.~~
2. ~~Few-shot scripts for Pascal VOC experiments: 07/12.~~
3. README/INSTALL/detailed notations: 07/17.
4. Few-shot scripts for MS COCO experiments: 07/20.

# Abstract
Few-shot object detection (FSOD) helps detectors adapt to unseen classes with few training instances, and is useful when manual annotation is time-consuming or data acquisition is limited.
Unlike previous attempts that exploit few-shot classification techniques to facilitate FSOD, this work highlights the necessity of handling the problem of scale variations, which is challenging due to the unique sample distribution.
The lack of labels of novel classes leads to a sparse scale space which may be totally divergent from the original distribution of abundant training data. 
To this end, we propose a Multi-scale Positive Sample Refinement (MPSR) approach to enrich object scales in FSOD. 
It generates multi-scale positive samples as object pyramids and refines the prediction at various scales. 
We demonstrate its advantage by integrating it as an auxiliary branch to the popular architecture of Faster R-CNN with FPN. 

<div align=center>
<img src="https://github.com/jiaxi-wu/MPSR/blob/master/tools/fewshot_exp/MPSR_arch.jpg">
</div>

The whole detection framework for training consists of Faster R-CNN with FPN and the refinement branch working in parallel while sharing the same weights.
For a given image, it is processed by the backbone network, RPN, RoI Align layer, and the detection head in the standard two-stage detection pipeline. 
Simultaneously, an independent object extracted from the original image is resized to different scales as object pyramids. 
We manually select the corresponding scale level of feature maps and the fixed center locations as positives for each object, keeping it consistent with the standard FPN assigning rules.
After selecting specific features from these feature maps, we feed them directly to the RPN head and the detection head for refinement.

# Installation
Check INSTALL.md for installation instructions. Since maskrcnn-benchmark has been deprecated, please follow these instructions carefully (e.g. version of Python packages).

# Prepare datasets

## Prepare original Pascal VOC & MS COCO datasets
First, you will need to download the VOC & COCO datasets.
We recommend to symlink the path to the coco dataset to `datasets/` as follows

We use `minival` and `valminusminival` sets from [Detectron](https://github.com/facebookresearch/Detectron/blob/master/detectron/datasets/data/README.md#coco-minival-annotations) ([filelink](https://dl.fbaipublicfiles.com/detectron/coco/coco_annotations_minival.tgz)).

```bash
# symlink the coco dataset
cd ~/github/maskrcnn-benchmark
mkdir -p datasets/coco
ln -s /path_to_coco_dataset/annotations datasets/coco/annotations
ln -s /path_to_coco_dataset/train2014 datasets/coco/train2014
ln -s /path_to_coco_dataset/test2014 datasets/coco/test2014
ln -s /path_to_coco_dataset/val2014 datasets/coco/val2014

# for pascal voc dataset:
ln -s /path_to_VOCdevkit_dir datasets/voc
```

## Prepare base and few-shot datasets
For a fair comparison, we use the few-shot datasets from [Few-shot Object Detection via Feature Reweighting](https://github.com/bingykang/Fewshot_Detection) as a standard evaluation.
To download their datasplits and transfer it into maskrcnn-benchmark style, you need to run this script:
```bash
bash tools/fewshot_exp/datasets/init_fs_dataset_standard.sh
```
This will also generate the datasets on base classes for base training.

# Training and Evaluation
4 scripts are used for full splits experiments and you can modify them later.
```bash
tools/fewshot_exp/
├── train_voc_base.sh
├── train_voc_standard.sh
├── train_coco_base.sh
└── train_coco_standard.sh
```
You may need to change GPU device which is `export CUDA_VISIBLE_DEVICES=0,1` by default.

## Perform few-shot training on VOC dataset
1. Run the following for base training on 3 VOC splits
```bash
bash tools/fewshot_exp/train_voc_base.sh
```
(explanation here later)

This will generate base models (e.g. `model_voc_split1_base.pth`) and corresponding pre-trained models (e.g. `voc0712_split1base_pretrained.pth`).

2. Run the following for few-shot fine-tuning
```bash
bash tools/fewshot_exp/train_voc_standard.sh
```
(expalnation here later)

This will perform evalution on 1/2/3/5/10 shot of 3 splits. 
Result folder is `fs_exp/voc_standard_results` by default, and you can run `python tools/fewshot_exp/cal_novel_voc.py` for a quick summary.

## Perform few-shot training on COCO dataset

