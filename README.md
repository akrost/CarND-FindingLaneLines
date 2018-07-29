# **Finding Lane Lines on the Road** 

## README

### **Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)
[img_original]: ./examples/01_original_image.jpg "Original Image"
[img_grayscale]: ./examples/02_grayscale.jpg "Grayscale Image"
[img_white_mask]: ./examples/03_white_mask.jpg "White Mask"
[img_yellow_mask]: ./examples/04_yellow_mask.jpg "Yellow Mask"
[img_combined_masks]: ./examples/05_combined_masks.jpg "Combined Masks"
[img_gray_with_color_mask]: ./examples/06_grayscale_with_color_mask.jpg "Grayscale Image with Color Mask"
[img_gaussian]: ./examples/07_gaussian_blur.jpg "Gaussian Blur"
[img_canny]: ./examples/08_canny.jpg "Canny Edge Detection"
[img_roi]: ./examples/09_region_of_interest.jpg "Region of Interest"
[img_hough]: ./examples/10_hough_lines.jpg "Hough Lines"
[img_clustered_lines]: ./examples/12_clustered_lines.jpg "Clustered Lines"
[img_clustered_lines_on_original]: ./examples/13_clustered_lines_on_original.jpg "Clustered Lines on Original Image"

---

**Requirements**

[Anacoda 3](https://www.anaconda.com/download/) is installed on your machine.

---
### **1. Getting started**

1. Clone repository:<br/>
```sh
git clone https://github.com/akrost/CarND-FindingLaneLines.git
cd carnd-findinglanelines
```

2. Create and activate Anaconda environment:
```sh
conda create --name carnd-p1
source activate carnd-p1
```
Activating the environment may vary for your OS.

3. Install packages:
```sh
pip install -r requirements.txt
```

4. Run project
```sh
jupyter notebook P1.ipynb
```

---

### **2. The pipeline**

The pipeline consists of 10 steps. 
1. Import the original image<br/>
This is the original colored input image. If the pipeline is used to detect lines in video clips, the original images are the frames from the clip.<br/>
![Original Image][img_original]


2. Convert original image to grayscale<br/>
This step converts a three channel (RGB) image into a one channel image. This makes the processing in the next steps much easier. But obviously it also reduces the information content from the image. Therefore step 3 is crucial for a robust line detection.<br/>
![Grayscale Image][img_grayscale]

3. Create color mask<br/>
After using a noise reduction method (step 5), lane markings are often times hard to distinguish from the pavement solely by their color. This is especially true for bright road surfaces and difficult environmental conditions (like shadows etc.). To make the lines stand out from the rest of the road, we create a color mask that highlights white and yellow colors in the picture. This way there is a higher color gradient between lane lines and the pavement an thus the lines are detected more easily.<br/>
    1. Create a white color mask based on original image<br/>
    The OpenCV function `inRange()` highlights all the pixels that are within a given color range. For the white color mask, this range is defined as `boundaries = ([200, 200, 200], [255, 255, 255])`. All pixels that are brighter than rgb(200, 200, 200) are used as a mask.<br/>
    ![White Mask][img_white_mask]

    2. Create a yellow color mask based on original image<br/>
    Same as step 3.1 except with other boundaries. For the yellow mask `boundaries = ([200, 200, 0],[255,255,125])` where used.<br/>
    ![Yellow Mask][img_yellow_mask]

    3. Combine the two color masks<br/>
    The two color masks are combined by using a bitwise or-operator on the two masks.<br/>
    ![Combined Masks][img_combined_masks]

4. Merge grayscale image with combined color masks<br/>
The color mask from step 3 is merged with the grayscale image of step 2. Comparing the following image with the image form step 2, one can see that the lane lines are highlighted. This will make it easier to detect the lines in the next steps.<br/>
![Grayscale Image with Color Mask][img_gray_with_color_mask]

5. Apply Gaussian Blur on the image<br/>
The [Gaussian blur](https://en.wikipedia.org/wiki/Gaussian_blur) helps to reduce noise in the image. Noise would lead to falsely detected edges in the next steps.<br/>
![Gaussian Blur][img_gaussian]

6. Apply Canny edge detection<br/>
[Canny edge detection](https://en.wikipedia.org/wiki/Canny_edge_detector) uses intensity gradients to find edges in images. This explains why it is helpful to use a color mask (step 3). <br/>
![Canny Edge Detection][img_canny]

7. Set region of interest<br/>
A polygon (in this case a trapezoidal shaped polygon) is used to filter the region of interest. This avoids false matches in the environment but also limits the lane line detection to a very narrow region.<br/>
![Region of Interest][img_roi]

8. Find Hough lines<br/>
The [Hough transformation](https://en.wikipedia.org/wiki/Hough_transform) helps finding lines from a set of points (the output of the Canny edge detection)<br/>
![Hough Lines][img_hough]

9. Cluster Hough lines<br/>
The previous step tends to find multiple line for both sides of the lane. The clustering first groups those lines into two categories, left and right lines based on their slope. After that the lines of both clusters are aggregated using a mean slope and the min and max y-value. If there is no Hough line in either of the two clusters, the line of the previous frame is used!<br/>
The result are two lines, one on the left and one on the right side of the lane.<br/>
![Clustered Lines][img_clustered_lines]

10. Overlay clustered lane lines on original image<br/>
In the last step the clustered line from step 9 are merged with the original input image.<br/>
![Clustered Lines on Original Image][img_clustered_lines_on_original]

<br/>

It is possible to introduce more steps to the pipeline, for example change the brightness of the grayscale iamge or use more than two color masks. 

---

### **3. Shortcomings**

- Robustness:<br/>
    Algorithm is not robust against shadows, changing road surfaces etc.
- Phantom lines:<br/>
    Current implementation keeps lines from a previews frame if no line was found in latest frame. This might cause pantom lines. 
- Region of interest (ROI): <br/>
    - The ROI is currently defined by a static set of parameteres. One might want to have a dynamic ROI, dependening on the current steraring angle...
    - With the current implementation the shape of the ROI is limited to rectangular and trapezoidal shapes. If the stearing angle is given, a ROI in
    the shape of a parallelogram might make more sense.

---

### **4. Possible improvements**

- Fine tune parameteres:<br/>
    Fine tuning the parameters of the algorithm can lead to a better performance for changing conditions in lighting and changing pavement colors.
- Support more than two lines:<br/>
    Currently only two lines are supported. The lines are clustered based on their slope. If more than two lines are present in the ROI, the resulting
    lines will be an average of all lines. For real world use-cases it would be useful to not only detect lane lines of the currently used lane, but also for 
    neighbor lanes (for example for overtaking, high precise location, etc.) 
- Support parallelogram-shaped ROI:<br/>
    When driving a curve, the ROI changes depending on the stearing angle. Supporting a parallelogram-shaped ROI would yield in better lane line detection 
    while driving along curves.
- Support curved lines:<br/>
    The current implementation only supports straight lines. For different use cases it may be required to support curved lines as well.
