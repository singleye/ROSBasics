[ROS tutorial](http://wiki.ros.org/ROS/Tutorials)

# Workspace basic
## create workspace

```
mkdir -p catkin_ws/src
cd catkin_ws
catkin_make
```

**Python3**

```
catkin_make -DPYTHON_EXECUTABLE=/usr/bin/python3 -DPYTHON_INCLUDE_DIR=/usr/include/python3.7m -DPYTHON_LIBRARY=/usr/lib/libpython3.7m.so
```


```
$ . devel/setup.bash
$ echo $ROS_PACKAGE_PATH
/home/wangq/Projects/ROS/catkin/catkin_ws/src:/opt/ros/melodic/share
```


## create package

```
$ cd catkin_ws/src
$ catkin_create_pkg beginner_tutorials std_msgs rospy roscpp
```

## build package

Usage:

```
# In a catkin workspace
$ catkin_make [make_targets] [-DCMAKE_VARIABLES=...]
$ catkin_make --source my_src
$ catkin_make install --source my_src  # (optionally)
```

```
$ cd catkin_ws
$ catkin_make
```


# msg and srv

* [Message Description Language](http://wiki.ros.org/action/show/msg?action=show&redirect=ROS%2FMessage_Description_Language)

* [Building messages, services or actions](http://docs.ros.org/latest/api/catkin/html/howto/format2/building_msgs.html)

## Create a message:

```
$ roscd beginner_tutorials
$ mkdir msg
$ echo "int64 num" > msg/Num.msg
```

Add dependencies into packages.xml

```
  <build_depend>message_generation</build_depend>
  <exec_depend>message_runtime</exec_depend>
```

Add message_generation into CMakeLists.txt

```
# Do not just add this to your CMakeLists.txt, modify the existing text to add message_generation before the closing parenthesis
find_package(catkin REQUIRED COMPONENTS
   roscpp
   rospy
   std_msgs
   message_generation
)
```

Export the message runtime dependency

```
catkin_package(
  ...
  CATKIN_DEPENDS message_runtime ...
  ...)
```

Uncomment it by removing the # symbols and then replace the stand in Message\*.msg files with your .msg file, such that it looks like this:

```
add_message_files(
  FILES
  Num.msg
)
```

Now we must ensure the generate_messages() function is called.

```
generate_messages(
  DEPENDENCIES
  std_msgs
)
```

Now you can generate new messages:

```
$ roscd beginner_tutorials
$ cd ../..
$ catkin_make install
$ cd -
```

Check the generated messages:

```
$ rosmsg show beginner_tutorials/Num
int64 num
```

Generated:

1. The C++ message header file will be generated in ~/catkin_ws/devel/include/beginner_tutorials/
2. The Python script will be created in ~/catkin_ws/devel/lib/python2.7/dist-packages/beginner_tutorials/msg
3. The lisp file appears in ~/catkin_ws/devel/share/common-lisp/ros/beginner_tutorials/msg/.


## Create srv

```
$ roscd beginner_tutorials
$ mkdir srv
```

Create by copying from an example:

```
$ roscp rospy_tutorials AddTwoInts.srv srv/AddTwoInts.srv
$ cat srv/AddTwoInts.srv
int64 a
int64 b
---
int64 sum
```

Make sure messages are enabled in package.xml

Make sure messages/services are enabled in CMakeLists.txt

Service:

```
add_service_files(
  FILES
  AddTwoInts.srv
)
```


```
$ rossrv show beginner_tutorials/AddTwoInts
int64 a
int64 b
---
int64 sum
```

# Writing programs

## Create topic pub/sub messages

Add 'beginner_tutorials/src/talker.cpp' and 'beginner_tutorials/src/listener.cpp'

Modify CMakeLists.txt accordingly.


### run

```
$ rosrun beginner_tutorials talker      (C++)
$ rosrun beginner_tutorials talker.py   (Python) 
```

## Create service

### Server:

[Writing a Simple Publisher and Subscriber](http://wiki.ros.org/ROS/Tutorials/WritingPublisherSubscriber%28c%2B%2B%29)

```
// define the service callback
bool add(beginner_tutorials::AddTwoInts::Request  &req,
         beginner_tutorials::AddTwoInts::Response &res)
{
...
}


int main(int argc, char **argv)
{
  // declare a handle
  ros::NodeHandle n;
  
  // create the service
  ros::ServiceServer service = n.advertiseService("add_two_ints", add);
}
```


### Client:

[Writing a Simple Service and Client](http://wiki.ros.org/ROS/Tutorials/WritingServiceClient%28c%2B%2B%29)

```
  // declare a handle
  ros::NodeHandle n;
  // create service client with handle
   ros::ServiceClient client = n.serviceClient<beginner_tutorials::AddTwoInts>("add_two_ints");
  // declare service
   beginner_tutorials::AddTwoInts srv;
  srv.request.a = atoll(argv[1]);
  srv.request.b = atoll(argv[2]);
  if (client.call(srv)) {
    // success
  }else {
    // failure
  }
```

### compile:

CMakeLists.txt

```
add_executable(add_two_ints_server src/add_two_ints_server.cpp)
target_link_libraries(add_two_ints_server ${catkin_LIBRARIES})
add_dependencies(add_two_ints_server beginner_tutorials_gencpp)

add_executable(add_two_ints_client src/add_two_ints_client.cpp)
target_link_libraries(add_two_ints_client ${catkin_LIBRARIES})
add_dependencies(add_two_ints_client beginner_tutorials_gencpp)
```


## Create action

[actionlib tutorial](http://wiki.ros.org/beginner_tutorials/Tutorials)

### Define action

Create 'beginner_tutorial/action/Fibonacci.action'

```
#goal definition
int32 order
---
#result definition
int32[] sequence
---
#feedback
int32[] sequence
```

Change 'CMakeLists.txt'

```
find_package(catkin REQUIRED COMPONENTS actionlib_msgs)

add_action_files(
  DIRECTORY action
  FILES Fibonacci.action
)

generate_messages(
  DEPENDENCIES actionlib_msgs std_msgs  # Or other packages containing msgs
)

catkin_package(
  CATKIN_DEPENDS actionlib_msgs
)
```

Change 'package.xml'

```
<build_depend>actionlib_msgs</build_depend>
<build_depend>actionlib</build_depend>
<exec_depend>actionlib</exec_depend>
<exec_depend>actionlib_msgs</exec_depend>
```


### Server

[Writing a Simple Action Server using the Execute Callback](http://wiki.ros.org/beginner_tutorials/Tutorials/SimpleActionServer%28ExecuteCallbackMethod%29)

Create 'src/fibonacci_server.cpp'

```
// declare action server
actionlib::SimpleActionServer<beginner_tutorials::FibonacciAction> as_;

// declare action feedback
beginner_tutorials::FibonacciFeedback feedback_;

// declare action result
beginner_tutorials::FibonacciResult result_;

// declare action callback
void executeCB(const beginner_tutorials::FibonacciGoalConstPtr &goal)
{
    ...

    // construct feedback
    feedback_.sequence.clear();
    feedback_.sequence.push_back(0);
    feedback_.sequence.push_back(1);

    ...

    // publish feedback
    as_.publishFeedback(feedback_);

    // construct result
    result_.sequence = feedback_.sequence;
    as_.setSucceeded(result_);
}

// initialize action server and register action callback
as_(nh_, name, boost::bind(&FibonacciAction::executeCB, this, _1), false)
```


Change 'CMakeLists.txt'

```
add_executable(fibonacci_server src/fibonacci_server.cpp)

target_link_libraries(
  fibonacci_server
  ${catkin_LIBRARIES}
)

add_dependencies(
  fibonacci_server
  ${beginner_tutorials_EXPORTED_TARGETS}
)
```


### Client

[Writing a Simple Action Client](http://wiki.ros.org/beginner_tutorials/Tutorials/SimpleActionClient)

Create 'src/fibonacci_client.cpp'

```
// declare client
actionlib::SimpleActionClient<beginner_tutorials::FibonacciAction> ac("fibonacci", true);

// wait for the action server to start
ac.waitForServer();

// send a goal to the action
beginner_tutorials::FibonacciGoal goal;
goal.order = 20;
ac.sendGoal(goal);

//wait for the action to return
bool finished_before_timeout = ac.waitForResult(ros::Duration(30.0));

// get action state
actionlib::SimpleClientGoalState state = ac.getState();
```

Change 'CMakeLists.txt'

```
add_executable(fibonacci_client src/fibonacci_client.cpp)

target_link_libraries( 
  fibonacci_client
  ${catkin_LIBRARIES}
)

add_dependencies(
  fibonacci_client
  ${beginner_tutorials_EXPORTED_TARGETS}
)
```


# Time control

ros::Rate

ros::Timer

[Understanding Timers](http://wiki.ros.org/roscpp_tutorials/Tutorials/Timers)
[Overview of Timer](http://wiki.ros.org/roscpp/Overview/Timers)
