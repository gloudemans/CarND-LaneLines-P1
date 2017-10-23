# **Finding Lane Lines on the Road** 

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline consisted of the following 5 steps:

*	Detect edges in the color image
* Mask the edge detection image to blacken all but a trapezoidal region containing the driving lane.
*	Apply the probabilistic Hough transform to the masked edge detection image to identify line segments.
*	Post process the line segments to update estimated lane line positions.
*	Draw the estimated lane lines and add them to the original image.

In order to improve performance of the edge detection algorithm, I applied Canny edge detection to each color plane and combined the resulting edge set. This change helped improve detection by finding changes in color as well as brightness. I modified the draw lines function to separate updating the lane line estimates from drawing the estimated lane lines on the image since these are conceptually different operations. The Hough transform typically identifies many line segments within the masked image region. Within my lane line estimate update function I added logic to associate these line segments with the left or right lanes and to discard those having low magnitude slopes. To merge the filtered line segments into a single line lane estimate, the lane line update logic converts each surviving line segment estimate to a set of points and fits a least squares line through all points associated with the lane. In order to improve processing of the challenge video, I added state variables to store the lane line estimate from the prior frame, and I implemented logic to adjust the estimated lane line positions more smoothly using an infinite impulse response filter on the horizontal positions of the lane line endpoints. Furthermore, I added logic to discard line segments from the Hough transform when their endpoints fell too far from the existing lane line estimates. This effectively narrowed the region of interest to two small strips once the algorithm had formed an initial estimate of the lane line position. Collectively, these modifications allow the pipeline to perform reasonably well on each of the videos including the challenge video.

### 2. Identify potential shortcomings with your current pipeline

One significant shortcoming of the current pipeline is that it aggressively narrows the region of interest to include only a narrow stripe around each current estimated lane position. This narrow focus effectively discards outliers from the Hough transform when the lane line estimate is accurate, but if the lane line estimate should ever become corrupted (as will surely happen) the filtering process will begin to discard all or most of the relevant line segments and could therefore become stuck in a confused state for a prolonged period.

### 3. Suggest possible improvements to your pipeline

The most essential improvement for the current pipeline would be to provide a means to detect and recover from the shortcoming described above in which the estimated lane position diverges and the algorithm is potentially trapped by discarding relevant line segment information. One approach to correcting this deficiency would be to fit lane line estimates both with and without discarding based on proximity to the current lane line and then choose the “better” estimate as the next estimated lane line state. This begs the question as to how one might measure “better”. One possible quality measure might be the total length of all line segments in the Hough algorithm output whose endpoints are within a small distance of the resulting lane line estimate. This metric should reliably reject the fit based on a diverged lane line estimate in favor of a new fit based on consideration of additional line segments.
