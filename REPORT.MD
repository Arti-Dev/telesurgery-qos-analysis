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
## 6. Code Changes
## 7. Usage Instructions
## 8. Results
## 9. Performance Profiling
## 10. Tests
## 11. Limitations and Future Improvements
## 12. GenAI Use Disclosure