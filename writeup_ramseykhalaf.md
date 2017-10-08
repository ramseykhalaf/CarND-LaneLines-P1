# **Finding Lane Lines on the Road**
## Ramsey Khalaf

---

**Finding Lane Lines on the Road**
[//]: # (Image References)

[image1]: ./examples/grayscale.jpg "Grayscale"

---

### Reflection

### 1. Approach

My approach was to try and isolate, understand and develop each stage at a time. It can be very difficult to understand what's going on and tune parameters when all you have is the first input and final output images. There will be issues that might seem to be from the line detection stage, when actually the edge detection was not quite right. Therefore I tried to isolate each stage, and tune it separately. For example, I tried to complete the edge detection to where edges all markings were visible clearly. Then I moved onto the next layer and tuned the parameters for that layer alone, so as not to loose a handle on where issues are rising from.

In order to develop I needed two things, one was good visualisation, and the other quick feedback on changes by reducing iteration time.

I first set up some helpers to visualise the outcome of each stage. I created a method to draw images side by side at the various stages of the pipeline. I also gathered all parameters into a cell just above the visualisation, to quickly change and see results without needing to scroll.

I tried to keep my code clean and refactored during development. I split my code into functions to be able to isolate inputs and outputs to debug issues more easily.

### 2. Pipeline

My pipeline consisted of the following steps.

- Greyscale
- Canny edge detection
- Region on interest mask
- Line detection
- Isolation of right vs left lane lines
- Averaging of lines
- Extrapolation/extending of lines

The first 4 were presented in class, and my approach followed those, with small param changes.

**Isolating Left vs Right lines**
I simple divided the image in half, and this worked. No lines bridged the halves, so I just checked one point from each line, if it was left of centre if was a left line.

**Averaging lines**
To average the lines I tried 3 approaches:

1. Mean start and end points - average `(x1,y1)` `(x2,y2)`
2. Mean gradients and intercepts - average `m`, `c`
3. Median gradients and intercepts - middle `m`, `c`

The average points was influenced too heavily by the cluster of small lines that would form further in the distance, it made for a jittery output on the video. It seemed the line would pivot around a point about 2/3 of the way up.

The median gradients was also jittery, as though I was selecting the middle gradient, which line this corresponded to could vary a lot from frame to frame.

Therefore the mean gradients was the best performing that I tried.


### 3. Potential Issues

There are several shortcomings I can think of with my approach so far:

- I rely on the markings being clearly visible - not all roads have good (or any) lane markings.

 - I am fitting a line to represent lanes - this may be an issues with curves.

- Region of interest has hard coded sides - sharp corners may exceed these limits and will not be visible.

- The region of interest horizon is hardcoded - bumps up and down, ramps, crests, all my move this and cause additional line segments to be found that should not be.

- Cars in front or changing lanes could obscure markings

- Weather (e.g. snow/sand) could cover markings.

- Night may make the lane markings hard to see - perhaps cat's eyes, not the white markings could work better?

### 4. Possible Small Improvements

There are a few immediate ideas for improvement I have:

- Removing outliers from smaller lines before taking the mean, so as to make the average line more robust to noise.

- Averaging in time (between frames) in video. As each frame is nearly the same as the last, there should only be a small change each time. Larger changes are probably due to noise. Perhaps larger jumps in 1 frame be smoothed or rejected.

- Using a different line fitting algorithm. One that comes to mind is considering fitting a line via linear regression to the output from the edge detection.

### 5. Probabilistic modelling

One thing that I became frustrated with was how sensitive the output was to small changes in the params. E.g. a small change in `max_line_gap` would suddenly cause a line to bridge the gap between left and right (an obvious mistake).

Would it be possible to use a probabilistic approach to all these tuneable params. An example would be that, rather than have the output of the line detection be line vs no line, we could have a line, with an associated probability of it being genuine. This way lines dues to noise, vs lines which are more obvious aren't treated the same, and stages afterwards can take advantage of this nuanced view of the world.

An example that might highlight this is when averaging the lane markings over time by looking at previous frames in the video. If we see many clear lines in previous frames telling us the road is straight, then for this new frame we already have a prior distribution on the lane markings position. If we see weak evidence that agrees with the prior (e.g. we see a faded line that is also going straight), then we are pretty confident that we can keep going straight.

However if we see weak evidence in this frame for a sharp curve (maybe a car is blocking the markings, or a dark patch due to roadworks), we might ignore it and keep going straight.
