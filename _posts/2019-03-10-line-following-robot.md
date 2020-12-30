---
layout: post
title: "Raspberry Pi Line Following Robot Tutorial"
date: 2019-03-10 20:11:14 -0500
---

<img class="img-fluid w-100" src="/images/line-robot/robot_moving.gif">

## What do you get out of this project?
 1. Experience assembling and soldering and assembling Raspberry Pi robot
 2. Knowledge of basic PID control, and programming a PID controller from scratch
 3. An introduction to OpenCV image processing, and its application to a real-world problem.
 4. Something you can show off!

## Estimated completion time
2-3 nights, if you copy-and-paste the code, or 1-2 weeks if you do it from scratch, with a bit of guidance from here.

## What does this project consist of?
1. Assembly
2. OpenCV image processing
3. PID control
4. Integration of parts 2 and 3 to implement motion control

## 1. Assembly
I purchased my parts according to a simple [Raspberry Pi robot kit](https://learn.adafruit.com/simple-raspberry-pi-robot/overview) on Adafruit, and followed the assembly instructions of this guide. Additionally, you should purchase a [Pi camera](https://www.adafruit.com/product/3099). Including the camera and the Pi, everything in the kit costs around $150. If this seems like an unreasonable price, consider that the Raspberry Pi, camera, chassis, motors, etc., are all extremely versatile, and need not (and should not!) be used for just this project. Consider this to be an investment.

If you don't want to spend too much time assembling the robot, and want to get into the coding part of the project like I did, this kit is a great option.

### Attaching the camera

Since Adafruit's assembly guide is for a generic, camera-less robot, we also need to attach a camera. The camera must be attached at a downwards angle, as the robot needs to capture what is immediately in front of it. One very hacky (weird?) trick I did was inserting a toothpick below the camera, and wrapping it in electrical tape until I was satisfied with the camera angle. If you care more about your robot's dignity, perhaps you can find a more elegant solution.

|                         The finished robot                         |                   Close-up of the camera fixture                    |
| :----------------------------------------------------------------: | :-----------------------------------------------------------------: |
| <img class="img-fluid" src="/images/line-robot/robot_closeup.png"> | <img class="img-fluid" src="/images/line-robot/camera_closeup.png"> |

## 2. OpenCV image processing

After we've assembled our robot, the next step in the process is to determine the robot's error at each frame, so we'll know how much to adjust. But, even before we do that, we should determine what error we're looking for. Simply put, **the error of the robot is its horizontal (x) distance from the center of the line.** If the robot is turned excessively to the left, for instance, the image it captures will show that it's off-center to the left. The challenge is in parsing out only the line in the image, and determining its center.


|                                    <img src="/images/line-robot/error_diagram.png" class="img-fluid center">                                    |
| :---------------------------------------------------------------------------------------------------------------------------------------------: |
| Example: The robot is facing to the left, so the error we're trying to calculate is represented by the red line. <br> (Not to scale, obviously) |


The code below shows how we transform the image the robot takes at each frame into a contour (an outline specified by an array of (x, y) coordinates) of the line.

The process consists of:

1. Converting the image to grayscale.
2. Applying a Gaussian blur to decrease noise.
3. Thresholding the image, converting the irrelevant parts to black.
    and the relevant parts white.
4. Finding all contours in the thresholded image.
5. Returning the largest contour (by area), which is hopefully the line.

``` python
# Gets the largest contour in the image, by area,
# as a list of (x,y) coordinate pairs.
def getLargestContour(input, threshold, otsu=True):
    # Convert the image to grayscale, 
    gray = cv2.cvtColor(input, cv2.COLOR_BGR2GRAY)
    # Apply Gaussian blur to decrease noise.
    gray = cv2.GaussianBlur(gray, (3, 3), 0)
    '''
    Threshold the image, converting the irrelevant parts to 0 (black)
    and the relevant parts (the line) to 255 (white). Notice the use of 
    cv2.THRESH_BINARY_INV. This tells OpenCV to convert things BELOW 
    the threshold (as opposed to above) to white. This is important 
    since our line is black. Otsu thresholding is an adaptive thresholding 
    algorithm, which I found was unnecessary for this project.
    '''
    threshType = cv2.THRESH_BINARY_INV | cv2.THRESH_OTSU if (otsu) \
            else cv2.THRESH_BINARY_INV
    _,thresh = cv2.threshold(gray, threshold, 255, \
            threshType)
    thresh = close(thresh)
    # Find all contours in the thresholded image.
    _,contours,_ = cv2.findContours(thresh, cv2.RETR_TREE, \
            cv2.CHAIN_APPROX_SIMPLE)
    # Return the largest contour (by area).
    if (len(contours) > 0):
        return max(contours, key = cv2.contourArea)
    else:
        return None

```

## 3. PID control
PID control is a mechanism that we use to control our robot's turning and speed. Keep in my mind that what I discuss below is just basic PID control, since that's all we need for this project. If you want to learn more, this is a huge field, with lots of resources available online.

I learned about PID control from Udacity's free [Artificial Intelligence for Robotics course](https://www.udacity.com/course/artificial-intelligence-for-robotics--cs373). PID control is in the later part of the course, and even if you don't want to go through all of it, I recommend watching the videos about PID control. They're short, and very well put together. If you *still* don't want to watch the videos 😒, I'll explain the basics below.

### Why do we need PID control? 

Let's take a step back. From part 2, we've figured out the error of the robot at each frame, so we know how much the robot needs to adjust. If we simply turn the robot in the opposite direction, proportionally to its error, our robot's movement will be extremely choppy, and it may even steer off path. 

This becomes especially significant when the robot needs to navigate non-straight lines. If we only turn and slow down proportionally to the error, when the robot encounters a turn, it won't slow down in time and its inertia will propel it off path.

So, we need the robot to move smoothly and be able to handle turns. This is where PID control comes in.

### What is PID control? 

According to [Wikipedia](https://en.wikipedia.org/wiki/PID_controller),

 > A **proportional–integral–derivative controller (PID controller** or **three-term controller**) is a control loop feedback mechanism widely used in industrial control systems and a variety of other applications requiring continuously modulated control.
 
 > A PID controller continuously calculates an error value *e(t)* as the difference between a desired setpoint (SP) and a measured process variable (PV) and applies a correction based on proportional, integral, and derivative terms (denoted P, I, and D respectively), hence the name.

We know our error and our desired setpoint (the center of the line), and we need continuously modulated control. The "modulated" part means that we need to vary the amplitude of the correction in response to the variation of the error. A PID controller sounds exactly like what we want!

Now that we know what a PID controller is on a high level, let's understand its individual components.

The **P** stands for **proportional**, and is pretty intuitive to understand. We have an error, and we should take its magnitude into account.

The **I** stands for **integral**. This term is simply the sum of *n* previous error values, and accounts for the time the error has persisted. If we've had an error for a longer time, this means we should probably apply more correction.


The **D** stands for **derivative**. This term is simply equal to ```(current_error - last_error)```. It accounts for future events, like an upcoming turn. If we see that our error has rapidly increased, as will be observed in a turn, the D term will apply some extra correction to avoid the robot veering off path due to momentum.

|            P             |                     I                     |                           D                           |
| :----------------------: | :---------------------------------------: | :---------------------------------------------------: |
|       Proportional       |                 Integral                  |                      Derivative                       |
| Proportional correction. | Corrects based on sum of last *n* errors. | Corrects based on ```(current_error - last_error)```. |

### What are PID *gains*?

A PID controller calculates a correction values, with contribution from three terms. However, we need to consider how much each term should actually contribute to the correction. A gain is simply *a weight* we give to each component of the controller. Usually P has the largest gain, which should be intuitive to understand. If you look closely at the code below, you'll notice an additional **s_gain** variable at the bottom. This is a gain I use specifically to correct the robot's speed, and is an optimization I picked up from [this paper](https://www.sciencedirect.com/science/article/pii/S1877050915038302).

Below is the code for the main method of the PID controller with comments to explain each part. As you can see, it's quite short and simple.

``` python
# x and y dist are distances from some reference point.
def pid(self, x_dist, yDist, dt):
    # yDist doesn't get used.
    # cte = cross track error.
    cte = x_dist
    # Calculate derivative.
    d_cte = (cte - self.prev_error) / dt
    self.prev_error = cte
    if (len(self.prev_vals) == self.buffer_size):
        # buffer_size is how many past observations we should consider in I.
        # Get rid of LRU error.
        lru_error = self.prev_vals.pop()
        self.error_sum -= lru_error
        self.abs_error_sum -= abs(lru_error)
    # Calculate new I.
    self.error_sum += cte
    self.abs_error_sum += abs(cte)
    self.prev_vals.append(cte)
    # Apply gains and calculate final correction.
    return - self.p_gain * cte - self.d_gain * d_cte \
            - self.i_gain * self.error_sum \
            - self.s_gain * self.abs_error_sum
```

## 4. Putting it all together!

The code below is the main video capture loop. In it, we:

1. Parse out the contour from the raw frame.
2. Get center x and y points from the contour.
3. Draw the contour, and some other things on the image to help with visualization.
3. Determine proper steer and speed corrections from the PID controllers.
4. Actually move the robot.
5. Clear the buffer in preparation for the next frame.

*Note*: Even though we calculate the y distance, it's actually not used anywhere.

If you check out the full source code on [my GitHub](https://github.com/mpokryva/LineTrackerPy/blob/master/line.py), you will notice I use a relatively low framerate of 15 fps. This is because the Pi cannot process the frames quicker than this. In fact, 15 is probably too high, and a framerate of 10 fps or less would suffice as well.

``` python
for frame in camera.capture_continuous(rawCapture, format="bgr", \
        use_video_port=True):
    image = frame.array
    # We crop the image vertically, to eliminate irrelevant details.
    image = image[camera.resolution[1] - 80: camera.resolution[1]]
    # Any pixel with a value above this threshold will be set to 0 (black).
    threshold = 50
     # Don't do Otsu (I found it performs better without).
    contour = getLargestContour(image, threshold, False)
    '''
    Optimization for handling slight overshoot of turns, when line
    briefly goes out of view.
    '''
    if (contour is None):
        if (current_try > max_tries):
            print("No lines found... Stopping.")
            break
        print("No lines found... Try #" + str(current_try))
        current_try += 1
        robot.smooth_turn(int(prev_speed), int(prev_steer))
        rawCapture.truncate(0)
        continue
    moment = cv2.moments(contour)
    if (moment["m00"] != 0):
         # Get the center x.
        cx = int(moment["m10"]/moment["m00"])
         # Get the center y.
        cy = int(moment["m01"]/moment["m00"])
    else:
        cx, cy = 0, 0
    
    width = np.size(image, 1)
    height = np.size(image, 0)
    bottomCenter = (width/2, height)
    xDist = (cx - bottomCenter[0])
    yDist = abs(cy - bottomCenter[1])
    # Calculate angle.
    #angle = np.arctan2(xDist, yDist) * (180 / np.pi)

    # Draw stuff
    cv2.circle(image, (cx, cy), 4, (255, 255, 0), 2)
    drawContour(image, contour, (0, 0, 255), 4)
    cv2.line(image, (cx, cy), bottomCenter, (0, 255, 0), 4)
    
    # Get steer and speed corrections from PID controllers.
    steer = turnControl.pid(xDist, yDist, 1/camera.framerate)
    speedDiff = speedControl.pid(xDist, yDist, 1/camera.framerate)

    # Display the current frame.
    cv2.imshow("Frame", image)
    
    speed = maxSpeed + speedDiff
    prev_speed = speed
    prev_steer = steer
    errorThresh = 5.0
    if (move and abs(xDist) > errorThresh):
        absSteer = abs(steer)
        robot.smooth_turn(int(speed), int(steer))
    key = cv2.waitKey(1) & 0xFF

    # Clear the stream for the next frame.
    rawCapture.truncate(0)
    '''
    If "q" is pressed, stop. 
    If "m" is pressed, toggle robot movement.
    This is useful for debugging, since you can pick the robot up,
    move it somewhere else, and then press "m" again.
    '''
    if key == ord("q"):
        break
    elif key == ord("m"):
        move = not move
        if not move:
            robot.stop()
    i += 1
```

## The end result

{% include youtube_video.html id="QYSfXf6VR3A" %}      
#### Full implementation can be found [here](https://github.com/mpokryva/LineTrackerPy).
