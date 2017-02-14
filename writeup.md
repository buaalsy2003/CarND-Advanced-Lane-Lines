##Udacity CarND project Writeup


---

**Advanced Lane Finding Project**

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[//]: # (Image References)

[image1]: ./output_images/calibration9_withcorner.png "Calibration Image with Corners"
[image2]: ./output_images/test5_undistort.png "Test Image Undistort"
[image3]: ./output_images/testimage_color_gradient.png "Color and Gradient"
[image4]: ./output_images/testimage_perspective.png "Perspective Transform"
[image5]: ./output_images/testimage_lanelinedetection.png "Lane Line Detection"
[image6]: ./output_images/testimage_validation.png "Validation"
[image7]: ./output_images/testimage_withlaneinfo.png "Result with lane information"
[video1]: ./result.mp4 "Result Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points
###Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
###Writeup / README

####1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

Here you go! This is my writeup!

###Camera Calibration

####1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "main.ipynb" .  

I started by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  I used `cv2.findChessboardCorners()` to get the "image points". `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection. The sample `imgpoints` is shown below:

![alt text][image1]

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test images using the `cv2.undistort()` function and obtained this result: 

![alt text][image2]

###Pipeline (single images)

####1. Provide an example of a distortion-corrected image.
The right side of above image already provided an example of a distortion-corrected image. 

####2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.
I did the perspective transform first since we are noly interested in lane line detection. Using the warped image, I used a combination of S channel, R channel and SorbelX to generate a binary image. The threshold I used is s_thresh=(160, 255), r_thresh = (220, 255), sx_thresh=(60, 80) respectively. S Channel is good at picking up yellow line and R Channel is good at picking up white line. Sobel X is good at detecting vertical lines. I used high therhod value for R channel because it brought in some noises in some images. Here's an example of my output for this step.  

![alt text][image3]

####3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform is in the 3rd code cell of main.ipynb).  The `perspective_transform()` function takes as inputs an image (`img`) and applis the `calculate_undistort()` first to undistort the image. Then it utilizes OpenCV `cv2.getPerspectiveTransform()` and `cv2.warpPerspective()` to generate the wraped image. I hardcoded the source and destination points based on provided test images (the goal is to only show two lane lines in the result):

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 595, 460      | 150, 40       | 
| 300, 680      | 150, 680      |
| 1090, 680     | 1180, 680     |
| 720, 460      | 1180, 40      |

My perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image. It can be seen in the following image:

![alt text][image4]

####4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

From the binary images with lane line pixes detected, I fit my lane lines with a 2nd order polynomial using the code in the "Find Lane Line" section of main.ipynb. 
First of all, I used the peak of histogram to find the possilbe lane line location as course materials suggested. 
Then I used the Sliding Window Search method to conduct the search for lane line pixel candidates. Any pixel within the search window will be used to fit the polynomial curve. The output image can been seen as in the left side of the following image:

![alt text][image5]

####5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I calculated the radius of curvature using the method introducted in the course materials by taking the derivatives of the 2nd order ploynomial curve. I calculated the vehicle position respecting to the center by getting a middle point of fitted lane curves and comparing it to 640, half of the image size in X direction. I added the information to the final result as shown in the image below:

![alt text][image7]

####6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I plotted the lane line polynomial curves back to the original image by applying OpenCV `cv2.warpPerspective` with inverse matrix of original perspective transformation matrix. The result on a test image can be found in the right side of the following image: 

![alt text][image5]

---

###Pipeline (video)

####1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./result.mp4)

---

###Discussion

####1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Above discussed methods are good enough to generate acceptable results for individual test images. When applying it as a video pipeline, the result is not enough. I got some bad results caused by shadows or road patches. The issue could be solved by leveraging the related information between continuous frames. 

Here are some ways I borrowed or modified from Udacity course materials "Tips and Tricks for Project":
* I used neighboring regions near the last fitted curves to search for candidate lane line pixels. The region is much smaller than the one of original window search method. It is quicker in performance and more importantly it is more reliable since it could remove lots of noise pixels. 
* I used the idea of lane line parallel to determine whether fitted curves from one frame are valid or not. For each frame, I recorded 3 sets of points, indicated by the end points of each red line in the following image. I wanted to make sure the distance change of each line between current frame and last frame should not be too big. Based on the ratio of the change, I classifed the detected curves as "good", "ok" and "bad". I kept the good and ok frames and added the fitted curves to a weighted moving average of results. Good frames were weighted higher than Ok frames. For bad frames, I discarded them and used the moving average fitted curves instead. But once I rejected consecutive 3 frames or more, I would use the original Window Search method to search for new fitted lines and use a looser standard to accept the "new" one. (I had tried to use fitted curve curvature to validify the results but found that curvature is too sensitive to minor lane line change.)
![alt text][image6] 
* I used the fitted curves from the first 5 consecutive "good" frame as my confident starting point of my moving average fitted curves. 

The result is much more reliable and smoother after applying above techniques. However, I still saw few fitted line off the real lane line a little, especially at shadow or curved lanes. There are two directions that I could improve the algorithm/pipeline. The first one is to try different color or gradient methods to reduce the shadow and noise. The second is to use other or better way to smooth the result. 



