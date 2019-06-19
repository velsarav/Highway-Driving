# Highway Driving
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)

[//]: # (Image References)

[image1]: ./docs/Result_15_mins.png "Simulator"
[image2]: ./docs/Result_1.png "Performance"
[image3]: ./docs/Result_7_mins.png "Results"
[image4]: ./docs/Behavior_control.png "Behavior Control"
[image5]: ./docs/Compilation.png "Compilation"
[image6]: ./docs/Simulation_connection.png "Simulator connection"


![alt text][image3]
   
## Project Introduction

Design a path planner that is able to create smooth, safe paths for the car to follow along a 3 lane highway with traffic. Project goal is to safely navigate around a simulated highway with other traffic that is driving +-10 MPH of the 50 MPH speed limit. 

## Simulator
The Udacity provided [simulator](https://github.com/udacity/self-driving-car-sim/releases/tag/T3_v1.2) sends car telemetry information (car's position and velocity) and sensor fusion information about the rest of the car in the highway (car id , velocity and position). The communication between the simulator and the path planner is done using [WebSocket](https://en.wikipedia.org/wiki/WebSocket). 

## Dependencies

* cmake >= 3.5
* make >= 4.1
* gcc/g++ >= 5.4
* [uWebSockets](https://github.com/uWebSockets/uWebSockets)
* For Windows machine [Linux Bash Shell](https://www.howtogeek.com/249966/how-to-install-and-use-the-linux-bash-shell-on-windows-10/)
* Udacity simulator
* Udacity [seed project](https://github.com/udacity/CarND-Path-Planning-Project)
* [Spline](http://kluge.in-chemnitz.de/opensource/spline/) for creating smooth trajectories

## Rubric Points

The [rubric](https://review.udacity.com/#!/rubrics/1971/view) points were individually addressed in the [implementation](https://github.com/velsarav/Highway-Driving)

### Compilation and connection to simulator
The code compiles correctly.

![alt text][image5]

The path planner is running and listening on port 4567, once the simulator is started it is getting connected message.

![alt text][image6]

The car started the trajectory of the path

![alt text][image2]

After 15 mins still the car is driving without incidents

![alt text][image1]

### Reflection

#### Behavior control

![alt text][image4]

Behavior control of the car is based on the path planning, which consists of prediction,behavior planning and trajectory generation.

#### Prediction
#### Behavior planning
#### Trajectory generation

## Reference
* [Path planning project](https://github.com/darienmt/CarND-Path-Planning-Project-P1)

* [Path planning](https://medium.com/intro-to-artificial-intelligence/path-planning-project-udacitys-self-driving-car-nanodegree-be1f531cc4f7)



