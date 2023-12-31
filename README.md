# NvidiaDockerOnWSL

# 1. Install Nvidia Driver
- Go to the URL below:
> https://www.nvidia.com/Download/index.aspx?lang=en-us

# 2. Install WSL on Windows10 (22H2)
- Install WSL
```
> wsl --install
```
- Check the version of WSL
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
- Start WSL
```
> wsl
```

# Appendix: sudo without password
```
# sudo visudo
<username> ALL=NOPASSWD: ALL
```

# 3. Install Docker
I recommend the step of CLI. See #3-2.

# 3-1. Install Docker via Docker Desktop for Windows
> https://learn.microsoft.com/ja-jp/windows/wsl/tutorials/wsl-containers

# 3-2. Install Docker via CLI
```
sudo apt install ca-certificates curl gnupg -y
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```
```
echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
```
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```


# 4. Install CUDA in your Ubuntu on WSL
```
sudo apt-key del 7fa2af80
wget https://developer.download.nvidia.com/compute/cuda/repos/wsl-ubuntu/x86_64/cuda-wsl-ubuntu.pin
sudo mv cuda-wsl-ubuntu.pin /etc/apt/preferences.d/cuda-repository-pin-600
wget https://developer.download.nvidia.com/compute/cuda/12.0.0/local_installers/cuda-repo-wsl-ubuntu-12-0-local_12.0.0-1_amd64.deb
sudo dpkg -i cuda-repo-wsl-ubuntu-12-0-local_12.0.0-1_amd64.deb
sudo cp /var/cuda-repo-wsl-ubuntu-12-0-local/cuda-*-keyring.gpg /usr/share/keyrings/
sudo apt update
sudo apt install cuda -y
```

# 4-1. Install nvidia-container-toolkit
```
distribution=$(. /etc/os-release;echo $ID$VERSION_ID) \
      && curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
      && curl -s -L https://nvidia.github.io/libnvidia-container/$distribution/libnvidia-container.list | \
            sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
            sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
```
```
sudo apt update
sudo apt install nvidia-container-toolkit -y
```

# 4-2. Restart the docker service and some containers
```
$ sudo service docker restart
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
$ sudo docker run -it --rm --gpus all --name "ubuntu" ubuntu:20.04
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

# 5-1. Mount a directory from container
```
$ sudo docker run -it --rm --gpus all -v /mnt/c/Temp:/mnt --name "ubuntu" ubuntu:20.04
```
```
$ mount |grep mnt
C:\ on /mnt type 9p (rw,noatime,dirsync,aname=drvfs;path=C:\;uid=1000;gid=1000;symlinkroot=/mnt/,mmap,access=client,msize=65536,trans=fd,rfd=5,wfd=5)
```

# 6. Pull and Run the CUDA and Nvidia-Cuda-Toolkit as a Container
```
sudo docker pull nvidia/cuda:11.7.1-devel-ubuntu20.04
sudo docker run -it --rm --gpus all --name "cuda" nvidia/cuda:11.7.1-devel-ubuntu20.04
```

# 7. Run Matrix Benchmark (行列演算)
```
apt update
apt install git -y
```
```
cd /tmp
git clone https://github.com/developer-onizuka/matrix
cd matrix
apt install libssl-dev -y
./gcc.sh
```
```
# ls 
-rw-r--r-- 1 root root   2463 Dec 29 07:34 matrix_omp.c
-rw-r--r-- 1 root root   4484 Dec 29 07:34 matrix_gds.cu
-rw-r--r-- 1 root root   2993 Dec 29 07:34 matrix.cu
-rw-r--r-- 1 root root   2147 Dec 29 07:34 matrix.c
-rwxr-xr-x 1 root root    401 Dec 29 07:34 gcc.sh
-rw-r--r-- 1 root root   1473 Dec 29 07:34 README.md
-rwxr-xr-x 1 root root  16992 Dec 29 07:36 matrix.o
-rwxr-xr-x 1 root root  17592 Dec 29 07:36 matrix_omp.o
-rwxr-xr-x 1 root root 822624 Dec 29 07:36 matrix.co
-rwxr-xr-x 1 root root 831256 Dec 29 07:36 matrix_gds.co
```
```
# ./matrix.co
```
```
$ nvidia-smi
Fri Dec 29 16:40:55 2023
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 515.48.07    Driver Version: 516.40       CUDA Version: 11.7     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                               |                      |               MIG M. |
|===============================+======================+======================|
|   0  Quadro P620         On   | 00000000:01:00.0  On |                  N/A |
| 46%   61C    P0    N/A /  N/A |    440MiB /  2048MiB |     52%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| Processes:                                                                  |
|  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
|        ID   ID                                                   Usage      |
|=============================================================================|
|    0   N/A  N/A      1641      C   /matrix.co                      N/A      |
+-----------------------------------------------------------------------------+
```

# 8. Clean up
ディストリビューションの登録を解除し、ルート ファイルシステムを削除します。
```
wsl --unregister Ubuntu
```
