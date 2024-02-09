---
description: When robustness is required
---

# PID Control

Feedforward control is great because it requires no sensor inputs.  It however is not necessarily robust to disturbances and cannot perform accurate position control on its own.  This is where PID (proportional intergral derivative) control comes into play. &#x20;

I will assume you are familiar with the overall concepts, if not I've explained it more thoroughly [here](https://www.ctrlaltftc.com/the-pid-controller).  The core principle is that at each iteration we calculate the deviation between the desired state and the measured state and generate the appropriate corrective command to minimize this error.&#x20;

In sunset we recommend for position control systems to use a PD controller (no Integral term).  This is because most disturbances can be accounted for by a simple feedforward term such as the static friction feedforward from the previous section.  Additionally, issues such as integral windup often result in oscillatory behavior which can damage your system. &#x20;

Here is an example on how to use the sunset [PID controller](https://github.com/SunriseRobotics/sunset-robotics-framework/blob/3d7e09389526ef806cb3da6ebece8dbc4437d15d/sunset\_math/AutomaticControl/LinearControl.py#L4)

```python
Kp = 3
Ki = 0
Kd = 0.3
controller = PID(Kp, Ki, Kd)
desired = 10
while True: 
    measurement = get_position()
    # use the scheduler timer to ensure replay mode works.
    msg = scheduler.sysTimeTopic().message
    dt = msg["DeltaTimeSeconds"]
    power = controller.calculate(desired, measurement, dt)
    motor.setPower(power)
```

## Tuning Enhancements

We derived an equation that states given kP, kA, and kD the critically damped (fastest, zero overshoot) gain for kD is:

$$
k_D=2\sqrt{k_a \cdot k_p}-k_v
$$

A proper derivation can be found [here](https://docs.google.com/document/d/1z8FsP15JZzCeDjgSEzDGUuL1snXx7JuwIbqEBelWHRM/edit?usp=sharing)

We provide [two functions](https://github.com/SunriseRobotics/sunset-robotics-framework/blob/3d7e09389526ef806cb3da6ebece8dbc4437d15d/sunset\_math/AutomaticControl/LinearControl.py#L54) to perform this calculation for you.&#x20;

```python
kd = calculateDerivativePositionControl(kp,kv,ka)

# or if you already have feedforward 
# note that static will not effect kD calculation.
feedforward = SimpleFeedforward(kv, ka, static)
kd = derivativeFromFeedforward(kp, feedforward) 
```
