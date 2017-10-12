**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report

---

## Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My initial pipeline is coded in `def process_image` and has these steps:
- Filter the input image by yellow, b/c we care only about contrast between line colors of {white, yellow} and background colors of {asphalt, concrete}
- Apply Gaussian blurring
- Extract points at high gradient using Canny
- Mask out all but the region of interest
- Extract line segments using Hough

My improved pipeline extrapolates one line in each of left and right halves of the image.
It is coded in `def process_image_extrapolated` and has these additional steps:
- Group Hough line segments by left and right halves of the image
- Extrapolate one straight line out of all line segments, per half

My bonus pipeline adds time-sensitivity to the processing.
It is coded in `def process_image_time_avged` and has this additional logic:
- Remember the left and right extrapolated lines one frame ago and two frames ago
- When processing a new left/right pair of extrapolated lines, average out the left line with those from the most recent 2 frames, and do the same with the right line

I changed the flow of the notebook a bit. Mine looks like this:
1. Draw pre-extrapolated Hough line segments on images + videos
1. Iteratively improve the initial pipeline, and repeat the above
1. Draw extrapolated lines on images + videos
1. Iteratively improve the improved pipeline, and repeat the above
1. Draw time-averaged extrapolated lines on the challenge video
1. Iteratively improve the bonus pipeline, and repeat the above

I found it necessary to see the line segments on videos in order to:
- validate that undesired line segments are sufficiently ignored, in anticipation of working on extrapolation
- find parameters that needed to be customized for specific images, e.g. the mask region for the challenge video

#### Results

For the outputs, please refer too these directories:

```
./test_images_output_preextrapolated
./test_videos_output_preextrapolated
./test_images_output_extrapolated
./test_videos_output_extrapolated
```


### 2. Identify potential shortcomings with your current pipeline

The whole processing is very sensitive to parameters, which should therefore be hand-picked.

Luckily I was able to reuse the parameters for Gaussian blur, Canny, and Hough from the answers to the previous exercises (see the default argument values to function `process_image`), and I found them to be the best for all images and frames.

However, they're not perfect, as the yellow line video and the challenge video show. The performance of any one set of parameters is dependent on the ranges of color I encounter. It is hard to distinguish light-brown sand on the shoulder with the yellow line sometimes; and with concrete as the background, the gradient becomes flat and/or noisy.

For the yellow line video, I did try adjusting by
- increasing canny_threshold_low to 80, to exclude gradients caused by sand on the shoulder
- reducing hough_min_line_len to 20 and increasing hough_max_line_gap to 100, to better identify the dotted white line on the right
But after playing with them, I reverted them.

It turned out far more critical to clean up the result of Hough:
- Reduce the region of interest
- Eliminate lines that are too orthogonal to the direction of travel
- Extrapolate, i.e. average line segments by space
- Average the extrapolated lines by time

For example, as the top edge of the region of interest was lowered and narrowed slightly, many noisy Hough lines at the "end of the road" were eliminated, resulting in much cleaner extrapolated lines. This is ok in practical terms if we're not travelling too fast.

But, had I encountered any sharper turn, any incline, bumps, or wider lanes, or if the camera was not centered or zoomed, I would not have been able to keep using the same region of interest. In fact, case in point, the challenge video shows the bonnet, which I had to crop out.

#### Results

Note, in [this image](test_images_output_preextrapolated/whiteCarLaneSwitch.jpg), I left a little imperfection, but I chose not to optimize for this image, and let space- and time-averaging remove the error instead.

#### Aside

I think my time-averaging code would break if the first frame didn't yield both left and right extrapolated lines. If you test my code with such videos, please crop till the first frame has both.


### 3. Suggest possible improvements to your pipeline

Parameters could be adjusted dynamically:

- If a curve is detected, either reduce the height of the region of interest (i.e. reduce visiblity); or, fit a curve using `np.polyfit(..., deg = 2)` with `deg` of 2 or higher.
- After the region of interest is selected, detect color distribution, apply a different color filtering to the original image, and re-do whole pipeline in a second pass.
- When extrapolating left/right curves, retain a history of errors while fitting using `np.polyfit`. If the past few frames have been noisy, 1) increase number of frames to average over, and 2) give less weight to the noisy items.

Also, convolution neural net all the things?

Finally, I think an OO style would be easier to read than passing around overriding functions.
