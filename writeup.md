**MPC Controller** 

**MPC Controller Project**

The goals / steps of this project are the following:
* Implement the MPC
* Keep the car in the simulator on track using the cte and the MPC with an appropriate set of parameters
* Control the car speed via MPC
* Handle the delay
* Summarize the results with a written report



---
### The MPC process

The MPC, or model predictive control, tries to optimize a process making predictions based on some assumptions on the model, and reducing the cost function associated to it. In out case, given a path, it tries to follow it minimizing the angle and cross track errors.

#### Input and output

The MPC takes the following variables as input:

* v      the speed at which the object is currently moving
* cte    the cross track error (or an approximation thereof)
* epsi   the error on the angle (psi)
* coeffs the coeffiecients of the polynomial fitted to the points given by the simulator

and outputs throttle and steering value.

#### Preprocessing and fitting polynomial

While v is given by the simulator the other variables are computed:

First the given points are rotated and traslated to be relative to the car's position (main.cpp:99) then

* coeffs are fit on the transform of the given points with polyfit(ptsx, ptsy, 3) (main.cpp:113)
* cte is an approximation of cross-track error based on the coefficients with polyeval(coeffs, 0) (main.cpp:115)
* epsi is computed simplifying the formula for the angular error assuming that the car is at the center of coordintes and angle is 0, which we made sure before (main.cpp:117)


#### Latency

The latency was dealt with assuming that the car moves linearly and at constant speed, by calculating the new position, making no assumptions on acceleration or yaw rate (main.cpp:96)

#### N and dt

The number of steps of look ahead and delta t between them were

* dt 0.1  a dt of a tenth of second is short enough that the car won't miss details of the curve it should follow, and has time to adjust smoothly, but long enough to avoid a large computational cost.
* N 20  which allows to predict two seconds in the future, allowing to handle steep curves, without adding too much computational cost.

### Model

The model is otimized on a cost function that is the sum of the cost on the predicted points.

#### Cost function

The cost function (MPC.cpp:52-68) is given by the sum of

* Difference between cte and reference cte (set to 0, a non-zero reference cte means we are telling the car to stay off the center) at the power of 6, in my case, since a higher power allows the algorithm to disregard smaller cte errors, possibly following a smoother and safer path.
* Difference between angular error and reference epsi (set to 0, non 0 would cause the car to keep trying to oversteer on one direction, contrasted by the cte error) squared.
* Difference between speed and reference speed, set by the user, squared. A speed higher than 0 is necessary to avoid early stopping (the car does not move if that minimizes the cost funcntion, eg if it's at the center of the track and parallel to it). It is also possible to assign a cost to the distance to a given point, to force the car to move without a set speed.  I set the parameters for speed low so if the car is too fast to handle a steep curve it brakes first.
* Cte variation, squared. It penalizes large cte variations, like when the car overcorrects. It is very similar to the PID derivative component, although it is summed over many steps, and integrates the job of the epsi angular error (which tries to keep the trajectory parallel to the track).
* Acceleration variation, squared. It penalizes fast accelerations and decelerations. On top of being uncomfortable for a potential human passenger, fast acceleration may cause the MPC to behave erratically, as the temporal horizon (given by dinstance run over dt) changes rapidly.
* Cte variation difference, squared. It penalizes fast correction of the cte variation, thus forces the car to follow an even smoother path. For example the car could oscillate a bit around the ideal path keeping the cte and cte variation small, but it would introduce many cte variation difference errors which would add up.
* Acceleration variation difference, squared. Penalizes fast correction of the acceleration variation, like above.

#### Updating the prediction

Finally the predicted position speed and relative errors and steering angle are updated (MPC.cpp:70-104).

The first acceleration and steering angle of prediction are set up (MPC.cpp:220-221) to be used as output (main.cpp:168-169)
