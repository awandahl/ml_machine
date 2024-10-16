Boot successfully repaired.  

Please write on a paper the following URL:  
https://paste.ubuntu.com/p/Kjjt5kVZSV/  


In case you still experience boot problem, indicate this URL to:  
boot.repair@gmail.com or to your favorite support forum.  

You can now reboot your computer.  
Please do not forget to make your UEFI firmware boot on the The OS now in use - Ubuntu 24.04.1 LTS entry (sda1/efi/ubuntu/grubx64.efi file) !  
If your computer reboots directly into Windows, try to change the boot order in your UEFI firmware.  
If your UEFI firmware does not allow to change the boot order, change the default boot entry of the Windows bootloader.  
For example you can boot into Windows, then type the following command in an admin command prompt:  
bcdedit /set {bootmgr} path \EFI\ubuntu\grubx64.efi  


bcdedit /set {bootmgr} path \EFI\ubuntu\grubx64.efi

# ml_machine
Installation instructions for a garbage room ML machine


https://developer.nvidia.com/cuda-downloads?target_os=Linux&target_arch=x86_64&Distribution=Ubuntu&target_version=22.04&target_type=deb_network

![ml](https://github.com/awandahl/ml_machine/assets/62021989/e980add7-a13b-4d02-a754-a597edd56644)

# nbr 1

### Comprehensive System Setup for Machine Learning

#### Hardware Specifications
- **CPU**: Intel(R) Core(TM) i7-6700K CPU @ 4.00GHz
- **GPU**: NVIDIA GTX 790
- **Primary Disk**: 1TB
- **Additional Disk**: 500GB
- **RAM**: 16GB

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

# sudo ubuntu-drivers install --gpgpu 
sudo apt install nvidia-driver-550

# https://developer.nvidia.com/cuda-11-5-0-download-archive?target_os=Linux&target_arch=x86_64&Distribution=Ubuntu&target_version=20.04&target_type=deb_local   
# aw@gnaget:~$ sudo apt install nvidia-cuda-toolkit  
export PATH=/usr/lib/nvidia-cuda-toolkit/bin:$PATH  
export LD_LIBRARY_PATH=/usr/lib/nvidia-cuda-toolkit/lib64:$LD_LIBRARY_PATH  
export CUDA_HOME=/usr/lib/nvidia-cuda-toolkit  



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
Check nvidia is working, install tensorflow and prerequisites:  https://www.tensorflow.org/install/pip    


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


### Getting the OpenAlex snapshot:

Read: https://docs.openalex.org/download-all-data/download-to-your-machine  

AWS S3 Explorer:  https://openalex.s3.amazonaws.com/browse.html   

Install AWS client
````
sudo apt install awscli
````
About 300Gb which takes approximately three hours to download.

````
aws s3 sync "s3://openalex" "openalex-snapshot" --no-sign-request
````


### Open a new db

First get the DuckDB binary:  
````
wget https://github.com/duckdb/duckdb/releases/download/v0.10.1/duckdb_cli-linux-amd64.zip
````
then

````
./duckdb open_alex_test.duckdb
````

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


Step 2: Create the Dockerfile

    Create a new folder on your computer where you will store the Dockerfile and any other related files. For example, create a folder named rstudio-docker.

    Open a text editor (such as Notepad, VSCode, or any other editor of your choice).

    Paste the following contents into the editor. This is your Dockerfile, which tells Docker how to build your container:

    Dockerfile

    # Base image with R and RStudio Server
    FROM rocker/rstudio:latest

    # Install R packages for text mining
    RUN install2.r --error \
        tm \
        text2vec \
        tidytext \
        wordcloud \
        ggplot2 \
        dplyr

    # Expose RStudio Server port
    EXPOSE 8787

    # Set user password for RStudio
    ENV PASSWORD=your_password_here

    Save the file as Dockerfile in the rstudio-docker folder you created.

Step 3: Build the Docker Image

    Open a terminal (Command Prompt or PowerShell on Windows, Terminal on macOS or Linux).
    Navigate to the directory containing your Dockerfile. For example:

    bash

cd path/to/rstudio-docker

Replace path/to/rstudio-docker with the actual path where your Dockerfile is saved.
Run the following command to build your Docker image:

bash

    docker build -t rstudio-textmining .

    This command tells Docker to build an image named rstudio-textmining from the Dockerfile in the current directory (.).

Step 4: Run the Docker Container

Once the image is built, you can run it:

    Execute the following command in your terminal:

    bash

    docker run -d -p 8787:8787 -e PASSWORD=your_password_here rstudio-textmining

    This starts a container in detached mode (-d), maps port 8787 from the container to port 8787 on your host machine (-p 8787:8787), and sets the environment variable PASSWORD to the password you choose (-e PASSWORD=your_password_here). Replace your_password_here with a strong password of your choice.

Step 5: Access RStudio

    Open a web browser and navigate to http://localhost:8787.
    Log in to RStudio using:
        Username: rstudio
        Password: The password you set in the Docker run command.
These detailed instructions ensure that both system administrators and users are equipped to utilize the system effectively, with clear guidelines on setting up and using the environment for machine learning and data science tasks.


https://hub.docker.com/r/okwrtdsh/anaconda3

https://github.com/xychelsea/anaconda3-docker  
docker exec -it <container-name-or-id> bash    

## SDG


    Create a Dockerfile based on nvidia/cuda:11.0-base-ubuntu20.04
    Download and include the model from Zenodo
    Install R and necessary R packages
    Install the Aurora SDG API
    Set up an entry point to run the API

Here's a step-by-step guide:

    Create a Dockerfile:

text
FROM nvidia/cuda:11.0-base-ubuntu20.04

# Install system dependencies
RUN apt-get update && apt-get install -y \
    r-base \
    r-base-dev \
    libcurl4-openssl-dev \
    libssl-dev \
    libxml2-dev \
    wget \
    python3 \
    python3-pip \
    && rm -rf /var/lib/apt/lists/*

# Download the model from Zenodo
RUN wget https://zenodo.org/record/7304547/files/sdg_classifier_model.h5 -O /app/sdg_classifier_model.h5

# Install R packages
RUN R -e "install.packages(c('plumber', 'reticulate', 'keras', 'tensorflow'), repos='http://cran.rstudio.com/')"

# Install Python dependencies
RUN pip3 install tensorflow==2.4.0 h5py

# Copy the API code (assuming you have the R scripts locally)
COPY api.R /app/api.R

# Set the working directory
WORKDIR /app

# Expose the port your API will run on
EXPOSE 8000

# Start the API
CMD ["R", "-e", "library(plumber); pr <- plumb('api.R'); pr$run(host='0.0.0.0', port=8000)"]

    Create the api.R file:

You'll need to create an R script that implements the Aurora SDG API. Here's a basic structure:

r
library(plumber)
library(keras)
library(tensorflow)

# Load the model
model <- load_model_hdf5("sdg_classifier_model.h5")

#* @apiTitle Aurora SDG Classifier API

#* Classify text into SDGs
#* @param text The text to classify
#* @post /classify
function(text) {
  # Preprocess the text (you may need to implement this based on how the model expects input)
  preprocessed_text <- preprocess_text(text)
  
  # Make prediction
  prediction <- predict(model, preprocessed_text)
  
  # Process the prediction (you may need to implement this based on how the model outputs results)
  result <- process_prediction(prediction)
  
  return(result)
}

# You'll need to implement these functions based on the specific requirements of your model
preprocess_text <- function(text) {
  # Implement text preprocessing
}

process_prediction <- function(prediction) {
  # Implement prediction processing
}

    Build the Docker image:

text
docker build -t sdg-classifier .

    Run the container:

text
docker run --gpus all -p 8000:8000 sdg-classifier





Modellen:  https://zenodo.org/records/5918475

Har du provat med "docker pull ghcr.io/rocker-org/ml-verse:4.4.1"? Det är ju denna basimage som kontarion utökar. Se https://rocker-project.org/images/versioned/cuda.html och där det nämns "nvidia-docker" så är det numera detta som gäller: https://github.com/NVIDIA/nvidia-container-toolkit?tab=readme-ov-file#getting-started


