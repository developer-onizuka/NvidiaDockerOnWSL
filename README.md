# NvidiaDockerOnWSL

# 1. Install WSL on Windows10 (22H2)
Install WSL
```
> wsl --install
```
Check the version of WSL
```
> wsl --version
```
```
> wsl -l
Linux 用 Windows サブシステム ディストリビューション:
Ubuntu (既定)

> wsl -d Ubuntu uname -r
5.15.133.1-microsoft-standard-WSL2
```
```
Start WSL
```
wsl
```

# 2. Install Nvidia Driver
Go to the URL below:
> https://www.nvidia.com/Download/index.aspx?lang=en-us

# 3. Install CUDA in your Ubuntu on WSL
```
sudo apt-key del 7fa2af80
wget https://developer.download.nvidia.com/compute/cuda/repos/wsl-ubuntu/x86_64/cuda-wsl-ubuntu.pin
sudo mv cuda-wsl-ubuntu.pin /etc/apt/preferences.d/cuda-repository-pin-600
wget https://developer.download.nvidia.com/compute/cuda/12.0.0/local_installers/cuda-repo-wsl-ubuntu-12-0-local_12.0.0-1_amd64.deb
sudo dpkg -i cuda-repo-wsl-ubuntu-12-0-local_12.0.0-1_amd64.deb
sudo cp /var/cuda-repo-wsl-ubuntu-12-0-local/cuda-*-keyring.gpg /usr/share/keyrings/
sudo apt-get update
sudo apt-get -y install cuda
```

# 4. Run the docker service and some containers
```
$ sudo service docker start
```
```
$ sudo docker images
REPOSITORY        TAG       IMAGE ID       CREATED         SIZE
xeyes             latest    f01e21226fbd   12 months ago   138MB
ubuntu            latest    6b7dfa7e8fdb   12 months ago   77.8MB
progrium/stress   latest    db646a8f4087   9 years ago     282MB
```
Add the following lines in your .bashrc so that you can find xeyes when starting ubuntu on WSL.
```
xhost +
docker run -d --rm --name xeyes -v /tmp/.X11-unix:/tmp/.X11-unix -e DISPLAY=$DISPLAY xeyes:latest
```

# 5. Run an ubuntu container with GPU
```
$ sudo docker run -it --rm --gpus all --name "ubuntu" ubuntu:latest
root@592053d30c30:/# 
```
Check nvidia-smi
```
root@592053d30c30:/# nvidia-smi
Fri Dec 29 02:42:22 2023
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 515.48.07    Driver Version: 516.40       CUDA Version: 11.7     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                               |                      |               MIG M. |
|===============================+======================+======================|
|   0  Quadro P620         On   | 00000000:01:00.0  On |                  N/A |
| 34%   44C    P0    N/A /  N/A |    384MiB /  2048MiB |      0%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| Processes:                                                                  |
|  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
|        ID   ID                                                   Usage      |
|=============================================================================|
|  No running processes found                                                 |
+-----------------------------------------------------------------------------+
```
