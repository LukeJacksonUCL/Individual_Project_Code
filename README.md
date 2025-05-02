# Individual_Project_Code

###The Project is split into 3 docker containers, stored as images on Dockerhub
###mobilehand_container:latest holds the modified MobileHand 3D hand pose estimator
###mujoco_container_nohost:latest contains the retargeting code and Shadow Robot hand simulated in Mujoco
###ros_core_container holds the ROS node facilitating communication between the two

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

## 1. Pull the Docker Images

```bash
docker pull lukejackson0425/mobilehand_container:latest
docker pull lukejackson0425/ros_core_container:latest
docker pull lukejackson0425/mujoco_container_nohost:latest
```


## Using the MuJoCo Simulator Container


docker run -it \
  --name mujoco_sim \
  --net mano_bridge \
  --privileged \
  -v /tmp/.X11-unix:/tmp/.X11-unix \
  -e DISPLAY=$DISPLAY \
  lukejackson0425/mujoco_container_nohost:latest

  
