# CarND-Path-Planning-Project7
Self-Driving Car Engineer Nanodegree Program
![Uploded Screen_record](https://youtu.be/gDQn16kNIFE)
![Intro](https://lh3.googleusercontent.com/-MdmrM330ziI/XzbQO-i63FI/AAAAAAAAD6U/Yb-bHVqXY7I7tAHJMppwUbwN99eKApvvQCK8BGAsYHg/s0/path_planning.png)
   
### Intoduction
In this project the goal is to safely navigate around a virtual highway with other traffic that is driving +-10 MPH of the 50 MPH speed limit. The car's localization and sensor fusion data is provided, there is also a sparse map list of waypoints around the highway. The car's velocity is as close as possible to the 50 MPH speed limit, which means passing slower traffic when possible. The car tries to avoid hitting other cars at all cost as well as driving inside of the marked road lanes at all times, unless going from one lane to another. The car is able to make one complete loop around the 6946m highway.

### Simulator.
You can download the Term3 Simulator which contains the Path Planning Project from the [releases tab (https://github.com/udacity/self-driving-car-sim/releases).


## Basic Build Instructions

1. Clone this repo.
2. Make a build directory: `mkdir build && cd build`
3. Compile: `cmake .. && make`
4. Run it: `./path_planning`.


## Rubic Points

### Compilation

#### The code compiles correctly.

I used two additional libraries in this project:

- [spline](http://kluge.in-chemnitz.de/opensource/spline/) *(Cubic Spline interpolation implementation)*
- [cppfsm](https://github.com/eglimi/cppfsm) *(Simple, generic, header-only state machine implementation for C++)*


### Valid Trajectories

#### The car is able to drive at least 4.32 miles without incident.

The car was able to drive 10 miles without incidents:

![10miles](https://lh3.googleusercontent.com/-HY14C4LyBOo/XzbQxxV0C8I/AAAAAAAAD6s/4OIefeXGQeoQZYMnJ8dlaE0fcPHfmybcwCK8BGAsYHg/s512/10mile.png)


#### The car drives according to the speed limit.

The car was able to drive 10 miles without a red speed limit message.


#### Max Acceleration and Jerk are not Exceeded.

The car was able to drive 10 miles without red max jerk message.

#### Car does not have collisions.

The car was able to drive 10 miles without a collision.

#### The car stays in its lane, except for the time between changing lanes.

The car doesn't spend more than a 3 second length out side the lane lanes during changing lanes, and every other time the car stays inside one of the 3 lanes on the right hand side of the road.

#### The car is able to change lanes

The car change lanes when the there is a slow car in front of it, and it is safe to change lanes (no other cars around).


### Reflection

This project uses the provided code from the seed project. A lot of the concepts (splines, etc) were taken from the Q&A video that is provided by Udacity. I added additional comments to the code to improve the readability. The functionality is separated into 3 main parts: Prediction, Behaviour Planning and Trajectory Calculation.


#### 1. Prediction

In the code, you can find this part between the lines 302 and 342.

It deals with the telemetry and sensor fusion data and intents to reason about the environment. First, it iterates over the sensor data for each detected car and determines its lane (id). Then it calculates whether this particular car is ahead/left/right of our car with a distance less than 30 meters or not.

#### 2. Behaviour Planning

This part uses a Finite State Machine to determine the behavior. You can find the code here:

- Declaration *(Lines 178-187)*
- Transitions *(Lines 237-255)*
- Trigger *(344-351)*

It decides if the car changes its state to accelerate, decelerate or change lanes. 4 states are defined:

![FSM](https://lh3.googleusercontent.com/-i09EhZaLaj4/X0PE0xOCNdI/AAAAAAAAED0/CqA1YcQWYs4JAKacyg7dUSmLHNOg-8XFwCLcBGAsYHQ/s0/fsm.png)

- **N** = Normal *(Initial State / Accelerate if necessary)*
- **L** = Change To Left Lane
- **R** = Change To Right Lane
- **F** = Follow Vehicle *(Decelerate if necessary)*

In addition, two triggers are used:

- CarAhead *(A car directly in front of us has been detected)*
- Clear *(No car directly in front of us has been detected)*


Based on the prediction of the situation we are in, a trigger will be executed on the state machine. Depending on the current state and the conditions *(defined here: Lines 237-255)*, the car might transition from it's current state to another state. All states can reach all other states directly with the exeption of Left/Right for the reason of simplicity.


#### 3. Trajectory Calculation

In the code, you can find this part between the lines 354 and 469. The ideas and concepts are taken from the Q&A video. 

The trajectories are calculated with the help of splines based on the speed and lane output from the behavior, car coordinates and past path points.

In order to calcuate the splines, the last two points of the previous trajectory *(or the car position if there are no previous trajectory / lines 367-379)* are used in conjunction with three points at a far distance *(30, 60, 90 meters / lines 397-407)* to initialize the spline calculation *(lines 418-437)*. To make the work less complicated to the spline calculation based on those points, the coordinates are transformed (shift and rotation) to local car coordinates *(lines 409-416)*.

To keep a continuous trajectory *(in addition to adding the last two points of the pass trajectory to the spline adjustment)*, the pass trajectory points are copied to the new trajectory. The rest of the points are calculated by evaluating the spline and transforming the output coordinates to not local coordinates.

