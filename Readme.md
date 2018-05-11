# Trajectory Generation

## Creating a smooth trajectory

We start off by generating 5 points in xy coordinates to fit the spline curves in. Two of these points come from the previous trajectory points or if the vehicle is just starting off then the initial starting points are used. The next three points are calculated by incrementing the car's s by 30 and setting d to the center of the current lane while passing these values to getXY function.
Once we have the 5 points we transform the points from the global system to car's local coordinates with car at zero and its yaw at 0 degree. This makes all the subsequent math much easier.
We then set the spline to these 5 points. The unused points from previous trajectory is pushed to the current trajectory.
since we always need 50 points in the trajectory we generate the missing points (50 - number of unused points from previous trajectory) by using the spline curve. The x value is calculated in such a way as to maintain the reference velocity. This x value is passed to the spline to get the corresponding y value.
these points are transformed to the global coordinates before being added to the current trajectory.

This makes the trajectory that is smooth and can maintain the reference velocity.

## Limiting jerk while accelerating/decelerating

Since the simulator is using a perfect controller we need to take care of acceleration and jerk constraints by placing the trajectory points at proper spacing.
while generating the trajectory, as described above, the x increment to the points are based on the reference velocity. This is why we initialize the reference velocity at 0 MPH and increment it with a constant acceleration.
However, this increment in reference velocity is done only if the current lane does not have a vehicle in front of it which is closer than the safe distance specified (30 m).
Similarly, when a vehicle is detected as too close the reference velocity is decremented with the same constant acceleration.

## Collision avoidance and Lane switching

When the vehicle detects another vehicle in the same lane which is closer than the safe distance then an actions must be performed.
These actions a based on inputs from sensor fusion which gives the current positions and velocity of other vehicles around the car.

Before we take an action we need to calculate the future state of other cars and compare that to the future state of our car. 
This comparison is used to answer these questions and the result is stored in Boolean variables:

*  too_close - Is there a car ahead in the same lane which is too close to us?

*   car_left - Is there a car (ahead or behind us) that would collide if we switch lane to the left?

*   car_right - Is there a car (ahead or behind us) that would collide if we switch lane to the right?
			
*   cars_right -  Is there a car (ahead or behind us) that would collide if we switch TWO lanes to the right?

*   cars_left - Is there a car (ahead or behind us) that would collide if we switch TWO lanes to the left?

*  cars_halfway_right - Is there a car (ahead or behind us in the middle lane) that would collide with us halfway between if we switch TWO lanes to the right?

*  cars_halfway_left - Is there a car (ahead or behind us in the middle lane) that would collide with us halfway between if we switch TWO lanes to the left ?


These Boolean variables are used to decide which action is taken to change the trajectory of the car. These actions are only required if too_close is true.
The actions are (priority wise):

* change lane to the right -  If the car is not already in the right lane and car_right is false we shift to the right lane.  
* change lane to the left - If the car is not already in the left lane and car_left is false we shift to the left lane.
* change two lanes to right - If the car is in the leftmost lane and cars_right and cars_halfway_right are both false we shift two lanes to the right.  
* change two lanes to left - If the car is in the rightmost lane and cars_left and cars_halfway_left are both false we shift two lanes to the left.
* slow down - if none of the other actions are available then decelerate at the reference acceleration value.


