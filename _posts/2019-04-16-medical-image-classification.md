---
title: Medical Image Classification
excerpt: My Final Year Project.
published: true
date: 2019-04-16
categories: project image
tags: 
---

## [LIDC-IDRI](https://wiki.cancerimagingarchive.net/display/Public/LIDC-IDRI#a2b592e6fba14f949f6e23bb1b7804cc)

- [The Lung Image Database Consortium (LIDC) and Image Database Resource Initiative (IDRI): a completed reference database of lung nodules on CT scans.](https://www.ncbi.nlm.nih.gov/pubmed/21452728)
- [Lung Image Database Consortium (LIDC) Nodule Size Report](http://www.via.cornell.edu/lidc/)


- [Highly accurate model for prediction of lung nodule malignancy with CT scans](https://arxiv.org/pdf/1802.01756.pdf)

## Bookmarks

- [Review of CAD for lung cancer](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC3995505/)
    - meet the following requirements:
        - improve the performance of radiologists providing high sensitivity in the diagnosis
        - a low number of false positives (FP)
        - high processing speed
        - present high level of automation
        - low cost (of implementation, training, support and maintenance)
        - the ability to detect different types and shapes of nodules
        - software security assurance
<https://towardsdatascience.com/computer-vision-feature-extraction-101-on-medical-images-part-1-edge-detection-sharpening-42ab8ef0a7cd>
- [Pyradiomics](https://www.nature.com/articles/ncomms5006)
- [Top placing](https://datasciencebowl.com/2017algorithms/)

### Evaluation
<https://machinelearningmastery.com/metrics-evaluate-machine-learning-algorithms-python/>
<https://www.ritchieng.com/machine-learning-evaluate-classification-model/>


## Training Model
[Image Classification using Python and Scikit-learn](https://gogul09.github.io/software/image-classification-python#training-classifiers)
[Python machine learning: Introduction to image classification](https://blog.hyperiondev.com/index.php/2019/02/18/machine-learning/)


## Online Courses
- [Stanford CS class CS231n: Convolutional Neural Networks for Visual Recognition](https://cs231n.github.io/)
- [MIT: Biomedical Signal and Image Processing](https://ocw.mit.edu/courses/health-sciences-and-technology/hst-582j-biomedical-signal-and-image-processing-spring-2007/lecture-notes/)
- [Udacity](https://www.youtube.com/watch?list=PLtizWl5sTV3d4uQ6PvzXKrlkp_3XOCotN&v=2S4nn7S8Hk4&app=desktop)

- <https://blog.paralleldots.com/data-science/research-papers-image-classification/>


## SPIE-AAPM Lung CT Challenge
```
classification
```
> quantitative image analysis methods for the diagnostic classification of malignant and benign lung nodules

- [TCIA Dataset](https://wiki.cancerimagingarchive.net/display/Public/SPIE-AAPM+Lung+CT+Challenge)
- Readings:
    1. [Special Section Guest Editorial: LUNGx Challenge for computerized lung nodule classification: reflections and lessons learned](https://www.spiedigitallibrary.org/journals/Journal-of-Medical-Imaging/volume-2/issue-02/020103/Special-Section-Guest-Editorial--LUNGx-Challenge-for-computerized-lung/10.1117/1.JMI.2.2.020103.full)
    2. [LUNGx Challenge for computerized lung nodule classification](https://www.spiedigitallibrary.org/journals/Journal-of-Medical-Imaging/volume-3/issue-04/044506/LUNGx-Challenge-for-computerized-lung-nodule-classification/10.1117/1.JMI.3.4.044506.full?SSO=1)
- [Google Cloud Citations](https://cloud.google.com/healthcare/docs/resources/public-datasets/tcia-attribution/spie-aapm-lung-ct-challenge)

Collection Statistics | 
---|---
Modalities | CT
Number of Patients | 70
Number of Studies | 70
Number of Series | 70
Number of Images | 22,489
Images Size (GB) | 12.1

- training set (10 subjects) - CT-Training
- test set (60 subjects) - LUNGx




## LUng Nodule Analysis 2016 (LUNA16)
```
detection
```
>  1. *Nodule detection (NDET)*
Using raw CT scans, the goal is to identify locations of possible nodules, and to assign a probability for being a nodule to each location. The pipeline typically consists of two stages: candidate detection and false positive reduction.
2. *False positive reduction (FPRED)*
Given a set of candidate locations, the goal is to assign a probability for being a nodule to each candidate location. Hence, one could see this as a classification task: nodule or not a nodule. Candidate locations will be provided in world coordinates. This set detects 1,162/1,186 nodules.

- [Dataset](https://luna16.grand-challenge.org/data/)
- [Preparation to Data Science Bowl 2017](https://github.com/anlthms/dsb-2017)

- [tutorial](https://github.com/booz-allen-hamilton/DSB3Tutorial/blob/master/Tutorial.ipynb)



## Data Science Bowl 2017
```
classification
```
> determine when lesions in the lungs are cancerous
- [Kaggle Dataset](https://www.kaggle.com/c/data-science-bowl-2017)
- [Discussion: Congratulations to our Top 3 Kernels Prize Winners!](https://www.kaggle.com/c/data-science-bowl-2017/discussion/31569#latest-381216)
- [2nd place solution for the 2017 national datascience bowl](https://juliandewit.github.io/kaggle-ndsb2017/)
- [lung cancer - Full Preprocessing Tutorial](https://www.kaggle.com/vanausloos/full-preprocessing-tutorial)




## Lung CT Segmentation Challenge 2017 (LCTSC)
```
detection
```
> comparison of various auto-segmentation algorithms

- [TCIA Dataset](https://wiki.cancerimagingarchive.net/display/Public/Lung+CT+Segmentation+Challenge+2017)
- [Google Cloud Citation](https://cloud.google.com/healthcare/docs/resources/public-datasets/tcia-attribution/lctsc)

Collection Statistics | 
---|---
Modalities | CT, RT
Number of Patients | 60
Number of Studies | 60
Number of Series | 96
Number of Images | 9,569
Images Size (GB) | 4.8
 



## LungCT-Diagnosis
---
```
detection, prediction
```
> extract prognostic image features that will describe lung adenocarcinomas and will associate with overall survival

- [TCIA Dataset](https://wiki.cancerimagingarchive.net/display/Public/LungCT-Diagnosis#f140375b3e21403f8c07285f1164420e)
- [Google Cloud Citation](https://cloud.google.com/healthcare/docs/resources/public-datasets/tcia-attribution/lungct-diagnosis)

Collection Statistics | 
---|---
Modalities | CT
Number of Patients | 61
Number of Studies | 61
Number of Series | 61
Number of Images | 4682
Images Size (GB) | 2.5




## Dataset

Data downloaded from [cancerimagingarchive.net](https://wiki.cancerimagingarchive.net/display/Public/Lung+CT+Segmentation+Challenge+2017#dccd4ece5b694851bdc03b4240f75f89).

``` bash
sudo apt-get install icedtea-netx

javaws /path/to/your.jnlp
```




## Breast Cancer
<https://www.kaggle.com/junkal/breast-cancer-prediction-using-machine-learning>
<https://medium.com/@Petuum/deep-learning-for-breast-cancer-identification-from-histopathological-images-f38de0a658a5>