# Model Reflection
---
Path Planning Project (Self-Driving Car Engineer Nanodegree Program)

Inorder to create smooth and safe trajectories for the car to move through the highway, a model that uses splines has been used in this project. Since the speed limit is 50 mph, a maximum speed of 49.5 mph is set ([main.cpp](src/main.cpp) line 267). The various parts of the model is explained below.

### Lane Change
The simulator provides the sensor fusion data having the details of the vehicles it identified near our vehicle. This data contains a unique id, the x and y coordinates of the car, the velocity components and the Frenet coordinates, s and d.

Each element of the vector `sensor_fusion` is examined to check whether any of these vehicles are in the same lane as our car. If they are in the same lane and the other vehicle is in front of our car and is within a buffer distance (here, a buffer distance of 30m is used), then a `lane_change` and `too_close` signals are turned on ([main.cpp](src/main.cpp) lines 270-291). These are used to see if the lanes can be changed and also to adjust our speed so that collision is avoided. 

If the lane change signal is turned on, the lane change possibilities are examined. The possibilities are examined by ensuring the following steps. Initially it is ensured that the lane change does not result in moving out of the three lanes. This is checked by making sure that a lane change does not result in `lane` going out of 0-2. If a lane change is possible, then a check is done to identify if any vehicle in the intended lane will be within a buffer distance of 25m. If a vehicle is found to be in that buffer, then the lane change is marked as unsafe. Both the lane changes have variables `good_left_lane_change` and `good_right_lane_change` to identify whether a lane change is safe or not.

Inorder to identify whether to switch lanes to left or right, a simple cost function is included. This cost function identifies if there is a car is present in the next lane at a distance greater than 25m and is moving at a higher speed than the car in front in the current lane, and assigns a high cost if the car in the other lane is moving slower than the car in the current lane. It means there is no point in moving to a lane where a slow moving car is present in front.

The `lane` variable is set based on the safest and less cost lane change ([main.cpp](src/main.cpp) lines 293-366).

## Velocity Adjustment ([main.cpp](src/main.cpp) lines 369-383)
Since high acceleration and jerk results in uncomfort for the passengers, the velocity change had to be taken care of. In this model, a maximum acceleration of ±5 m/s^2^. For this, a change in velocity of 0.1m/s is required (v = 5*0.02 = 0.1m/s). This translates to 0.224mph. The velocity is set using the `ref_vel` variable.

The initial velocity is set as 0.224. If `too_close` variable is set and our car's velocity is greater than the front car's velocity, then the velocity is decreased by 0.224. If the `too_close` variable is set and our velocity is less than the car in the front, a constant velocity is maintained. In other cases, the velocity is increased until the velocity reaches the `max_speed` value.

## Trajectory Generation
A spline is used to create a smooth jerk minimising trajectory. For setting the spline, two points near the current car's position and points at 30, 60 and 90m away are selected. This results in a smooth curve and also minimises the jerk ([main.cpp](src/main.cpp) lines 412-451).

The simulator sends back the points that were not driven through in the previous cycle. If such points are present, then the last two points in those points are selected as the starting points of the spline. This ensures smooth transition of the car. If such points are not present, then the car's current x and y points and a point tangential to this point are selected. Then points that are 30, 60 and 90m away are also selected.

The points are transformed to the car's local coordinate system. This makes the math easy so that points can be sampled from the spline easily to adjust the velocity. These points are used to define the spline ([main.cpp](src/main.cpp) lines 454-466).

Now, the spline is assumed to be a straight line, a target distance is identified and using these assumptions, the number of points to be sampled from the spline are identified([main.cpp](src/main.cpp) lines 482). A total of 50 points is set to be sent to the simulator. The leftover points from the simulator are taken. The remaining points are sampled from the spline and converted to the global coordinate system. These points are sent to the simulator along with the previous points([main.cpp](src/main.cpp) lines 486-502). The previous points are considered to ensure a smooth transition.