# CarND-Controls-MPC
Self-Driving Car Engineer Nanodegree Program

## Introduction
In this project, I am implementing a Model Predictive Control to drive a car around the track. 

Model predictive controllers rely on dynamic models of the process, most often linear empirical models obtained by system identification. The main advantage of MPC is the fact that it allows the current timeslot to be optimized, while keeping future timeslots in account. This is achieved by optimizing a finite time-horizon, but only implementing the current timeslot and then optimizing again, repeatedly[...](https://en.wikipedia.org/wiki/Model_predictive_control)

## The Model
The model used in this project is a kinematic bicycle model which is a non linear model. Below are the models variables in use.

* x: coordinate
* y: coordinate
* psi: heading angle
* v: speed
* cte: cross to center error
* epsi: heading angle error

Below are the 2 actuators 
* delta: steering
* a: throttle

The update euqation of the model is as below.
 ```
 x_[t+1] = x[t] + v[t] * cos(psi[t]) * dt
 y_[t+1] = y[t] + v[t] * sin(psi[t]) * dt
 psi_[t+1] = psi[t] + v[t] / Lf * delta[t] * dt
 v_[t+1] = v[t] + a[t] * dt
 cte[t+1] = f(x[t]) - y[t] + v[t] * sin(epsi[t]) * dt
 epsi[t+1] = psi[t] - psides[t] + v[t] * delta[t] / Lf * dt
 ```

## Timestep Length and Elapsed Duration (N & dt)

Here I choose 25 for N and 0.05 for dt. These values gave me the best and smooth vehicle control i the simulator. I tested these values for the vehicle speed from 30mph to about 80mph. You can change the speed limit using `ref_v` variable in MPC.cpp.

## Polynomial Fitting and MPC Preprocessing

The waypoints are used to polyfit the desired trajectory with 3rd polynomial fitting.
In order to render the waypoints and polyfitted trajectory in simulator. All the points 
are transformated from global coordination system to vehicle coordination system.
This involves coordinate translation and rotation. ( main.cpp lines 104 - 111)

## Model Predictive Control with Latency

The Simulator in this case, has a 100ms latency. Latency is the time it takes for the system to perform the updates controls from the moment the singal was issued by the model. Latency should be taken into account in most cases because, the vehicle might not be in the same pocition where it was when We calcuated the control updates when we actually perform them on the vehicle physically. This might introduce lot of inaccuracies. 

There are 2 basic ways to solve this problem. 

1. The prospective position of the car is estimated based on its current speed and heading direction by propagating the position of the car forward until the expected time when actuations are expected to have an effect. The NMPC trajectory is then determined by solving the control problem starting from that position.

2. The control problem is solved from the current position and time onwards. Latency is taken into account by constraining the controls to the values of the previous iteration for the duration of the latency. Thus the optimal trajectory is computed starting from the time after the latency period. This has the advantage that the dynamics during the latency period is still calculated according to the vehicle model.

I adapted the 2nd option above and implemeted this in the Solve() function of the MPC class (MPC.cpp Lines 209 - 218).

Here, the `latency` is set to 2 which is 100ms / 5ms (dt). This means that the controls of the first two time steps are not used in the optimization.


---

## Dependencies

* cmake >= 3.5
 * All OSes: [click here for installation instructions](https://cmake.org/install/)
* make >= 4.1
  * Linux: make is installed by default on most Linux distros
  * Mac: [install Xcode command line tools to get make](https://developer.apple.com/xcode/features/)
  * Windows: [Click here for installation instructions](http://gnuwin32.sourceforge.net/packages/make.htm)
* gcc/g++ >= 5.4
  * Linux: gcc / g++ is installed by default on most Linux distros
  * Mac: same deal as make - [install Xcode command line tools]((https://developer.apple.com/xcode/features/)
  * Windows: recommend using [MinGW](http://www.mingw.org/)
* [uWebSockets](https://github.com/uWebSockets/uWebSockets)
  * Run either `install-mac.sh` or `install-ubuntu.sh`.
  * If you install from source, checkout to commit `e94b6e1`, i.e.
    ```
    git clone https://github.com/uWebSockets/uWebSockets 
    cd uWebSockets
    git checkout e94b6e1
    ```
    Some function signatures have changed in v0.14.x. See [this PR](https://github.com/udacity/CarND-MPC-Project/pull/3) for more details.
* Fortran Compiler
  * Mac: `brew install gcc` (might not be required)
  * Linux: `sudo apt-get install gfortran`. Additionall you have also have to install gcc and g++, `sudo apt-get install gcc g++`. Look in [this Dockerfile](https://github.com/udacity/CarND-MPC-Quizzes/blob/master/Dockerfile) for more info.
* [Ipopt](https://projects.coin-or.org/Ipopt)
  * Mac: `brew install ipopt`
       +  Some Mac users have experienced the following error:
       ```
       Listening to port 4567
       Connected!!!
       mpc(4561,0x7ffff1eed3c0) malloc: *** error for object 0x7f911e007600: incorrect checksum for freed object
       - object was probably modified after being freed.
       *** set a breakpoint in malloc_error_break to debug
       ```
       This error has been resolved by updrading ipopt with
       ```brew upgrade ipopt --with-openblas```
       per this [forum post](https://discussions.udacity.com/t/incorrect-checksum-for-freed-object/313433/19).
  * Linux
    * You will need a version of Ipopt 3.12.1 or higher. The version available through `apt-get` is 3.11.x. If you can get that version to work great but if not there's a script `install_ipopt.sh` that will install Ipopt. You just need to download the source from the Ipopt [releases page](https://www.coin-or.org/download/source/Ipopt/).
    * Then call `install_ipopt.sh` with the source directory as the first argument, ex: `sudo bash install_ipopt.sh Ipopt-3.12.1`. 
  * Windows: TODO. If you can use the Linux subsystem and follow the Linux instructions.
* [CppAD](https://www.coin-or.org/CppAD/)
  * Mac: `brew install cppad`
  * Linux `sudo apt-get install cppad` or equivalent.
  * Windows: TODO. If you can use the Linux subsystem and follow the Linux instructions.
* [Eigen](http://eigen.tuxfamily.org/index.php?title=Main_Page). This is already part of the repo so you shouldn't have to worry about it.
* Simulator. You can download these from the [releases tab](https://github.com/udacity/self-driving-car-sim/releases).
* Not a dependency but read the [DATA.md](./DATA.md) for a description of the data sent back from the simulator.


## Basic Build Instructions


1. Clone this repo.
2. Make a build directory: `mkdir build && cd build`
3. Compile: `cmake .. && make`
4. Run it: `./mpc`.

## Tips

1. It's recommended to test the MPC on basic examples to see if your implementation behaves as desired. One possible example
is the vehicle starting offset of a straight line (reference). If the MPC implementation is correct, after some number of timesteps
(not too many) it should find and track the reference line.
2. The `lake_track_waypoints.csv` file has the waypoints of the lake track. You could use this to fit polynomials and points and see of how well your model tracks curve. NOTE: This file might be not completely in sync with the simulator so your solution should NOT depend on it.
3. For visualization this C++ [matplotlib wrapper](https://github.com/lava/matplotlib-cpp) could be helpful.

## Editor Settings

We've purposefully kept editor configuration files out of this repo in order to
keep it as simple and environment agnostic as possible. However, we recommend
using the following settings:

* indent using spaces
* set tab width to 2 spaces (keeps the matrices in source code aligned)

## Code Style

Please (do your best to) stick to [Google's C++ style guide](https://google.github.io/styleguide/cppguide.html).

## Project Instructions and Rubric

Note: regardless of the changes you make, your project must be buildable using
cmake and make!

More information is only accessible by people who are already enrolled in Term 2
of CarND. If you are enrolled, see [the project page](https://classroom.udacity.com/nanodegrees/nd013/parts/40f38239-66b6-46ec-ae68-03afd8a601c8/modules/f1820894-8322-4bb3-81aa-b26b3c6dcbaf/lessons/b1ff3be0-c904-438e-aad3-2b5379f0e0c3/concepts/1a2255a0-e23c-44cf-8d41-39b8a3c8264a)
for instructions and the project rubric.

## Hints!

* You don't have to follow this directory structure, but if you do, your work
  will span all of the .cpp files here. Keep an eye out for TODOs.

## Call for IDE Profiles Pull Requests

Help your fellow students!

We decided to create Makefiles with cmake to keep this project as platform
agnostic as possible. Similarly, we omitted IDE profiles in order to we ensure
that students don't feel pressured to use one IDE or another.

However! I'd love to help people get up and running with their IDEs of choice.
If you've created a profile for an IDE that you think other students would
appreciate, we'd love to have you add the requisite profile files and
instructions to ide_profiles/. For example if you wanted to add a VS Code
profile, you'd add:

* /ide_profiles/vscode/.vscode
* /ide_profiles/vscode/README.md

The README should explain what the profile does, how to take advantage of it,
and how to install it.

Frankly, I've never been involved in a project with multiple IDE profiles
before. I believe the best way to handle this would be to keep them out of the
repo root to avoid clutter. My expectation is that most profiles will include
instructions to copy files to a new location to get picked up by the IDE, but
that's just a guess.

One last note here: regardless of the IDE used, every submitted project must
still be compilable with cmake and make./
