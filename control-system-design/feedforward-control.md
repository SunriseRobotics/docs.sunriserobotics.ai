---
description: Feedforward predicts control inputs
---

# Feedforward Control

Feedforward or open-loop control is a way to control your robot actuators by estimating what the optimal inputs to the system are.  In robotics,  most systems are powered by dc motors and for these type of systems a generalizable motion model exists



$$
u=k_v\cdot\dot{x}+k_a\cdot\ddot{x}
$$

This equation relates our motor input 'u' to the desired velocity and acceleration of the robot.  Since giving a motor a constant power will result in convergence to a constant velocity we can find the voltage/velocity slope to use as our value of kV.  Similarly,  since input is proportional to force and force is proportional to acceleration, we can find the input / acceleration relationship to be the value of kA.  This simple linear model can also be converted to a [transfer function for more advanced ](https://docs.google.com/document/d/1z8FsP15JZzCeDjgSEzDGUuL1snXx7JuwIbqEBelWHRM/edit?usp=sharing)control.&#x20;

You can use [SimpleFeedforward](https://github.com/SunriseRobotics/sunset-robotics-framework/blob/3d7e09389526ef806cb3da6ebece8dbc4437d15d/sunset\_math/AutomaticControl/LinearControl.py#L41) to implement this controller within sunset. &#x20;

<pre class="language-python"><code class="lang-python"><strong>kv = volts / meter / second
</strong><strong>ka = volts / meter / second ^ 2 
</strong><strong>
</strong><strong># static is the minimum power needed to overcome friction.
</strong><strong>feedforward = SimpleFeedforward(kv,ka,static = 0)
</strong><strong>voltage = feedforward.update(velocity, acceleration)
</strong><strong>motor.setPower(voltage)
</strong></code></pre>

## Rejecting nonlinear effects

Our equation above assumes that our system is linear -- which it is not.  In many cases this assumption is adequate but in others we require additional terms to combat nonlinear effects.

The most common addition is a static friction or kstatic term implemented as&#x20;



$$
u_s = sign{(\dot{x})} \cdot k_s
$$



{% hint style="success" %}
Notice this is part of the SimpleFeedforward example!&#x20;
{% endhint %}

This term takes the direction that your robot should be traveling and multiplies it by the static friction constant, effectively canceling the effect that friction has on the system.  For rotating arms,  implementing this to factor in the cos(theta) of your arms angle will work well and produce a robust result.&#x20;











