**MPC Controller** 

**MPC Controller Project**

The goals / steps of this project are the following:
* Implement the MPC
* Keep the car in the simulator on track using the cte and the MPC with an appropriate set of parameters
* Control the car speed via MPC
* Handle the delay
* Summarize the results with a written report



---
### The MPC

The MPC, or model predictive control, tries to optimize a process making predictions based on some assumptions on the model, and reducing the cost function associated to it. In out case, given a path, it tries to follow it minimizing the angle and cross track errors.

#### Input and output

The MPC takes the following variables as input:

* v      the speed at which the object is currently moving
* cte    the cross track error (or an approximation thereof)
* epsi   the error on the angle (psi)
* coeffs the coeffiecients of the polynomial fitted to the points given by the simulator


and outputs throttle and steering value.

While v is given by the simulator the other variables are computed:

First the given points are rotated and traslated to be relative to the car's position (MPC.cpp:99) then

* coeffs are fit on the transform of the given points (MPC.cpp:113)
* cte is an approximation of cross-track error based on the coefficients (MPC.cpp:115)
* epsi is computed simplifying the formula for the angular error assuming that the car is at the center of coordintes and angle is 0, which we made sure before (MPC.cpp:117)


#### Latency

The latency was dealt with assuming that the car moves linearly and at constant speed, by calculating the new position, making no assumptions on acceleration or yaw rate (MPC.cpp:96)

#### N and dt

The number of steps of look ahead and delta t between them were

* dt 0.1  a dt of a tenth of second is short enough that the car won't miss details of the curve it should follow, and has time to adjust smoothly, but long enough to avoid a large computational cost.
* N 20  which allows to predict two seconds in the future, allowing to handle steep curves, without adding too much computational cost.

