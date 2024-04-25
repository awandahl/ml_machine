# ml_machine
Installation instructions for a garbage room ML machine


![ml](https://github.com/awandahl/ml_machine/assets/62021989/e980add7-a13b-4d02-a754-a597edd56644)

# nbr 1

### Comprehensive System Setup for Machine Learning

#### Hardware Specifications
- **CPU**: Intel(R) Core(TM) i7-6700K CPU @ 4.00GHz
- **GPU**: NVIDIA GTX 970
- **Primary Disk**: 1TB
- **Additional Disk**: 500GB
- **RAM 

#### Initial Ubuntu Installation
1. **Install Ubuntu** using the live installation media.
   - Choose "Something else" during the installation for manual partitioning.
   - Create the following partitions on the 1TB disk:
     - **EFI System Partition**:
       - **Size**: 512 MB
       - **Type**: EFI System Partition (use FAT32 if UEFI)
     - **Root Partition** (`/`):
       - **Size**: 100 GB
       - **Type**: ext4
     - **Swap Partition**:
       - **Size**: 32 GB
     - **Home Partition** (`/home`):
       - **Size**: 200 GB
       - **Type**: ext4
     - **Docker and Container Data Partition** (`/containers`):
       - **Size**: 600 GB
       - **Type**: ext4
   - Format and assign mount points during installation as specified above.

#### Setup the 500GB Disk for Data Storage
2. **Format and Mount the 500GB Disk**:
   - Format the disk and create a single partition using the entire disk:
     ```bash
     sudo mkfs.ext4 /dev/sdb1  # Replace 'sdb1' with the actual disk identifier
     ```
   - Mount the disk to `/data`:
     ```bash
     sudo mkdir /data
     sudo mount /dev/sdb1 /data
     ```
   - Automatically mount on boot by adding to `/etc/fstab`:
     ```bash
     echo 'UUID=$(sudo blkid -s UUID -o value /dev/sdb1) /data ext4 defaults 0 2' | sudo tee -a /etc/fstab
     ```

#### NVIDIA GPU and CUDA Setup
3. **Install NVIDIA Drivers and CUDA**:
   - Install the proprietary NVIDIA drivers and CUDA toolkit for GPU acceleration:
     ```bash
     sudo add-apt-repository ppa:graphics-drivers/ppa
     sudo apt update
     sudo ubuntu-drivers autoinstall
     sudo apt install nvidia-cuda-toolkit
     ```
   - Reboot to apply driver installations:
     ```bash
     sudo reboot
     ```

#### Docker Installation and Configuration
4. **Install Docker and Configure for GPU Support**:
   - Install Docker:
     ```bash
     sudo apt install docker.io
     ```
   - Add the NVIDIA Docker repository and install `nvidia-docker2`:
     ```bash
     distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
     curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
     curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list
     sudo apt update && sudo apt install -y nvidia-docker2
     sudo systemctl restart docker
     ```
   - Configure Docker to use the `/containers` partition:
     ```bash
     echo '{ "data-root": "/containers", "runtimes": { "nvidia": { "path": "nvidia-container-runtime", "runtimeArgs": [] }}}' | sudo tee /etc/docker/daemon.json
     sudo systemctl restart docker
     ```

#### User Configuration and Permissions
5. **Create Users and Manage Permissions**:
   - Create user groups for managing access:
     ```bash
     sudo groupadd dockerusers
     sudo groupadd datascientists
     ```
   - Add a user and configure groups and directories:
     ```bash
     sudo useradd -m -s /bin/bash -G dockerusers,datascientists alice
     sudo passwd alice
     sudo mkdir /data/alice
     sudo chown alice:datascientists /data/alice
     sudo chmod 770 /data/alice
     ```
   - Add user to the Docker group:
     ```bash
     sudo usermod -aG docker alice
     ```

#### Installation of Development Tools for Python and R
6. **Install Python, R, and Related Tools**:
   - Install Python, R, and Jupyter Notebook:
     ```bash
     sudo apt install python3 python3-pip r-base jupyter-notebook
     ```
   - Install Python virtual environment tools and create an environment


# nbr 2

### System Setup for Data Science and Machine Learning

#### Hardware Specifications
- **CPU**: 4 GHz
- **GPU**: NVIDIA GTX 970
- **Disk**: 1TB

#### Step 1: System Partitioning
Partition your 1TB disk during the Ubuntu installation process as follows:
- **EFI System Partition** (for systems with UEFI):
  - **Size**: 512 MB
  - **Type**: EFI System Partition
  - **Filesystem**: FAT32
- **Root Partition** (`/`):
  - **Size**: 100 GB
  - **Type**: ext4
- **Swap Partition**:
  - **Size**: 32 GB
- **Home Partition** (`/home`):
  - **Size**: 200 GB
  - **Type**: ext4
- **Docker and Container Data Partition** (`/containers`):
  - **Size**: 600 GB
  - **Type**: ext4

#### Step 2: Install Ubuntu
- Use a live installation media and choose "Something else" for manual partitioning. Assign mount points and format as specified.

#### Step 3: Setup User Permissions and Groups
- Create user groups and add users. Example for user 'alice':
  ```bash
  sudo groupadd dockerusers
  sudo groupadd datascientists
  sudo useradd -m alice -G dockerusers,datascientists
  sudo passwd alice
  sudo chown root:dockerusers /containers
  sudo chmod 775 /containers
  ```

#### Step 4: Install and Configure Docker
- Install Docker and add users to the Docker group:
  ```bash
  sudo apt install docker.io
  sudo usermod -aG docker alice
  sudo systemctl enable docker
  sudo systemctl start docker
  ```

#### Step 5: Configure NVIDIA Docker
- Install NVIDIA drivers and NVIDIA Docker to enable GPU support in containers:
  ```bash
  distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
  curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
  curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list
  sudo apt update && sudo apt install -y nvidia-docker2
  sudo systemctl restart docker
  ```

#### Step 6: Setup Docker Storage
- Configure Docker to use `/containers` for data:
  ```bash
  echo '{ "data-root": "/containers" }' | sudo tee /etc/docker/daemon.json
  sudo systemctl restart docker
  ```

#### Step 7: Install Python, R, and Virtual Environments
- Install Python, R, and virtual environment tools:
  ```bash
  sudo apt update
  sudo apt install python3 python3-pip r-base python3-venv
  ```
- Encourage the use of virtual environments for Python projects:
  ```bash
  python3 -m venv myprojectenv
  source myprojectenv/bin/activate
  ```

#### Additional Recommendations
- Provide Dockerfiles and guidance for building containerized Python and R environments.
- Set up monitoring, backups, and security measures to ensure a robust operational environment.


# nbr 3

### System Administrator Instructions

#### **1. Initial System Setup and Partitioning**
- **Task**: Install Ubuntu and partition the disks.
- **Details**:
  - Use the 1TB disk for system and primary data partitions. 
  - Partition as follows:
    - **EFI System Partition**: 512 MB for systems with UEFI.
    - **Root Partition** (`/`): 100 GB for OS and applications.
    - **Swap Partition**: 32 GB to support memory overflow and hibernation.
    - **Home Partition** (`/home`): 200 GB for user files and directories.
    - **Docker and Container Data Partition** (`/containers`): 600 GB for Docker images and container data.
  - Format the additional 500GB disk and mount it as `/data` for user data and large datasets.
- **Importance**: Proper partitioning ensures organized data management and optimizes performance. Each partition serves a specific purpose, enhancing security and system efficiency.

#### **2. NVIDIA GPU and CUDA Setup**
- **Task**: Install NVIDIA drivers and the CUDA toolkit.
- **Details**:
  - Add the proprietary graphics drivers PPA, install the drivers, and the CUDA toolkit.
  - This setup enables the system to support machine learning and other GPU-accelerated applications.
- **Importance**: The GPU is crucial for accelerating machine learning tasks. Installing the latest drivers and CUDA toolkit ensures compatibility and optimal performance.

#### **3. Docker Installation and Configuration**
- **Task**: Install Docker and configure it to use the NVIDIA runtime and the `/containers` partition.
- **Details**:
  - Install Docker and `nvidia-docker2` to allow Docker containers to access the GPU.
  - Modify Docker's configuration to change its default data directory to `/containers`.
- **Importance**: Docker allows for the creation of isolated environments for different projects, ensuring reproducibility and scalability. Using the `/containers` partition helps manage storage efficiently.

#### **4. User Management**
- **Task**: Create user accounts, groups, and assign permissions.
- **Details**:
  - Set up groups such as `dockerusers` and `datascientists`.
  - Create user directories under `/data` for each user, ensuring appropriate ownership and permissions.
- **Importance**: Managing user access through groups ensures that permissions are easily controlled and secure. Providing individual data directories under `/data` allows personalized space for large data without cluttering the home directory.

### End-User Instructions

#### **1. Setting Up Python and R Environments**
- **Task**: Create Python virtual environments and R environments using `renv`.
- **Details**:
  - **Python**: Users should create a virtual environment in their personal or data directory to manage Python packages independently of system-wide installations.
    ```bash
    python3 -m venv ~/myenv
    source ~/myenv/bin/activate
    ```
  - **R**: Set up `renv` in project directories to manage R package dependencies locally.
    ```R
    install.packages("renv")
    renv::init()
    ```
- **Importance**: Virtual environments in Python and `renv` in R allow users to maintain project-specific dependencies, avoiding conflicts and ensuring reproducibility of the code.

#### **2. Using Jupyter and RStudio**
- **Task**: Run Jupyter Notebooks and RStudio for interactive development.
- **Details**:
  - **Jupyter**: Start Jupyter from within a virtual environment to access notebooks using the installed packages.
    ```bash
    jupyter notebook
    ```
  - **RStudio**: Launch RStudio from the desktop environment or connect to RStudio Server using a web browser.
- **Importance**: Jupyter and RStudio provide powerful interfaces for data analysis and development. They support interactive coding, visualization, and collaboration.

These detailed instructions ensure that both system administrators and users are equipped to utilize the system effectively, with clear guidelines on setting up and using the environment for machine learning and data science tasks.
