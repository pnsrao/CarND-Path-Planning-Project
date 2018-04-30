# Trajectory Planning

As a starting point, I planned the trajectory roughly following the guidelines in the project walkthrough video. 
As mentioned in the video,th ekey idea is to use a spline function to create a smooth trajectory between the previous
path of the car and the intended final position. This minimizes aspects such as acceleration and jerk.

## Creation of the spline
The spline is created by using the last two unconsumed points of the previous path and three target points from the 
future map waypoints. While the video recommended the target points to be 30, 60 and 90m down the road, I found that 
this resulted in jerk/acceleration violations during lane changes at higher speeds (lane changes are described later).
I got around this by making the target points a function of the current speed (60, 75 and 90m). In essence, while
traveling at higher speeds, you aim for points further down the road.
Note that the lane number is a variable when calculating the spline.

## Computing the next set of path points for the simulator
Once the spline is computed, the next set of points for the simulator is computed using the rest of the unconsumed points 
from the previous path and future points from the "spline trajectory". Assuming the vehicle is moving at a certain reference 
velocity, we compute the distance traveled in one time step. We then add x and y points from the spline to the future
path. These points are computed such that the distance between successive points is consistent with the expected
reference velocity. This computation maintains the vehicle speed at the targeted value.

In the above two steps, the lane number and reference velocity are variables that feed into the spline creation and 
calculation of the future path points. Below, I discuss how these are determined.

## Maintaining a reference velocity
This aspect is critical in minimizing acceleration and jerk. The principles are
* Wherever possible, gradually increase the speed to 50 mph. Ensure that there are no sudden changes in speed.
* If you see a car ahead that is too close, start to slow down. 
These two aspects are implemented in the code as simple modifications on reference velocity. These modifications are pretty 
effective

## Lane changes
Lane changes are only triggered when a car ahead is detected as being too close to you and you start to slow down. 
During the lane change process, we first check if the left lane is clear. If it is clear, we initiate the lane change. 
If it is not clear, we check if the right lane is clear and initiate a right lane change. If both lanes are not clear, 
we maintain the current lane while decreasing the speed.

### Lane clearance check
A lane clearance check involves a rear clearance check if cars are behind you in the target lane, and a front clearance 
check if cars are ahead of you in the target lane. The default distance for the rear clearance check is 10m. However if the 
car behind you is speeding, then an additional clearance proportional to the difference in speeds is added. Currently this 
additional clearance is calculated based on 5 time steps which constitutes an aggressive lane change. 

Similarly, a front clearance check involves a default distance of 30m and an additional distance if the car ahead is slower 
than our own car.

### Executing a lane change
Since the lane number is a variable that feeds into spline calculation, executing a lane change is quite simple. We just
change the target lane number and the spline calculation automatically computes the path that leads to a lane change.

## Conclusion
With the above logic implemented, the car 
* drives about 4.3 miles on the road without incidents, 
* maintains an average speed of clsoe to 50mph wherever possible, 
* and executes left and right lane changes to overtake a slow moving car wherever lanes are clear. 
