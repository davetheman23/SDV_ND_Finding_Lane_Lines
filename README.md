# *Finding Lane Lines on the Road*
---

## Reflection

## 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

The pipeline consists of the following steps:
1. convert the image to grayscale
1. blur the image with gaussian blur so to remove some pixel noise
1. use the Canny Edge detection algorithm to convert the image with only edges
1. mask the image with a region of interest (the region is hard-coded)
1. apply Hough transform to find pixels that are connected or on the same line
1. group the hough lines together based on the slope of each line, and if a line has a slope very close to the average slope, then add it to become part of the line group
1. filter the line group by the following somewhat hacky criteria:
    * a line group should have at least 5 lines
    * a line group should have a slope (i.e. delta y / delta x) which absolute value is between 0.5 and 0.9
1. do linear regression fitting to get a slope and a point of all the line points collected for each line group
1. extrapolate the line based on the slope and point from the previous step, and use the region of interest to bound the extrapolated line
1. draw the extrapolated lines onto the original image

The Hough lines included a lot of lines. Depending on all the parameters needed by the hough line function, the resultant number of lines can be quite many. Worst yet, the biggest challenge is not knowing which lines belong to the left lane, and which lines belong to the right lane, and there are yet other lines that should not belong to either of the left or right lane groups. So some line grouping algorithm is devised. Step 6 - 7 are used for this purpose.

To do Step 9, some computation is needed from the fitted line parameters as the following:
```
def get_x(y):
    [vx, vy, x0, y0] = cv2.fitLine(self.points, cv2.DIST_L2, 0, 0.01, 0.01)
    return int(((y-y0) + vy/vx * x0)/ (vy/vx))
```


### 2. Identify potential shortcomings with your current pipeline

**Issue 1**

Step 7 is hacky because of the hard-coding, where the number of lines needed in a group and the range is just some eye-balling of better performance via trial-and-error, and this is highly dependent on the Hough transform parameters.

These hard-coding are needed because of the Hough lines are noisy, which could contain random edges that are not line-lanes. The current pipeline is difficult to detect them.

**Issue 2**

There is another shortcoming related to the extrapolation of the lines within the region of interest. Currently, simple the x's lower and upper limits of the region of interest are used to bound the line drawing. But it may not necessary really bound the line within the region of interest, especially when the region of interest is not of rectangular shape.


### 3. Suggest possible improvements to your pipeline

**For Issue 1**

Since we know that video frames that close in time shares a lot of common features. So the lane lines detected from the last frame may still be at a very similar location at the current frame. So if the line groups detected from the last frame can either be used to seed the detection in the current frame or even to be included in the line fitting in the current frame, it could potentially reduce the noise and improve the stability of the detection.

**For Issue 2**

It is entirely possible to use some polygon clipping library to clip the extrapolated lines instead of using the simple lower and upper bound methods.
