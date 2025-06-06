# Individual_Project_Code

The Project is split into 3 docker containers, stored as images on Dockerhub
- mobilehand_container:latest holds the modified MobileHand 3D hand pose estimator
- mujoco_container_nohost:latest contains the retargeting code and Shadow Robot hand simulated in Mujoco
- ros_core_container holds the ROS node facilitating communication between the two

## Prerequisites
- **Host OS:**
  - Linux distro (native docker). The project was developed on Ubuntu 24.04.  
  - Windows/macOS with Docker Desktop (WSL2 backend on Windows).  
- **Docker:**  
  - **Ubuntu:**  
    ```bash
    sudo apt-get update && sudo apt-get install -y docker.io docker-compose
    ```  
  - **Windows/macOS:**  
    Follow the official guide: https://docs.docker.com/get-docker/
---

## Pull the Docker Images

```bash
docker pull lukejackson0425/mobilehand_container:latest
docker pull lukejackson0425/ros_core_container:latest
docker pull lukejackson0425/mujoco_container_nohost:latest
```
## Create a Docker Network

```bash
docker network create ros_network
```

## Start the ROS Noetic master in detached mode, auto‑restarting on failure:

```bash
docker run -d \
  --name ros_core_container \
  --network ros_network \
  --restart unless-stopped \
  lukejackson0425/ros_core_container:latest \
  roscore
```

## Run the MobileHand Container in a new terminal
Ensure necessary permissions:
```bash
xhost +local:
```
Start the container
```bash
docker run -it \
  --name mobilehand_container \
  --network ros_network \
  --restart unless-stopped \
  -v $(pwd)/mobilehand:/workspace/mobilehand \
  lukejackson0425/mobilehand_container:latest \
  /bin/bash
```

## Verify the Necessary Environment Variables Inside the MobileHand Container
```bash
echo $ROS_MASTER_URI    # http://ros_core_container:11311
echo $ROS_HOSTNAME      # mobilehand_container
```

If these do not return as expected, edit ~/.bashrc to add:
```bash
source /opt/ros/noetic/setup.bash
export ROS_MASTER_URI=http://ros_core_container:11311
export ROS_HOSTNAME=mobilehand_container
```

Then reload:
```bash
source ~/.bashrc
```

## Launch the Hand Pose Estimator
Activate the Conda environment:
```bash
conda activate mhand38
```

Run the pose estimator:
```bash
cd mobilehand/code
python demo.py -m camera
```

## Start the MuJoCo Simulator Container in another termial

```bash
docker run -it \
  --name mujoco_sim \
  --network ros_network \
  --privileged \
  -v /tmp/.X11-unix:/tmp/.X11-unix \
  -e DISPLAY=$DISPLAY \
  lukejackson0425/mujoco_container_nohost:latest
```

To start the container and attach, run:
```bash
docker start -ai mujoco_sim
```

When in the container, run:
```bash
cd projects/shadow_robot/base_deps
source ~/projects/shadow_robot/base_deps/install_isolated/setup.bash
export ROS_MASTER_URI=http://127.0.0.1:11311
export ROS_HOSTNAME=127.0.0.1
roslaunch sr_mano_retarget teleop_sim.launch
```


## References

This project was developed using [MobileHand][mobilehand], the [Mujoco] simulator, and the [Shadow Robot] hand:


[mobilehand]: https://github.com/gmntu/mobilehand
[MuJoCo]: https://mujoco.org/
[Shadow Robot]: https://shadowrobot.com/ 


  
