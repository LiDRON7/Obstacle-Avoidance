# OAK D PRO GAZEBO SETUP

First we are gonna need to download two things before we get to actually adding the camera to Gazebo. Make sure you are in your Ubuntu/WSL terminal (the one that doesnt say /px4#. I got confused.)

## Step 1 Install ROS (if u already have ROS can check with rosversion -d u can skip this step)

```powershell
apt update
apt install -y curl gnupg lsb-release

curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.asc | apt-key add -
echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" /etc/apt/sources.list.d/ros1.list
apt update
```

## Step 2 Install de necessary Gazebo ROS packages

```powershell
apt-get update
apt-get install -y curl gnupg2 lsb-release

# Add ROS key
curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.asc | apt-key add -

# Add ROS repo (for focal)
echo "deb http://packages.ros.org/ros/ubuntu focal main" > /etc/apt/sources.list.d/ros1.list

apt-get update
```

```powershell
apt install -y ros-noetic-ros-base ros-noetic-gazebo-ros ros-noetic-gazebo-ros-pkgs ros-noetic-image-view ros-noetic-rqt-image-view

echo "source /opt/ros/noetic/setup.bash" >> ~/.bashrc
source /opt/ros/noetic/setup.bash
```

## Step 3 Install nano. Will need to install to be able to write the gazebo code

```powershell
apt update
apt install -y nano
```

## Step 4 Enter our drone file

```powershell
find . -name "iris.sdf.jinja"
```
The return should be something like:
	./Tools/sitl_gazebo/models/iris/iris.sdf.jinja

Open it:

```powershell
nano ./Tools/sitl_gazebo/models/iris/iris.sdf.jinja
```
Now that your in nano we wanna write this: (make sure its inside of link name='base_link')

```powershell
<!-- OAK-D RGB-D Camera -->
<sensor name="oakd_rgbd" type="depth">
  <pose>0.12 0 0.05 0 0 0</pose>
  <update_rate>30</update_rate>

  <camera>
    <horizontal_fov>1.39626</horizontal_fov>
    <image>
      <width>1280</width>
      <height>720</height>
      <format>R8G8B8</format>
    </image>
    <clip>
      <near>0.1</near>
      <far>20.0</far>
    </clip>
  </camera>

  <plugin name="oakd_ros_rgbd" filename="libgazebo_ros_openni_kinect.so">
    <cameraName>camera</cameraName>
    <frameName>camera_link</frameName>
    <imageTopicName>/camera/rgb/image_raw</imageTopicName>
    <depthImageTopicName>/camera/depth/image_raw</depthImageTopicName>
    <pointCloudTopicName>/camera/depth/points</pointCloudTopicName>
  </plugin>
</sensor>
```

Then do Ctrl + O to save the changes when saved do Ctrl + X to exit nano

You can run make px4_sitl_default gazebo to test if your code is ok. If it runs that means the file reads no errors.

## Step 5 Set up Gazebo/ROS collaboration

```powershell
source /opt/ros/noetic/setup.bash
export GAZEBO_PLUGIN_PATH=$GAZEBO_PLUGIN_PATH:/opt/ros/noetic/lib
export ROS_MASTER_URI=http://localhost:11311
export ROS_IP=127.0.0.1
```

Add the ROS API plugin 

```powershell
nano /path/to/the/worldfile.world
```

In nano inside <world> write 

```powershell
<plugin name="gazebo_ros_api_plugin" filename="libgazebo_ros_api_plugin.so"/>
```

Now we check if the camera is being picked up correctly with

```powershell
rosnode list
rostopic list
```

If /gazebo, /camera/rgb/image_raw, /camera/depth/image_raw, /camaera/depth/points appears it means we are in the clear

##Step 6 Final Setup and Test

Run ROS

```powershell
source /opt/ros/noetic/setup.bash
roscore
```

In a 2nd terminal we will run our gazebo

```powershell
make px4_sitl_default gazebo
```

Now that Gazebo simulator is open, in a 3rd terminal find the current docker and run the test

```powershell
docker ps
docker exec -it <container_id> bash
source /opt/ros/noetic/setup.bash
rostopic echo -n 1 /camera/depth/points
```

Where you run this the values in the point cloud should change if you move the obstacle. Everything is working as intended.

