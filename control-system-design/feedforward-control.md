---
description: Feedforward predicts control inputs
---

# Feedforward Control

Feedforward or Open - Loop control is a way to control your robot by estimating what the optimal inputs to the system are.  In robotics,  most systems are powered by dc motors and for these type of systems a generalizable motion model exists



$$
u=k_v\cdot\dot{x}+k_a\cdot\ddot{x}
$$

This equation relates our motor input 'u' to the desired velocity and acceleration of the robot.  Since giving a motor a constant power will result in convergence to a constant velocity we can find the voltage/velocity slope to use as our value of kV.  Similarly,  since input is proportional to force and force is proportional to acceleration, we can find &#x20;



