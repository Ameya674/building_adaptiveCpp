
# Building AdaptiveCpp with LLVM, Docker and Nvidia Container Toolkit on Ubuntu

## AdaptiveCpp
AdaptiveCpp is a community-driven platform for programming with C++ that works with both CPUs and GPUs from different vendors. It helps applications automatically adjust to any hardware in the system, allowing a single program to run on various types of hardware or use multiple hardware types at the same time.

### Docker
Docker is a tool that makes it easy to create, share, and run applications in isolated environments called containers. These containers bundle everything the application needs to run, ensuring it works the same everywhere.

### Nvidia Container Toolkit
NVIDIA Container Toolkit is a set of tools that lets you use NVIDIA GPUs with Docker containers. It ensures your applications can take advantage of GPU power for faster performance.






## 1) Installing Docker

```bash
sudo apt update                    # Update the package list
sudo apt install -y docker.io      # Install Docker
sudo systemctl enable docker       # Enable Docker to start on boot
sudo systemctl start docker        # Start the Docker service
sudo docker run hello-world        # Run a container to verify the installation
```

## 2) Installing Nvidia Container Toolkit

```bash
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
  && curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
    sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
    sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list                                      # Add the NVIDIA Container Toolkit repository

sudo apt update                                                                                         # Update the package list
sudo apt install -y nvidia-container-toolkit                                                            # Install the NVIDIA Container Toolkit
sudo nvidia-ctk runtime configure --runtime=docker                                                      # Configure the NVIDIA runtime for Docker
sudo systemctl restart docker                                                                           # Restart the Docker service

sudo docker run --rm --runtime=nvidia --gpus all ubuntu nvidia-smi                                      # Run a test container to verify GPU support
```
If the the Nvidia GPU is detected the otuput should look like this
```bash
Mon Jun 24 22:28:09 2024       
+---------------------------------------------------------------------------------------+
| NVIDIA-SMI 535.154.05             Driver Version: 535.154.05   CUDA Version: 12.2     |
|-----------------------------------------+----------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |         Memory-Usage | GPU-Util  Compute M. |
|                                         |                      |               MIG M. |
|=========================================+======================+======================|
|   0  NVIDIA GeForce GTX 1650        Off | 00000000:01:00.0 Off |                  N/A |
| N/A   40C    P8               2W /  15W |      6MiB /  4096MiB |      0%      Default |
|                                         |                      |                  N/A |
+-----------------------------------------+----------------------+----------------------+
                                                                                         
+---------------------------------------------------------------------------------------+
| Processes:                                                                            |
|  GPU   GI   CI        PID   Type   Process name                            GPU Memory |
|        ID   ID                                                             Usage      |
|=======================================================================================|
|    0   N/A  N/A      2383      G   /usr/lib/xorg/Xorg                            4MiB |
+---------------------------------------------------------------------------------------+
```

Now that we have installed Docker and Nvidia Container Toolkit, let's build AdaptiveCpp.

## 3) Creating a Docker Container

```bash
docker run -it --gpus all --runtime=nvidia --name test nvidia/cuda:11.6.1-devel-ubuntu20.04    # Run Docker container with CUDA 11.6.1 and GPU support
docker exec -it test /bin/bash                                                                 # Start interactive bash session in container 'test'
nvidia-smi                                                                                     # To test if GPU is detected
```



## 4) Environment Variables

Inside the container, add these environment variables.

`export PATH=/usr/local/cuda/bin${PATH:+:${PATH}}`

`export LD_LIBRARY_PATH=/usr/local/cuda/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}`



## 5) Installing Dependencies

A few dependencies are required to build AdaptiveCpp

```bash
apt update && apt install -y sudo                                                         # Update package list and install sudo
sudo apt update && sudo apt install -y lsb-release wget software-properties-common gnupg  # Update package list and install essential tools
sudo apt install -y python3 cmake libboost-all-dev git build-essential                    # Install Python 3, CMake, Boost libraries, Git, and build essentials
```

## 6) Installing LLVM
```bash
wget https://apt.llvm.org/llvm.sh                                                                         # Convenience script that sets up the repositories
chmod +x llvm.sh                                                                                          # Make the script executable
sudo ./llvm.sh 16                                                                                         # Set up repositories for clang 16
sudo apt update && sudo apt install -y libclang-16-dev clang-tools-16 libomp-16-dev llvm-16-dev lld-16    # Install LLVM/Clang 16 and related packages
```

## 7) Building AdaptiveCpp

Clone the repo and build using CMake

```bash
git clone https://github.com/AdaptiveCpp/AdaptiveCpp  # Clone the AdaptiveCpp repository
cd AdaptiveCpp                                        # Change directory to AdaptiveCpp
mkdir build && cd build                               # Create and enter the build directory
cmake -DWITH_CUDA_BACKEND=ON ..                       # Configure the build with CUDA backend enabled
make install                                          # Build and install the project
```

Run the following command 
```bash
acpp-info -l
```

If the GPU is found, you see this
```bash
=================Backend information===================
Loaded backend 0: CUDA
  Found device: NVIDIA GeForce GTX 1650
Loaded backend 1: OpenMP
  Found device: hipSYCL OpenMP host device
```

We successfully built and compiled AdaptiveCpp !!! We can use it to compile our applications.

# Acknowledgements

 - [SYCL: A Portable Alternative to CUDA](https://chsasank.com/sycl-portable-cuda-alternative.html)
