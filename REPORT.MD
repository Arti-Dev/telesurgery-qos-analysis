# Undergraduate Research Trial Task Submission
## 1. Summary
## 2. Environment and Setup
I'm running this project on a Linux laptop with an Nvidia GPU on Ubuntu 24.04. 
I was able to set up all instructions per usual, but I had to install a few extra packages.
Additionally, the touch_haptic.py file was altered to prevent it looking for a haptic device that I don't have.

## 3. Part 1: Simulator Setup
When the replay data is imported it appears to be desynced, but I understand that is part of the assignment.

Video: https://www.youtube.com/watch?v=qr9gof-MeRk
## 4. Part 2: Data Visualization and Trajectory Comparison
The replay script is sending packets at around 500Hz, which is around a packet every 2ms. Packets contain commands for the PSMs, not full simulator data. Running the same replay file results in different results each time, with usually only one or no triangles being transferred to the other side.

![img.png](report-files/img.png)

I notice that the robot actions do somewhat match what the console is playing in the X, Z, and Yaw directions, but not the Y direction. 

On the kinematics graphs there is a phase delay between the robot and the console caused by the delay between me starting the simulation and starting the replay. This isn't a problem.

![img.png](report-files/img2.png)

Looking at the delay graph a lot of packets are being processed way beyond 2ms (max 13ms). I'm assuming the simulation doesn't validate packet order, so if a packet isn't processed or doesn't arrive in time an out-of-order packet might be processed first.
However, it could also be the case that this simulator is not frame-independent and thus lag spikes will cause physics differences. This is much more likely.

Either way, this would cause a simulation desync because the replay only stores input data, not the actual state of the simulation. So if the grabber were to collide with a peg when it did not at the time of recording, that will offset the grabber's position for the rest of the replay.

One way I thought of to try to replay the data properly was to slow down the rate that packets are sent, so I reduced the frequency by 10x to 50Hz. By doing this, the simulation was slightly more consistent, as I had one time where the first two pegs were successfully placed on the other side, but it still had problems.

A more permanent fix would involve modification of the simulation environment in two ways. First, it needs to validate packet order to prevent desyncing. Second, it should store packets in a buffer and replay at a guaranteed frequency. If for some reason the buffer runs out, the physics simulation should pause.
Lastly the simulation should be framerate-independent.

## 5. Part 3: Simulation Development

### (Trying to) export frames

The first thing I did was identify where the main simulation loop was. 
After skimming a little I noticed the function `getCameraImage()` from PyBullet that seemed to return exactly what was asked for.
I noticed commented out code that would convert this data into an image, so I uncommented it and used PIL to convert it.

![img.png](report-files/static.png)

The RGB frame looked like this at first, which did not look correct. I turned to Gemini to try to come up with a fix, which
insisted that it was a NumPy formatting error - it was not.

The PyBullet documentation gave me a good idea of what `getCameraImage()` returns, but not much else.
I was also confused as to why in its other usages, the results of `getCameraImage()` were unused.

Once I found the [pybullet-rendering](https://github.com/ikalevatykh/pybullet_rendering) documentation it made more sense.
This allows external renderers to intercept `getCameraImage()` and return different rendering results, which is done
by inheriting `BaseRenderer` and implementing the `render_frame` method. By altering the parameter `frame` passed into that method,
that changes what PyBullet's `getCameraImage()` returns.

I traced SurRol's external renderer to `surrol/gui/pybullet_renderer` and studied the code. The `frame` parameter was untouched
which would explain why I was not seeing anything. I uncommented some code that would take a screenshot of the rendering and
put that in `frame.color_img`, which successfully allowed me to extract that in `multiple_scenes_console_replay.py`.
However, that is just a screenshot and not the RGB frames.

I see a bunch of commented out code that would potentially allow me to properly extract the requested data.
However, I have had difficulty understanding it, and thus I haven't gotten there yet.

I have tried looking at the panda3d-based renderer provided by pybullet-rendering (`P3dRenderer`), but that seems to diverge from what SurRol does.
I have also looked at the `getCameraImageTest.py` script in the PyBullet examples.

### Writing to disk

I have split output into three folders, one for RGB, one for depth, and one for segmentation masks.

Every nth frame a new thread is spawned to read the output of `getCameraImage()`, convert the NumPy arrays to images, and save them to disk.

Multithreading allows the simulation to continue running without blocking the main thread with file writes.

## 6. Code Changes
## 7. Usage Instructions
## 8. Results
## 9. Performance Profiling
## 10. Tests
## 11. Limitations and Future Improvements
## 12. GenAI Use Disclosure