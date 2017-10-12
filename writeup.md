**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./examples/grayscale.jpg "Grayscale"

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline consisted of 5 steps. First, I converted the images to grayscale, then I .... 

In order to draw a single line on the left and right lanes, I modified the draw_lines() function by ...

If you'd like to include images to show how the pipeline works, here is how to include an image: 

![alt text][image1]


### 2. Identify potential shortcomings with your current pipeline


One potential shortcoming would be what would happen when ... 

Another shortcoming could be ...


### 3. Suggest possible improvements to your pipeline

A possible improvement would be to ...

Another potential improvement could be to ...






---

cut top off at 58% b/c in the example it didn't go all the way forward either, and i don't think that far ahead, it's important for sdc decision making.

weakness: have to tune
- canny for irrelevant gradients like lines that are orthogonal to direction fo travel
- hough for how the lines are shaped -- length, dotted
- with HoughLineP, to detect curve, must reduce min line length, which adds error
- default arguments for the fn are preset for a specific set of image inputs

tuning
- increase canny high threshold to 200
- lower the top vertex of the mask triangle to 58%
- reduce hough min len to 10
- increase hough max gap to 150

noise => to reduce noise, need time-based analysis s.t. we ignore lines that "pop up and disappear". specifically, we should retain lines that are identical in slope and offset with at least a line in sevveral frames ahead and behind. If we make the window too large, however, real-time ability to look ahead would suffer.
this would be much more effective than adjusting the canny and hough parameters further, or tailoring mask shape for each image/video.



todo
use cv2.inRange for color selection



for the yellow one ...
bring down the top edge of the mask
before 
increase canny_threshold_low to exclude gradients caused by sand on the shoulder (the yellow color was actually not an issue)
reduce hough max gap for the right-hand side dotted white line.




the fact that we're extrapolating helps a lot
that and the lowering of the mask window contributed the greatest. (eg getting rid of lines in the shoulder)
this is b/c the canny and hough params were already pretty optimal.



i needed to adjust parameters before extrapolating. case in point the challenge.



wrote in an obj-oriented way b/c o/w it became impossible to reuse behavior.





luckily i had to improve the algo only once, but if i were to experiment in detail, i'd use an objective approach so i can override and reuse pieces easilier.
but, considering our goal is go come up with as universal a solution as possible, maybe it's good that we're not making it easy to tweak..... meh



there are imperfections on the static images, but extrapolating mostly solves the issue
again, time-based averaging is really needed.



# I initially tuned the thrshold as below, but when I lowered the top edge of the mask, I no longer had to.

# def process_image_yellow(image):
#     return process_image_impl(
#         image,
#         canny_threshold_low = 80,
#         hough_max_line_gap = 10
#     )



for challenge, the lower canny threshold had to be lowered to cover the yellow <-> concrete and white <-> concrete border.



curvature => crop
messy lines at the end where two lines met => crop


mask region selection should happen automatically based on hough result.






i htink my challenge time-avg code would break if the firs tframe didn't have any lines that are ahead, b/c of None, but i'm not gonna test it.




cv2 read and write image is much faster than mpl
