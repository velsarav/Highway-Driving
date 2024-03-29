# Highway Driving
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)

[//]: # (Image References)

[image1]: ./docs/Result_15_mins.png "Simulator"
[image2]: ./docs/Result_1.png "Performance"
[image3]: ./docs/Result_7_mins.png "Results"
[image4]: ./docs/Behavior_control.png "Behavior Control"
[image5]: ./docs/Compilation.png "Compilation"
[image6]: ./docs/Simulation_connection.png "Simulator connection"
[image7]: ./docs/Behavioral_overview.png "Behavioral planning"


![alt text][image3]

---
   
## Project Introduction

Design a path planner that is able to create smooth, safe paths for the car to follow along a 3 lane highway with traffic. The Project goal is to safely navigate around a simulated highway with other traffic that is driving +-10 MPH of the 50 MPH speed limit. 

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

---

## Rubric Points

The [rubric](https://review.udacity.com/#!/rubrics/1971/view) points were individually addressed in the [implementation](https://github.com/velsarav/Highway-Driving)

### Compilation and connection to the simulator
The code compiles correctly.

![alt text][image5]

The path planner is running and listening on port 4567, once the simulator is started it is getting connected message.

![alt text][image6]

The car started the trajectory of the path

![alt text][image2]

After 15 mins still, the car is driving without incidents

![alt text][image1]

---

## Reflection

#### Behavior control

![alt text][image4]

Behavior control of the car is based on path planning, which consists of prediction, behavior planning and trajectory generation.

#### Prediction
The prediction component estimates what actions other objects might take in the future. For example, if another vehicle were identified, the prediction component would estimate its future trajectory. In this project prediction module predicts the following
* Car ahead is too close to our car
* Car in left and right is too close to our car.
* Car in left and right making any lane change not safe.

By calculating the lane each car is and the position it will be at the end of the last plan trajectory we will get the answers to the above questions. 
In the simulator, we have only three lanes and each lane has 4-meter width

```
    if(d > 0 && d < 4) {
        check_car_lane = 0;
    } else if(d > 4 && d < 8) {
        check_car_lane = 1;
    } else if(d > 8 and d < 12) {
        check_car_lane = 2;
    }

```

Estimate the car's position after executing the previous trajectory. A car is considered "dangerous" when its distance to our car is less than 30 meters.

```
    if(check_car_lane == lane) {
        // Car in our lane.
        car_ahead |= check_car_s > car_s && (check_car_s - car_s) < 30;					
    } else if((check_car_lane - lane) == -1) {
        // Car left
        car_left |= (car_s+30) > check_car_s  && (car_s-30) < check_car_s;
    } else if((check_car_lane - lane) == 1) {
        // Car right
        car_right |= (car_s+30) > check_car_s  && (car_s-30) < check_car_s;
    }

```

#### Behavior planning

Following the behavior of the car needs to be planned.
* Do we need to change the lane if there is a car in front of us?
* Do we need to speed up or slow down?

![alt text][image7]

Behavior planner takes the input of maps, route, and predictions about what other vehicles are likely to do and suggests the trajectory module create trajectory based on the suggestion from the behavior planner. Prediction set three flags(car_ahead, car_left, and car_right) according to the sensor fusion data. If the car is not ahead and velocity is not the same as max_accel, then it will increase the speed by small difference during each check.

```
if(car_ahead) {
    if(!car_left && lane > 0) {
        // if there is no car left and there is a left lane.
        lane--; // Change lane left.
    } else if(!car_right && lane !=2) {
        // if there is no car right and there is a right lane.
        lane++;// Change lane right.
    } else if(!car_left && lane !=2) {
        lane++;
    }else {
        ref_vel -= speed_diff;
    }
} else if(ref_vel < max_accel){
    ref_vel += speed_diff;
}
```


#### Trajectory generation
Calculation of the trajectory is based on the speed and lane output from the behavior, car coordinates and past path points.

Check any previous points is almost empty, then we use the current car's point to find the previous point and add them to the list, else just add the previous two points. Also, push back last know the previous point to reference x and y

```
// If previous states are almost empty, use the car as a startting point
if ( prev_size < 2 ) {

    //Use two points that make path tangent to the car
    double prev_car_x = car_x - cos(car_yaw);
    double prev_car_y = car_y - sin(car_yaw);

    ptsx.push_back(prev_car_x);
    ptsx.push_back(car_x);

    ptsy.push_back(prev_car_y);
    ptsy.push_back(car_y);

    } else {
    //Redefine the reference point to previous point
    ref_x = previous_path_x[prev_size - 1];
    ref_y = previous_path_y[prev_size - 1];

    double ref_x_prev = previous_path_x[prev_size - 2];
    double ref_y_prev = previous_path_y[prev_size - 2];
    ref_yaw = atan2(ref_y-ref_y_prev, ref_x-ref_x_prev);

    ptsx.push_back(ref_x_prev);
    ptsx.push_back(ref_x);

    ptsy.push_back(ref_y_prev);
    ptsy.push_back(ref_y);
}
```

Now we need to add 3 future points to ptsx, psy vecotrs. As car_s is frenet and we need to convert to the global x,y coordinates using getXY function. In total ptsx, ptsy has got 5 points in total each. Line number [226 to 237](https://github.com/velsarav/Highway-Driving/blob/master/src/main.cpp#L226)

For trajectory generation, we are using spline instead of polynomial trajectory generation. One of the reasons is it is simple to use and requires no dependencies.

Then we add all previous points to next_x_vals and next_y_vals as it going to be the final control values pass it to the simulator and it will help to get a smooth transition to the new points that we calculate later. Line number [252 to 259](https://github.com/velsarav/Highway-Driving/blob/master/src/main.cpp#L252)

Now we need to find the all spline points till the horizon(say 30m) value so that spacing the way that other car can travel at the desired speed. Remember the speed of the car is depending on the spacing between the points. If we know x point(ie 30m in this case), spline will be able to get us corresponding spline y points. Line number [262 to 264](https://github.com/velsarav/Highway-Driving/blob/master/src/main.cpp#L262)

Our number of points has to be calculated is 50. We need to future points and previous points for the trajectory calculation. next_x_vals and next_y_vals calculation between line number[268 to 288](https://github.com/velsarav/Highway-Driving/blob/master/src/main.cpp#L268) will have all 50 points which consist of 3 future points and 47 previous points.


---

## Reference
* [Path planning project](https://github.com/darienmt/CarND-Path-Planning-Project-P1)

* [Path planning](https://medium.com/intro-to-artificial-intelligence/path-planning-project-udacitys-self-driving-car-nanodegree-be1f531cc4f7)



