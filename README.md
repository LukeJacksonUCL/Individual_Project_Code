# Individual_Project_Code

The Project is split into 3 docker containers, stored as images on Dockerhub
-mobilehand_container:latest holds the modified MobileHand 3D hand pose estimator
-mujoco_container_nohost:latest contains the retargeting code and Shadow Robot hand simulated in Mujoco
-ros_core_container holds the ROS node facilitating communication between the two

## Prerequisites
- **Host OS:**  
  - The project was developed on Ubuntu 24.04, though any Linux distro with native Docker.  
  - Windows/macOS with Docker Desktop (WSL2 backend on Windows).  
- **Docker:**  
  - **Ubuntu:**  
    ```bash
    sudo apt-get update && sudo apt-get install -y docker.io docker-compose
    ```  
  - **Windows/macOS:**  
    Follow the official guide: https://docs.docker.com/get-docker/
---

##Pull the Docker Images

```bash
docker pull lukejackson0425/mobilehand_container:latest
docker pull lukejackson0425/ros_core_container:latest
docker pull lukejackson0425/mujoco_container_nohost:latest
```
##Create a Docker Network

```bash
docker network create my_ros_network
```

##Start the ROS Noetic master in detached mode, auto‑restarting on failure:

```bash
docker run -d \
  --name ros_core_container \
  --network my_ros_network \
  --restart unless-stopped \
  lukejackson0425/ros_core_container:latest \
  roscore
```

##Run the MobileHand Container

```bash
docker run -it \
  --name mobilehand_container \
  --network my_ros_network \
  --restart unless-stopped \
  -v $(pwd)/mobilehand:/workspace/mobilehand \
  lukejackson0425/mobilehand_container:latest \
  /bin/bash
```

##5. Verify the necessary environment variables
```bash
echo $ROS_MASTER_URI    # http://ros_core_container:11311
echo $ROS_HOSTNAME      # mobilehand_container
```


## Using the MuJoCo Simulator Container

```bash
docker run -it \
  --name mujoco_sim \
  --net mano_bridge \
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

  
