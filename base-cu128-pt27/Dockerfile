################################################################################
# Dockerfile that builds 'yanwk/comfyui-boot:cu128-base'
# An environment for PyTorch 2.7, CUDA 12.8, Python 3.13
################################################################################

FROM docker.io/opensuse/tumbleweed:latest

LABEL maintainer="YAN Wenkun <code@yanwk.fun>"

RUN set -eu

################################################################################
# NVIDIA CUDA devel
# Ref: https://gitlab.com/nvidia/container-images/cuda/
# Break down the steps, so we have more but smaller image layers.

RUN --mount=type=cache,target=/var/cache/zypp \
    printf "\
[cuda-opensuse15-x86_64]\n\
name=cuda-opensuse15-x86_64\n\
baseurl=https://developer.download.nvidia.com/compute/cuda/repos/opensuse15/x86_64\n\
enabled=1\n\
gpgcheck=1\n\
gpgkey=https://developer.download.nvidia.com/compute/cuda/repos/opensuse15/x86_64/D42D0685.pub\n" \
        > /etc/zypp/repos.d/cuda-opensuse15.repo \
    && zypper --gpg-auto-import-keys \
        install --no-confirm --no-recommends --auto-agree-with-licenses \
cuda-cccl-12-8 \
cuda-command-line-tools-12-8 \
cuda-compat-12-8 \
cuda-cudart-12-8 \
cuda-minimal-build-12-8 \
cuda-nvcc-12-8 \
cuda-nvprof-12-8 \
cuda-nvtx-12-8 \
libcublas-12-8 \
libnpp-12-8

RUN --mount=type=cache,target=/var/cache/zypp \
    zypper --gpg-auto-import-keys \
        install --no-confirm --no-recommends --auto-agree-with-licenses \
cuda-cudart-devel-12-8 \
cuda-nvml-devel-12-8 \
cuda-nvrtc-devel-12-8 \
libcublas-devel-12-8 \
libnpp-devel-12-8

RUN --mount=type=cache,target=/var/cache/zypp \
    zypper --gpg-auto-import-keys \
        install --no-confirm --no-recommends --auto-agree-with-licenses \
cuda-libraries-12-8

RUN --mount=type=cache,target=/var/cache/zypp \
    zypper --gpg-auto-import-keys \
        install --no-confirm --no-recommends --auto-agree-with-licenses \
cuda-libraries-devel-12-8

ENV PATH="${PATH}:/usr/local/cuda-12.8/bin" \
    LD_LIBRARY_PATH="${LD_LIBRARY_PATH}:/usr/local/cuda-12.8/lib64" \
    LIBRARY_PATH="${LIBRARY_PATH}:/usr/local/cuda-12.8/lib64/stubs" \
    CUDA_HOME="/usr/local/cuda-12.8"

################################################################################
# Python and tools
# Since this image is so big, we use openSUSE-verified PIP packages for compatibility.

RUN --mount=type=cache,target=/var/cache/zypp \
    zypper addrepo --check --refresh --priority 90 \
        'https://ftp.gwdg.de/pub/linux/misc/packman/suse/openSUSE_Tumbleweed/Essentials/' packman-essentials \
    && zypper --gpg-auto-import-keys \
        install --no-confirm --auto-agree-with-licenses \
python313-devel \
python313-pip \
python313-wheel \
python313-setuptools \
python313-Cython \
python313-py-build-cmake \
python313-aiohttp \
python313-dbm \
python313-ffmpeg-python \
python313-GitPython \
python313-httpx \
python313-joblib \
python313-lark \
python313-matplotlib \
python313-mpmath \
python313-numba-devel \
python313-numpy1 \
python313-onnx \
python313-opencv \
python313-pandas \
python313-qrcode \
python313-rich \
python313-scikit-build \
python313-scikit-build-core-pyproject \
python313-scikit-image \
python313-scikit-learn \
python313-scipy \
python313-svglib \
python313-tqdm \
    && rm /usr/lib64/python3.13/EXTERNALLY-MANAGED \
    && update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.13 100

RUN --mount=type=cache,target=/var/cache/zypp \
    zypper --gpg-auto-import-keys \
        install --no-confirm --auto-agree-with-licenses \
