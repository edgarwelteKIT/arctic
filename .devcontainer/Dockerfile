# Use the NVIDIA CUDA 11.6 base image
FROM nvidia/cuda:11.6.2-base-ubuntu20.04
ARG USERNAME=USERNAME
ARG USER_UID=1000
ARG USER_GID=$USER_UID
ARG DEBIAN_FRONTEND=noninteractive


# Delete user if it exists in container (e.g Ubuntu Noble: ubuntu)
RUN if id -u $USER_UID ; then userdel `id -un $USER_UID` ; fi


# Create the user
RUN groupadd --gid $USER_GID $USERNAME \
    && useradd --uid $USER_UID --gid $USER_GID -m $USERNAME \
    #
    # [Optional] Add sudo support. Omit if you don't need to install software after connecting.
    && apt-get update \
    && apt-get install -y sudo \
    && echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME \
    && chmod 0440 /etc/sudoers.d/$USERNAME
RUN apt-get update && apt-get upgrade -y
ENV SHELL /bin/bash


# Install dependencies
RUN apt-get update && apt-get install -y \
    wget \
    bzip2 \
    ca-certificates \
    curl \
    git \
    && rm -rf /var/lib/apt/lists/*

RUN apt-get update && apt-get upgrade -y && \
    apt-get install -y --no-install-recommends \
    sudo \
    locales \
    lsb-release \
    software-properties-common \
    python3-pip \
    gdb \
    gedit \
    xtermcontrol \
    nano \
    unzip

RUN apt-get update && apt-get install ffmpeg libsm6 libxext6  -y
RUN apt-get install libxcb-xinerama0 -y
RUN apt-get install -y build-essential
RUN apt-get install '^libxcb.*-dev' libx11-xcb-dev libglu1-mesa-dev libxrender-dev libxi-dev libxkbcommon-dev libxkbcommon-x11-dev -y

# Install miniconda
ENV CONDA_DIR /opt/conda
RUN wget --quiet https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O ~/miniconda.sh && \
    chmod +x ~/miniconda.sh && \
    /bin/bash ~/miniconda.sh -b -p /opt/conda && \
    rm ~/miniconda.sh

# Put conda in path so we can use conda activate
ENV PATH=$CONDA_DIR/bin:$PATH
RUN conda update -y conda

RUN set -x && \
    ln -s /opt/conda/etc/profile.d/conda.sh /etc/profile.d/conda.sh && \
    echo ". /opt/conda/etc/profile.d/conda.sh" >> /home/${USERNAME}/.bashrc && \
    echo "conda activate arctic_env" >> /home/${USERNAME}/.bashrc && \
    find /opt/conda/ -follow -type f -name '*.a' -delete && \
    find /opt/conda/ -follow -type f -name '*.js.map' -delete && \
    /opt/conda/bin/conda clean -afy

# Create a new conda environment with Python 3
RUN conda create -n arctic_env python=3.10 -y

# Activate the conda environment
SHELL ["conda", "run", "-n", "arctic_env", "/bin/bash", "-c"]

# Copy the requirements file from the workspace
COPY requirements.txt /home/ws/requirements.txt

# Install the requirements
RUN pip install --no-cache-dir -r /home/ws/requirements.txt

# Install conda packages
RUN conda install -y pytorch=1.13.0 torchvision pytorch-cuda=11.6 -c pytorch -c nvidia
RUN conda install -c fvcore -c iopath -c conda-forge -y fvcore iopath
RUN conda install -c bottler -y nvidiacub
RUN conda install -y pytorch3d -c pytorch3d

RUN pip uninstall -y numpy
RUN pip install numpy==1.22.4
RUN pip install moderngl-window==2.4.6

# Uncomment the following line in the body_models.py file
RUN sed -i 's/# joints = self.vertex_joint_selector(vertices, joints)/joints = self.vertex_joint_selector(vertices, joints)/g' /opt/conda/envs/arctic_env/lib/python3.10/site-packages/smplx/body_models.py

# write to bashrc
RUN echo "export CUDA_HOME=/usr/local/cuda-11.6" >> /home/${USERNAME}/.bashrc
RUN echo "export PATH=\"/usr/local/cuda-11.6/bin:$PATH\"" >> /home/${USERNAME}/.bashrc
RUN echo "export CPATH=\"/usr/local/cuda-11.6/include:$CPATH\"" >> /home/${USERNAME}/.bashrc
RUN echo "export LD_LIBRARY_PATH=\"$LD_LIBRARY_PATH:/usr/local/cuda-11.6/lib64/\"" >> /home/${USERNAME}/.bashrc

# Set working directory
WORKDIR /home/ws

# Start with bash
CMD ["/bin/bash"]
