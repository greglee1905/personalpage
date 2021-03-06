---
layout: project
title: "Intestinal Organoid Recognition"
author: "Greg Lee"
categories: documentation
tags: [documentation]
image: organoid.jpg
---
Organoids are small, self-organized tissues grown from stem-cell progenitors. These *ex vivo* cultures help bridge the gap between animal models and cell culture experiments as they recapitulate the cellular heterogeneity without high cost. Organoids can be viewed via confocal microscopy and counted as a measure of stem-cell viability. This project aims to recognize and quantify intestinal organoids with YOLOv3. 
<br>
<hr>

## Aims
This project aims to explore if YOLO can accurately recognize and classify organoids in real-time. As a researcher, having a tool to quickly quantify organoid images would save time in data collection and processing enabling increased efficiency and assay turn-around. To minimize the processing time, quantification should happen on the microscope camera to avoid all post-processing steps. In an ideal workflow, organoid plates would be placed on a microscope basin and quantified automatically. This project serves as a proof-of-concept to verify intestinal organoid images can be quantified with high accuracy.

## Background

### Organoids
Organoids are a system to grow and study small "mini-organs". When seeded into a gelatinous, three-dimensional extracellular matrix, freshly harvested mammalian intestinal stem cells form organoids [1]. Once plated in gel, these cells self-assemble into a epithelial monolayer surrounded by a central lumen reminiscent of the origin tissue architecture. All cell types of the intestinal niche are present. The robust nature of stem cells allows growth to occur quickly, with mature organoids showing by day seven. Organoid systems enable careful study of the healing response and stem-cell differentiation. Furthermore, they represent a hybrid between traditional animal studies and cell culture studies, enabling high through-put experiments to better study biology.
<img src="{{ site.github.url }}/assets/img/organoid_imgs/organoids_growth.png" width="600">

Organoid assays are often quantified by taking representative images of the gel "domes" which contain between 0-200 growing organoids. Images are taken by a researcher with a 2D confocal microscope. Following, images are processed to extract the number and area of each organoid. These data are then used to describe healing, differentiation, etc.

### Computer Vision
There are many types of algorithms capable of image detection although two predominant architectures are most popular - YOLO[2], Region-based Convolutional Neural Networks (R-CNNs)[3]. Each algorithm excels under certain conditions:

1. **You Only Look Once (v3)**
    - Performs feature extraction at multiple zooms utilizing a fully convolutional backbone. Per each of these convolution windows, YOLO performs classification by separating the image into a grid and predicting bounding box coordinates of any objects within. 
    - Fast - often used for video processing and automobile application
    <img src="{{ site.github.url }}/assets/img/organoid_imgs/yolov3.png" width="800">
2. **Faster R-CNNs**
    - Uses a pre-trained CNN to extract region proposals where the algorithm believes an object to exist. A seperate networks processes these regions to identify what is within. [4]
    - Better accuracy but often slower
    <img src="{{ site.github.url }}/assets/img/organoid_imgs/faster_rcnn.png" width="500">

In 2019, Kassis et al. created Orgaquant - a program capable of localizing human organoids [4]. While this methodology may result in the highest classification accuracy, it is not fast enough to perform real-time detection on a microscope. This project builds on this existing literature by extending organoid recognition to mouse systems of study and exploring the possibility of real-time, microscope-based quantification. 