Mesa-libGL1 \
Mesa-libEGL-devel \
libgthread-2_0-0 \
make \
ninja \
git \
aria2 \
fish \
fd \
vim \
which \
opencv \
opencv-devel \
ffmpeg \
x264 \
x265 \
google-noto-sans-fonts \
google-noto-sans-cjk-fonts \
google-noto-coloremoji-fonts \
unzip 7zip

# Temp fix for OpenCV on openSUSE
ENV LD_PRELOAD=/usr/lib64/libjpeg.so.8

################################################################################
# GCC 14 
# Required for compiling CUDA 12.8-related code.

RUN --mount=type=cache,target=/var/cache/zypp \
    zypper --gpg-auto-import-keys \
        install --no-confirm --auto-agree-with-licenses \
gcc14 \
gcc14-c++ \
cpp14 \
    && update-alternatives --install /usr/bin/c++ c++ /usr/bin/g++-14 90 \
    && update-alternatives --install /usr/bin/cc  cc  /usr/bin/gcc-14 90 \
    && update-alternatives --install /usr/bin/cpp cpp /usr/bin/cpp-14 90 \
    && update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-14 90 \
    && update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-14 90 \
    && update-alternatives --install /usr/bin/gcc-ar gcc-ar /usr/bin/gcc-ar-14 90 \
    && update-alternatives --install /usr/bin/gcc-nm gcc-nm /usr/bin/gcc-nm-14 90 \
    && update-alternatives --install /usr/bin/gcc-ranlib gcc-ranlib /usr/bin/gcc-ranlib-14 90 \
    && update-alternatives --install /usr/bin/gcov gcov /usr/bin/gcov-14 90 \
    && update-alternatives --install /usr/bin/gcov-dump gcov-dump /usr/bin/gcov-dump-14 90 \
    && update-alternatives --install /usr/bin/gcov-tool gcov-tool /usr/bin/gcov-tool-14 90 

################################################################################
# PyTorch Packages

RUN --mount=type=cache,target=/root/.cache/pip \
    pip list \
    && pip install \
        --upgrade pip wheel setuptools

ARG PIP_INDEX_URL="https://download.pytorch.org/whl/test/cu128"
ARG PIP_EXTRA_INDEX_URL="https://pypi.org/simple"

RUN --mount=type=cache,target=/root/.cache/pip \
    pip install \
        --dry-run torch==2.7.1 torchvision torchaudio

RUN --mount=type=cache,target=/root/.cache/pip \
    pip install \
        --no-deps torch==2.7.1

RUN --mount=type=cache,target=/root/.cache/pip \
    pip install \
        torch==2.7.1 torchvision torchaudio

# Bind libs (.so files)
ENV LD_LIBRARY_PATH="${LD_LIBRARY_PATH}\
:/usr/local/lib64/python3.13/site-packages/torch/lib\
:/usr/local/lib/python3.13/site-packages/cusparselt/lib\
:/usr/local/lib/python3.13/site-packages/nvidia/cublas/lib\
:/usr/local/lib/python3.13/site-packages/nvidia/cuda_cupti/lib\
:/usr/local/lib/python3.13/site-packages/nvidia/cuda_nvrtc/lib\
:/usr/local/lib/python3.13/site-packages/nvidia/cuda_runtime/lib\
:/usr/local/lib/python3.13/site-packages/nvidia/cudnn/lib\
:/usr/local/lib/python3.13/site-packages/nvidia/cufft/lib\
:/usr/local/lib/python3.13/site-packages/nvidia/cufile/lib\
:/usr/local/lib/python3.13/site-packages/nvidia/curand/lib\
:/usr/local/lib/python3.13/site-packages/nvidia/cusolver/lib\
:/usr/local/lib/python3.13/site-packages/nvidia/cusparse/lib\
:/usr/local/lib/python3.13/site-packages/nvidia/nccl/lib\
:/usr/local/lib/python3.13/site-packages/nvidia/nvjitlink/lib\
:/usr/local/lib/python3.13/site-packages/nvidia/nvtx/lib"

# Handle xFormers dev version
RUN --mount=type=cache,target=/root/.cache/pip \
    pip install \
        xformers==0.0.31.dev1033

ARG PIP_INDEX_URL=
ARG PIP_EXTRA_INDEX_URL=
