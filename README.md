Turtlesim: A simple ROS publisher/subscriber program. 
=====================================================

Step 1: 
-------
we are simply creating a publisher node, which publishes the desired trajectory over the topic subscribed to by turtlesim, cmd_vel. To create a node, we must first create a package, using catkin_create_pkg, ensuring that it is created in the src file of the catkin workspace. To prevent having to manually enter the dependencies of our package, we can list the required dependencies after the package name:

> $catkin_create_pkg turtlesim_hw1 std_msgs rospy std_srvs rostime geom

The turtlesim ROS wiki page has information on all the dependencies and nodes, which becomes very useful when you want to use turtlesim or write nodes to communicate with it. 

To write a node that will publish velocity commands to the turtlesim node, we must first derive our motion model using velocity kinematics. The turtle is a “unicycle model”, which essentially means only two values of the cmd_vel topic, namely the total linear velocity and the total angular velocity, are required and sufficient when integrating the turtle’s kinematics.
Knowing the desired x and y coordinates of the desired trajectory, we can find the ODEs for linear and angular velocity in Mathematica and translated into the following python code, where the acceleration trajectory is defined for getting a
figure eight. The function get_vel, finds the linear and angular velocity at time t=the instance of control
functions u_t = (v, w). 

```
	v_x = 12*pi*cos(4*pi*t/T)/T
	v_y = 6*pi*cos(2*pi*t/T)/T

	a_x = (-48*pi**2*sin(4*pi*t/T))/T**2
	a_y = (-12*pi**2*sin(2*pi*t/T))/T**2
	
	velocity_linear = sqrt(v_x**2 + v_y**2)
	
	theta = atan(v_x/v_y)

	velocity_angular = ((a_y*v_x)-(a_x*v_y))/((v_x*v_x)+(v_y*v_y))
```

Now that the linear and angular velocities are defined, we can send them to the cmd_vel topic, to be heard
by turtlesim. To send the data, we must create a publisher node: 

```
def send_vel_command():
	
	pub = rospy.Publisher('turtle1/cmd_vel',Twist, queue_size = 10)
	rospy.init_node('send_vel_command', anonymous=True)
	r = rospy.Rate(62.5) #62.5hz

	while not rospy.is_shutdown():


		t = rospy.get_time()
		velocity_linear=get_vel(t)[0]
		velocity_angular= get_vel(t)[1]
		velocities = Twist(Vector3((velocity_linear),0,0), Vector3(0,0,(velocity_angular)))
		rospy.loginfo(velocities)
		pub.publish(velocities)


		r.sleep()
```

The first two lines of code after the function definition, define the publisher. The topic that data is being
published to is ‘turtle1/cmd_vel’, and the message type is Twist, Vector 3 (x, y, theta). rospy.Rate sets the
rate at which the data is being published.
The publisher node receives the calculated velocities from get_vel, and publishes them onto the topic.

Always remember to make the file executable!
```
#!/usr/bin/env python
```

In order for the code to run, the following libraries are required:

```
import rospy
from std_msgs.msg import String
from geometry_msgs.msg import Twist, Vector3
import math
from math import cos, sin, tan, atan, pi, sqrt
from turtlesim.srv import TeleportAbsolute
import sys
```

Turtlesim.srv is required by the turtle1/teleport_absolute service. Geometry_msgs.msg/Twist/Vector 3 are
used by the cmd_vel topic. The sys library is used when getting input from user from the command line
prompt.
Everytime turtlesim1 is run, the turtle starts at abstract locations. To define the initial configuration of the turtle, we can take advantage of the TeleportAbsolute service: 

```
rospy.wait_for_service('turtle1/teleport_absolute')
turtle1_teleport_absolute = rospy.ServiceProxy('turtle1/teleport_absolute', TeleportAbsolute)
resp1 = turtle1_teleport_absolute(5.54, 5.54, 0.5)
```

The period T it takes for the turtle to complete one 8-shaped trajectory can be input by a user from the
command line with the following simple command. The node accepts the user input as a private parameter
T. 

>T = input("How fast should Mr.Happy Turtle run?")

Rosbags can be used to see the output of a node without having to run the node. To create a rosbag,
you must first create a directory. Note that the bagfile is created in whichever directory the mkdir command
is given. 

>$mkdir ~/bagfiles

To create an instance of a bag file, we must start from inside the bagfile directory. To record 1 bag with
enough messages to show the turtle complete one circuit, we have rospy.Rate = 60 hz, and T=8, so it takes
the turtle 60*8= 480 messages to complete a cycle. Since $ rosbag record must be run first, then the
publisher node in order to record a full cycle, we store 500 message instances to encapsulate an entire
turtle circuit. 

>$cd ~bagfiles
>$rosbag record –l 500 /turtle/cmd_vel

To ensure one complete cycle was recorded, we can playback the bagfile:

>$rosbag play 2014-10-17-16-49-42.bag