## Data Collection
These images came directly from experiments performed by Greg Lee under the direction of Dr. Helong Zhao at the University of Utah. The following protocol was used to grow intestinal organoids from mice:

    C57BL/6J (wildtype) were purchased from Jackson Laboratories. All male and female mice used for intestinal crypt isolation were adults (8–10 weeks old). The isolation procedure was modified from an established protocol [5]. The jejunal sections of the mouse’s small intestine was harvested, opened longitudinally and washed with ice cold 1x phosphate buffer saline (PBS) (Sigma-Aldrich). Mucus and villi were removed using a thin glass coverslip and tissue was cut into 1 cm sections. The tissue pieces were washed in cold PBS and incubated in 30 mM ethylenediaminetetraacetic acid (EDTA) (Sigma-Aldrich), dissolved in PBS, for 8 minutes on ice. Under harsh stripping conditions, dithiothreitol (DTT) (Thermo-Fisher) was added to this solution at a concentration of 1.5 mM. Tissue pieces were transferred to PBS and incubated for 15 minutes on ice. Crypts were liberated by vigorously shaking for 3 minutes and isolated by passage through a 70 mm cell strainer. Purified crypts were pelleted by centrifugation (300 g for 5 minutes at 23 degrees C) and washed twice with PBS to remove any remaining contaminants. All animal studies were approved by University of Utah Institutional Animal Care and Use Committee under protocol number 16-05012 and 18-02010. All animal experiments were conducted in accordance with National Institute of Health Guide for the Care and Use of Laboratory Animals.

    Isolated crypts were counted then pelleted by centrifugation (300 g for 5 minutes at 23 degrees C). 1000 crypts were suspended in 80 mL of 50% growth-factor-reduced phenol-red-free Matrigel (Corning) diluted in conditioned LWRN culture medium containing WNT3a, R-spondin, and Noggin. This suspension was pipetted onto the center of a 24 well plate to create a dome. Each gel polymerized at 37°C for 20 minutes. 750 mL of media containing a treatment (singularly including b-estradiol (Sigma-Aldrich), 4-hydroxytamoxifen (Cayman), and G15 (Tocris)) was added to fully submerge the gel. Media was changed every 3 to 4 days, with treatment maintenance throughout. Organoids were counted under 4x magnification with images captured on the EVOSTM Auto Imaging System 4 and 7 days after plating. The number of organoids was normalized per each experiment.

