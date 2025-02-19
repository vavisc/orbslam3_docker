# ORB_SLAM3 docker

This docker is based on <b>Ros Noetic Ubuntu 20</b>. If you need melodic with ubuntu 18 checkout #8fde91d

There are two versions available:
- CPU based (Xorg Nouveau display)
- Nvidia Cuda based. 

To check if you are running the nvidia driver, simply run `nvidia-smi` and see if get anything.

Based on which graphic driver you are running, you should choose the proper docker. For cuda version, you need to have [nvidia-docker setup](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html) on your machine.

---

## Compilation and Running


Steps to compile the Orbslam3 on the sample dataset:

- ```./download_dataset_sample.sh```
- ```build_container_cpu.sh``` or ```build_container_cuda.sh``` depending on your machine.

Now you should see ORB_SLAM3 is compiling. 
- Download A sample MH02 EuRoC example and put it in the `Datasets/EuRoC/MH02` folder
```
mkdir -p Datasets/EuRoC 
wget -O Datasets/EuRoC/MH_02_easy.zip http://robotics.ethz.ch/~asl-datasets/ijrr_euroc_mav_dataset/machine_hall/MH_02_easy/MH_02_easy.zip
unzip Datasets/EuRoC/MH_02_easy.zip -d Datasets/EuRoC/MH02
```

Start container
```docker exec -it orbslam3 bash```

Both Python 2 and Python 3.8 are installed inside of the container, but it is important that Python 3 is seen as default for the build, so you can do:
```
sudo update-alternatives --install /usr/bin/python python /usr/bin/python3 1
sudo update-alternatives --config python
python --version
```

The last line should confirm that Python 3 is now seen as default.

Check OpenCV version opencv_version, it should output 4.4.0.
```
dpkg -s libopencv-dev | grep Version
```

Open CMakeLists.txt and alter lines 36-42 to the following (I do not think this is strictly necessary but I did it just in case):
```
find_package(OpenCV 4.4)
if(NOT OpenCV_FOUND)
  find_package(OpenCV 4.0)
  if(NOT OpenCV_FOUND)
    message(FATAL_ERROR "OpenCV > 4.0 not found.")
  endif()
endif()
```

Open /Examples/ROS/ORB_SLAM3/CMakeLists.txt and add project(ORB_SLAM3) below cmake_minimum_required(VERSION 2.4.6).
Also changes lines 33-42 to:

```
find_package(OpenCV 4.4 QUIET)
if(NOT OpenCV_FOUND)
   find_package(OpenCV 3.0 QUIET)
   if(NOT OpenCV_FOUND)
      message(FATAL_ERROR "OpenCV > 3.0 not found.")
   endif()
endif()

find_package(Eigen3 3.1.0 REQUIRED)
find_package(Pangolin REQUIRED)
```

Finally, build the ROS package with build_ros.sh inside of /ORB_SLAM3 (make sure it is executable first). You should now be able to run the ROS nodes as per the examples in the original ORB_SLAM3 repo.
