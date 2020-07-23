---
layout: project
title: "Invasive Species Segmentation from Spectral Images"
author: "Greg Lee"
categories: documentation
tags: [documentation]
image: phragmites.jpg
---

Phragmites australis is an invasive plant species threatening the Howard Slough Waterfowl Management Area in Salt Lake City, Utah. This project, contracted by the Utah Department of Natural Resources, aimed to segment P. australis from sUAS images in order to track the growth of the plant. A U-Net architecture yielded ~86% test set pixel-to-pixel accuracy in segmenting aerial images for P. australis. 
<hr>
<br>
## Background
An invasive subspecies of Phragmites australis was likely introduced to Utah in the early 1980s. Since then, it has aggressively expanded, increased fire danger and caused ecological damage. The plant grows rampant in a local wildlife reserve known as the Howard Slough Waterfowl Management Area, just north of Salt Lake City. P. Australis propogates by forming monotypic stands which harm the natural habitats of many migratory birds and shrubs.

<img src="{{ site.github.url }}/assets/img/phragmites.jpg" width="350" height="250">

 The spread of P. australis decreases diversity, altering nutrient cycling and hydrology. For many years, the Utah Department of Natural Resources (DNR) has been working to remove the plant. In an effort to track the progress and effectiveness of remediation, the DNR is imaging the Howard Slough Waterfowl Management Area. This system is necessary as the environment is extremely difficult to navigate and requires a Marsh Master to avoid vehicles getting stuck in the mud. A small Unmanned Aerial System (sUAS) is ideal for monitoring the situation. The sUAS is fitted with a multispectral image sensor (blue, green, red, red-edge, and near-infrared bands) and flown at a height to produce a ground sample distance of about 7.5 cm.

 <img src="{{ site.github.url }}/assets/img/parrot.jpg" width="400" height="350">

## Methodology
The primary objective is to produce classified maps of Phragmites with reasonable accuracy. The data from the sUAS produced a single 4775 x 15282 x 5 image in the TIFF file format. Due to limits in training data and time, the image was cropped to 630 x 6792 x 5, and split by 0.5 for train and test sets.

The training data were office-interpreted and geotagged photographs were utilized as a visual aid. In remote sensing, training data is imperfect as it is near impossible to collect ground truth due to the imperfect environment capture. Office interpretation is standard practice, and as a result, remote sensing is often considered a science and an art. All interpretations were performed by Troy Saltiel as he has first-hand knowledge of the Howard Slough area, a geospatial background and local DNR ecologist connections who assisted in areas of doubt. 

We utilized the Geographic Information System software, ArcGIS Pro, to label the training data. A portion of the original image (~5% by area) was selected for its diverse representation of the area. To label the data, a segmentation tool was used to label areas containing water, soil, algae, bulrush, cattail and P. australis. The tool includes parameters for spatial and spectral detail, and these were selected to ensure similar classes, i.e. vegetation species, were separate in the segmentation. When the program struggled with segmentation due to overlapping classes or occlusion, polygons were hand-drawn. These segments and polygons were exported as 3,350 image tiles made up of 32x32x6 arrays representing the 630 x 6392 x 5 image. 

In total, approximately 70% of these labeled image tiles contained the plant species of interest. To prepare for training, the tiles and corresponding labels was split into a train (80%) and test (20%) set. These data were then oriented into a converted to a numpy array for input into a Tensorflow model. To segment the areas containing P. australis, a U-Net CNN architecture was leveraged [1]. 

 <img src="{{ site.github.url }}/assets/img/unet.png" width="460" height="300">

This model is made up of an encoder and decoder network capable of “highlighting” areas of interest in a high dimensional space before mapping those points to a segmentation map. This map is highly detailed as the pixel density is a one to one match between input and output. Tensorboard was used to do a simple hyperparameter search on the dropout rate, optimization algorithm and kernel initializer. In early iterations, there was a high degree of overfitting, as the training accuracy was significantly higher than the test accuracy. In an attempt to mitigate this issue, L2 regularization was added to several of the convolutional layers and normalization was performed on the input image data.

 <img src="{{ site.github.url }}/assets/img/visualization.png" width="700" height="320">

<br>
## Results
The cropped original sUAS raster was separated into 3,353 images, of which 70% contained at least one pixel labeled as P. Australis. Data was split using a 80/20 schema leaving 2,682 images for training and 671 images for testing. Initially, it was anticipated hold-out images would be acquired in early May, 2020 to output a final, unbiased classifier evaluation metric. Unfortunately, this was not possible due to COVID-19 circumstances. With upcoming iterations, it would be better to perform a train/test/holdout split of 70/15/15 of similar label distribution to best assess the classifiers ability to segment P. Australis. 



<br>
## Limitations
The P. australis classifer is currently limited in the accuracy of its predictions and the inability to adapt to differing input data shapes. Currently the accuracy on the test set of images is approximately 86%. With a greater amount of data from labeling or augmentation, the accuracy and generalizability should improve significantly. The original rasters have irregular shape, making it difficult to create 

Currently the classifier is capable of only processing images of dimension 630x6792x5. Unfortunately the images taken are not of square format when imaged. Preprocessing of the network should be improved so images of any size can be processed and segmented with high accuracy. Overall, this is a fantastic version one though, as it classifies the images with reasonable accuracy!

<br>
## Ethical Considerations
The project was requested by a state government agency, and while it does involve imagery collected by a “drone”, the field site is very isolated and not near any private property. Phragmites australis is a well-known nuisance, and the primary stakeholders are the state government and duck/game hunters. Both are in support of removing the plant from the ecosystem, to improve the environment for the many birds that depend on the greater Great Salt Lake ecosystem. The largest consideration is that the DNR receives accurately classified maps, however this class project is not intended to produce any deliverable product.

<br>
## Sources
[1] Ronneberger O., Fischer P., Brox T. (2015). U-Net: Convolutional Networks for Biomedical Image Segmentation. *arXiv: 1505.04597*.