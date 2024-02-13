---
description: Define declarative behaviors for your robot.
---

# Command Programming

The sunset framework utilizes a command framework similar to the known as command base which is used in other systems such as WPILib.  This system works by building small individual commands and then composing complete actions by taking advantage of the linked-list property that these commands possess.&#x20;

## Modifying system state with commands

Imagine we have a Topic called `PublishValue` that does nothing but publish a message which corresponds to some internal state variable. &#x20;

```python
class PublishValue(Topic):
    def __init__(
        self, 
        topic_name="ExampleTopic", 
        is_sim=False
    ):
        super().__init__(topic_name, is_sim)
        self.replace_message_with_log = True
        self.value = 1

    def generate_messages_periodic(self):
        return {"important_value": self.value}
```

For example's sake, lets say we want to modify what this value is using a command. &#x20;

```python
class CommandModifyValue(Command):
    def __init__(self, value: int, pub_val: PublishValue):
        # topics themselves are subscribers
        super().__init__(subscribers=[pub_val])
        self.pub_val = pub_val
    def first_run_behavior(self):
        self.pub_val.value = 0
    def execute(self):
        self.pub_val.value = random.randrange(0,10)
    def is_complete(self) -> bool:
        return self.pub_val.value == 7
```

We can see this command requires us to inject an instance of `PublishValue` into it for modification.  This allows multiple instances of the same type to be used for different systems. &#x20;

In this particular example we can see use of all the user defined methods of `Command` as well as gain an understanding of the lifetime of a `Command`.  Upon initialization of the command, `__init__` os called just like any other class.  Specific implementation should not occur in `__init__` besides just initializing variables as it is likely the command will not be used until well after its initialization. Next we see `first_run_behavior`.  This is a user defined method that occurs once and only once when the command is first popped off the command queue.  This is useful for initializing state or objects used during the lifetime of the command.  Next is `is_complete` which is a flag function the user defines in order to determine if the command has finished running.  In other examples implementors might check if a certain amount of time has elapsed or if a motor encoders position is within some threshold.  This method will return a bool `True` if the command is finished or `False` if it needs to run again.  Finally is the execute method.  This is similar to [periodic](pub-sub-example.md#receiving-messages) from the subscribers / topics before.  This is used where you need to constantly update information until your command is finished such as the case for a proportional feedback controller.

Here is an example of such a command:

```python
class ProportionalControl(Command):
    def __init__(self, desired_pos: int, motor_controller: MotorController):
        # topics are subscribers!
        super().__init__(subscribers=[motor_controller])
        self.error = 0 
        self.desired_pos = desired_pos
        self.threshold = 10 
        self.kp = 0.1
    def first_run_behavior(self):
        self.error = self.desired_pos - motor_controller.read_pos()
    def execute(self):
        self.error = self.desired_pos - motor_controller.read_pos()
        self.motor_controller.write(self.error * self.kp)
    def is_complete(self) -> bool:
        return Math.abs(error) < threshold
```

{% hint style="success" %}
Just like the other periodic like methods, it is expected you do not write blocking code which halts the main loop.
{% endhint %}
