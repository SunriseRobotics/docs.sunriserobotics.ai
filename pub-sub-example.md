---
description: Here we show how Sunset has been integrated onto physical hardware.
---

# Pub Sub Example

For this section you might find it helpful to follow along with the code [here](https://github.com/SunriseRobotics/sunset-robotics-framework/blob/b0035926074fbf63880b998595b22a65321b9ce4/example/mecanum\_roboclaw\_example.py).&#x20;

Here is a video of the robot in particular.

{% embed url="https://github.com/BenCaunt/sunset-framework/assets/19732253/20ace386-5996-44c1-bff9-92a93b6a04af" %}
Video of mecanum mobile robot running sunset.  Left shows visualization of odometry, right is a video fo the actual robot.  Behind the robot video is an SSH connection where the `main.py`file is run.
{% endembed %}



<figure><img src=".gitbook/assets/Mecanum Drive Diagram Sunset.png" alt=""><figcaption><p>Block Diagram of the above robot featuring roboclaw motor controllers,  raspberry pi 4,  and Gobilda yellow jacket motors </p></figcaption></figure>

Our robot is  designed with four independently controlled mecanum wheels allowing for omni-directional motion. The system architecture employs a distributed approach where each motor controller operates as an individual node within the robot’s communication network. The topic / subscriber is integral to this setup, enabling systematic structure regarding how data flows through the system and how this is used to build high level behaviors.

In our network graph, these motor controllers are visualized as the "leaf" nodes, signifying their role as data endpoints in the larger robotic system. Each topic in the pub-sub system is treated as a discrete unit of computation, seamlessly passing messages and sensor readings along the chain of topics. This structure ensures that data flows logically from one topic to the next, mirroring the actual data pathways within the robot’s operational framework.

***

**Mecanum Wheel Robotics System Network**

```sh
- Roboclaw Motor Controllers (Leaf Nodes)
  - Topic: Wheel Velocities
- Raspberry Pi 4 (Central Node)
  - Management of Pub-Sub Topics
  - Data Aggregation
- Gobilda Yellow Jacket Motors
  - Physical motion of robot
```

By encapsulating each motor controller’s outputs into distinct topics, we maintain a clean and scalable system design, where extending functionality or troubleshooting becomes a matter of addressing specific units of computation within the pub-sub hierarchy.



<figure><img src=".gitbook/assets/Screenshot 2024-02-07 at 3.32.08 AM.png" alt=""><figcaption><p>In this image we can clearly see the MotorVelocity topic from above as the end leaf of this tree.  </p></figcaption></figure>

{% hint style="info" %}
Sunset is automatically able to determine that this node should have its execute method run first, then system time, then robot velocity, and finally once all the prerequisite data has been acquired, then can we compute the odometry update for this loop.  &#x20;
{% endhint %}



## How do we build a Topic?&#x20;

Topics in Sunset begin by inheriting from the Topic abstract class found in the architecture.architecture\_relationships module.&#x20;

```python
class Topic(Subscriber):

    def __init__(self, name="Abstract Topic", is_sim=False):

```

Something important to note is that each instanciated topic must have a different name.  For example if two roboclaw topics were to be spawned you must do something such as `sensor1` `sensor2`

This is because sunset uses name as the ID used to mark things such as which message came from which Topic as well as which topic should go first in execution order.  A runtime check is performed to help alleviate this type of error but due to the ease of bypassing this check it is recommended to pay special attention to topic names. &#x20;

```python
class MotorVelocity(Topic):
    """
    Topic that publishes individual motor velocities in ticks from the roboclaw.
    """
    def __init__(self, roboclaw_instance1, roboclaw_instance2, topic_name="MotorVelocity", is_sim=False):
        super().__init__(topic_name, is_sim)
        self.message = {"frontleft": 0, "frontright": 0, "backleft": 0, "backright": 0}
        self.replace_message_with_log = True
        self.roboclaw_instance1 = roboclaw_instance1
        self.roboclaw_instance2 = roboclaw_instance2

    def generate_messages_periodic(self):
        self.message["frontleft"] = self.roboclaw_instance2.ReadSpeedM1(ROBOCLAW_ADDRESS_2)[1]
        self.message["frontright"] = self.roboclaw_instance2.ReadSpeedM2(ROBOCLAW_ADDRESS_2)[1]
        self.message["backleft"] = self.roboclaw_instance1.ReadSpeedM2(ROBOCLAW_ADDRESS_1)[1]
        self.message["backright"] = self.roboclaw_instance1.ReadSpeedM1(ROBOCLAW_ADDRESS_1)[1]
        return self.message
```

Here is the actual example for how a Topic can be created that periodically publishes some sensor value.  In this case, each time the periodic method runs it obtains the estimated velocity from the motor controller and publishes them to its subscribers. &#x20;

## `replace_message_with_log = True`

Notice how the `replace_message_with_log` field is true here.  If you want to take advantage of sunsets logging / replay capabilities you must enable this for every root Topic only. &#x20;



{% hint style="warning" %}
This tells sunset that this Topic is not a pure function and relies on state from API access that is not a receiving message.   As such if we attempt to simulate this Topic there is no way to do that except for to use the logged information. &#x20;
{% endhint %}

```python
class MotorVelocity(Topic):
    """
    Topic that publishes individual motor velocities in ticks from the roboclaw.
    """
    def __init__(self, roboclaw_instance1, roboclaw_instance2, topic_name="MotorVelocity", is_sim=False):
        super().__init__(topic_name, is_sim)
        self.message = {"frontleft": 0, "frontright": 0, "backleft": 0, "backright": 0}
        self.replace_message_with_log = True # LOOK HERE!
        self.roboclaw_instance1 = roboclaw_instance1
        self.roboclaw_instance2 = roboclaw_instance2
```



## Note on blocking loops

Your periodic method of Topics / Subscribers needs to be designed to execute quickly.  You should also need to build it so that its designed to be called over and over again for the life time of your robot.   To ensure safety, sunset does not use threads which means that you should not (and generally speaking cannot) implement your own event loops in the periodic methods. &#x20;

Here is a rough pseudocode example of what is happening behind the scenes&#x20;

<pre class="language-python"><code class="lang-python"># DFS will order topics in dependency first order
topics = [motor_controller, drivetrain, distance_sensor]
# order of subscribers is inserted order
subscribers = [motor_velocity_writer, watchdog]
while True:
<strong>    for topic in topics:
</strong>        message = topic.generate_messages_periodic()
        send_message_to_subs(message, topic)
    for subscriber in subscribers: 
        subscriber.periodic()
</code></pre>

{% hint style="success" %}
See the [`scheduler`](https://github.com/SunriseRobotics/sunset-robotics-framework/blob/b0035926074fbf63880b998595b22a65321b9ce4/architecture/scheduler.py#L105C1-L106C1) to see specific implementation details.&#x20;
{% endhint %}

As a recap this is okay:

```python
def periodic():
    result = do_computation_once()
    new_result = do_something_again(result)
```

And this will break things:

```python
def periodic():
    while True:
        thing_to_do_many_times = something()
```

## Receiving Messages

The messaging bus of sunset allows us to distribute our computation into discrete, testable nodes which gives us confidence and understanding of our system.  Here we will see how to utilize this system.

Here is an example of a Topic that computes the odometry position of the mecanum robot:

```python
class Odometry(Topic):

    def __init__(
        self, 
        topic_name="Odometry_POSE2D",
        is_sim=False
    ):
        super().__init__(topic_name, is_sim)
        self.pose = SE3.from_euler_and_translation(
            0, 0, 0, 0, 0, 0
        )
        self.message = {"X": 0.0, "Y": 0.0, "THETA": 0.0}

    def subscriber_periodic(self):
        dt = self.messages["SystemTime"].message["DeltaTimeSeconds"]
        velocity = self.messages["RobotRelativeVelocity"].message
        twist = SE3.from_angular_and_linear_velocities([velocity["x"] * dt, velocity["y"] * dt, 0], 0, 0
                                                       , velocity["omega"] * dt)
        self.pose *= twist

    def generate_messages_periodic(self):
        self.message["X"] = float(self.pose.translation[0])
        self.message["Y"] = float(self.pose.translation[1])
        self.message["THETA"] = float(self.pose.rotation.to_euler()[2])
        if debug_odom:
            print(self.message)
        return self.message
```

The important part here is what is occuring in `subscriber_periodic`.  Here the robot is reading the time constant from the schedulers timer and then getting the robot relative `(x, y, omega)` velocities to compute with.  Then the `twist` is computed and the pose is transformed, creating the new pose.  Once this has been done, the topic is now ready to be published by `generate_message_periodic` which takes the pose field and extracts the necessary values and stores them as a message. &#x20;

{% hint style="success" %}
Notice how this topic relies on no other information other than what is passed into it.  This means that it can be reconstructed purely based on the messages from other topics.  For best usage with simulation, `replace_message_with_log should be false (default value).`&#x20;
{% endhint %}

{% hint style="danger" %}
If you want simulation to work you must only use the SystemTime topic that is instantiated within the scheduler.  See [here](pub-sub-example.md) for more information.  We can see the system time topic's messages being used for `dt` in the above example. &#x20;
{% endhint %}

{% hint style="warning" %}
We create the message using floats instead of the SE3 pose object because floats are more easy to serialize to JSON for our log. &#x20;
{% endhint %}