All images used within this project represent intestinal mice organoids seven days after seeding. Organoids were subject to treatments which may affect the generalizability of the classifier. There are 10 sample images within git [repo](https://github.com/greglee1905/Organoid_Recognition/tree/master/sample_images) and the full set of data is available upon [email](greglee1905@icloud.com) request (1,200 images with bounding box labels). 
<img src="{{ site.github.url }}/assets/img/organoid_imgs/org_sample_img.jpg" width="500">

## Data Labeling: 
YOLO follows a very specific labeling method utilizing a center x, center y, width and height. In order to create labels for all images, [labelImg](https://github.com/tzutalin/labelImg) was utilized. This program allows a user to define bounding boxes and convert them into a YOLO compatible format. Center x and y refer to the center of the bounding box normalized to represent a percentage of the original height and width. All images were compressed and standardized to 256x256 pixels to accelerate model learning. Utilizing lableImg, Greg Lee and Ryan Boekholder labeled 1,239 images leveraging personal domain expertise. A variable number of organoids (0-27) were seen within the image set, with an average of five organoids per image, heavily skewed towards fewer organoids per image. 
<img src="{{ site.github.url }}/assets/img/organoid_imgs/labeled_organoid.png" width="400">

## Training YOLO
PJ Reddie, the author of the YOLOv3, released a framework built on a darknet to run YOLO. After cloning the original [repo](git clone https://github.com/pjreddie/darknet) ,images and labels were combined into a single folder and copied into the working darknet directory. Note, all labels and images must have identical nomenclature aside from extension i.e. image1.txt and image1.jpg. Data was split into a test (20%) and train (%80) set. For compatibility with darknet, the paths to these images needs to be saved to file named train.txt and test.txt. a configuration, data and names file were edited to reflect the directory structure and total number of prediction classes. All parameters were held constant except the final layer in order to make best use of the feature extractors pretrained in darknet. 

## Results and Discussion: 
Both yolov3 and yolov3-tiny were trained for comparison. Interestingly, yolov3 appeared to outperform yolov3-tiny, most likely a result of overfitting. On a small hold-out set of images, Orgnet performs with a recognition accuracy of approximately 77% and is capable some generalization as evidenced by accurate predictions on never-before-seen images from the internet. This capability was not tested in full though and requires an experimental setup for validation. 
<img src="{{ site.github.url }}/assets/img/organoid_imgs/unseen_image.png" width="700">

Orgnet has predicable areas where it performs poorly. The current iteration struggles when organoids are in the bright/darkfield of the microscope, not in focus or occluded by other organoids. These edge cases are the most challenging as most human quantifiers would struggle to accurately quantify organoids within these fields. Based on the distribution of the training set, it is logical that Orgnet performs best when the number of organoids sits below six. If more training examples were added which showcase many organoids in field, the accuracy in these cluttered cases might improve. In order to further increase the accuracy of Orgnet, more data should be collected, labeled and used for training. Scraping should be considered as an option to collect more images alongside higher degrees of image augmentation including Cutmix techniques. 
<img src="{{ site.github.url }}/assets/img/organoid_imgs/samp_pred_1.png" width="700">

Orgnet performs with high accuracy when there is minimal shadow and blur. Compared to a previously created Matlab segmentation model, this deep learning approach performs with increased robustness and accuracy. For best performance, low seeding density should be used and organoids of interest should be in focus. Furthermore, consider altering the prediction threshold for your particular scope and setup as this value can greatly affect the accuracy of the classifier. Take note, altering this value will improve the accuracy but will introduce more false positives. 
<img src="{{ site.github.url }}/assets/img/organoid_imgs/samp_pred_2.png" width="700">

Orgnet stands to benefit from improved preprocessing, video adaption and usability. This algorithm struggles when organoids are within extreme lighting or out of focus. To address this, preprocessing techniques such as singular value decomposition or Fourier filtering should be utilized to increase the contrast of organoids against the background. This may help clarify edge cases, making them more obvious. Initially, the goal of this project was to create an algorithm capable of real-time detection of organoids so microscope cameras could be equipped to count these 3D structures within a 96-well plate. To truly leverage this platform, adaptions should be made to recognize organoids in real-time. This way, a plate could be scanned and counted to increase high-throughput experimentation. Additionally with video processing, the plate could be manipulated by height to bring different planes into focus. This would allow the algorithm to more accurately quantify all organoids by reducing blurriness. While accuracy is a concern, integrating the tool into the 3D culture community remains a priority. A simple GUI implementation or microscope software patch would greatly increase the usability of Orgnet allowing those without a coding background to increase experimental efficiency. 

## Conclusion
Orgnet represents a deep-learning approach to quantify intestinal mouse organoids. This algorithm performs well when organoids are separated, in good focus and not in extreme lighting. Many improvements stand to be in pre-processing, video adaption, GUI integration and segmentation. In its current version, this application works well for quantifying intestinal mouse organoids and may be generalizable to other organoid types, depending on their visual similarity. Please forward any questions, bugs or concerns to greglee1905@icloud.com! Currently, I am working on an implementation of YOLOv4 for organoid recognition. Please check back soon for updates!


## Sources
**[1]** Sato T., Vries G., Snippert J., et al. (2009, Jul.). Single Lgr5 stem cells build crypt-villus structures in-vitro without a mesenchymal niche. Nature.459:262-265.
<br>**[2]** Redmon J. and Farhadi A. (2018, Apr.). YOLOv3: An Incremental Improvement. arXiv. 1804.02767.
<br>**[3]** Ren S., He K., Girshick R., et al. (2015, Jul.). Faster R-CNN: Towards Real-Time Object Detection with Region Proposal Networks. arXiv. 1506.01497. 
<br>**[4]** Kassis, T., Hernandez-Gordillo, V., Langer, R. et al. (2019, Aug.). OrgaQuant: Human Intestinal Organoid Localization and Quantification Using Deep Convolutional Neural Networks. Sci Rep. 9:12479
<br>**[5]** Hamilton KE, Crissey MA, Lynch JP, et al. (2015, May.). Culturing adult stem cells from mouse small intestinal crypts. Cold Spring Harbor Protocols. 1940-3402. 