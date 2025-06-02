################################################################################
# Dockerfile that builds 'yanwk/comfyui-boot:cpu'
# A runtime environment for https://github.com/comfyanonymous/ComfyUI
# Using CPU only (not using GPU).
# The container will be running in root (easy for rootless deploy).
################################################################################

FROM docker.io/nvidia/cuda:12.8.0-runtime-ubuntu24.04

LABEL maintainer="YAN Wenkun <code@yanwk.fun>"

RUN set -eu

################################################################################
# Python and tools
# Since this image is so big, we use openSUSE-verified PIP packages for compatibility.
RUN apt-get update && \
    apt-get install -y --no-install-recommends curl tzdata ca-certificates && \
    rm -rf /var/lib/apt/lists/*

RUN apt-get update; \
    apt-get install -y --no-install-recommends \
    python312-devel \
    python312-pip \
    python312-wheel \
    python312-setuptools \
    python312-Cython \
    python312-py-build-cmake \
    python312-aiohttp \
    python312-dbm \
    python312-ffmpeg-python \
    python312-GitPython \
    python312-httpx \
    python312-joblib \
    python312-lark \
    python312-matplotlib \
    python312-mpmath \
    python312-numba-devel \
    python312-numpy1 \
    python312-onnx \
    python312-opencv \
    python312-pandas \
    python312-qrcode \
    python312-rich \
    python312-scikit-build \
    python312-scikit-build-core-pyproject \
    python312-scikit-image \
    python312-scikit-learn \
    python312-scipy \
    python312-svglib \
    python312-tqdm \
    && rm /usr/lib64/python3.12/EXTERNALLY-MANAGED \
    && update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.12 100

RUN apt-get update; \
    apt-get install -y --no-install-recommends \
        git \
        python3-dev \
        python3-pip \
        python3-venv \
        build-essential \
        cmake \
        ninja-build \
        libgtk-3-dev \
        libpng-dev \
        libjpeg-dev \
        libwebp-dev \
        libtiff5-dev \
        libopenexr-dev \
        libopenblas-dev \
        libx11-dev \
        libavutil-dev \
        libavcodec-dev \
        libavformat-dev \
        libswscale-dev \
        libswresample-dev \
        # libtbb2 \
        libssl-dev \
        libva-dev \
        libmfx-dev \
        libgstreamer1.0-dev \
        libgstreamer-plugins-base1.0-dev && \
    rm -rf /var/lib/apt/lists/*

ENV VIRTUAL_ENV=/opt/venv
RUN python3 -m venv $VIRTUAL_ENV
ENV PATH=$VIRTUAL_ENV/bin:$PATH

# hadolint ignore=DL3013
RUN python3 -m pip install --no-cache-dir --upgrade pip
RUN python3 -m pip install --no-cache-dir numpy==1.26.4


################################################################################
# Python Packages

# PyTorch (No xFormers for test-version)
# Break down the steps, so we have more but smaller image layers.
RUN --mount=type=cache,target=/root/.cache/pip \
    pip list \
    && pip install \
        --upgrade pip wheel setuptools \
    && pip install \
        --dry-run torch torchvision torchaudio \
        --index-url https://download.pytorch.org/whl/cu128 \
        --extra-index-url https://pypi.org/simple

RUN --mount=type=cache,target=/root/.cache/pip \
    pip install \
        --no-deps torch \
        --index-url https://download.pytorch.org/whl/cu128 \
        --extra-index-url https://pypi.org/simple

RUN --mount=type=cache,target=/root/.cache/pip \
    pip install \
        torch torchvision torchaudio \
        --index-url https://download.pytorch.org/whl/cu128 \
        --extra-index-url https://pypi.org/simple

# Bind libs (.so files)
ENV LD_LIBRARY_PATH="${LD_LIBRARY_PATH}\
:/usr/local/lib64/python3.12/site-packages/torch/lib\
:/usr/local/lib/python3.12/site-packages/cusparselt/lib\
:/usr/local/lib/python3.12/site-packages/nvidia/cublas/lib\
:/usr/local/lib/python3.12/site-packages/nvidia/cuda_cupti/lib\
:/usr/local/lib/python3.12/site-packages/nvidia/cuda_nvrtc/lib\
:/usr/local/lib/python3.12/site-packages/nvidia/cuda_runtime/lib\
:/usr/local/lib/python3.12/site-packages/nvidia/cudnn/lib\
:/usr/local/lib/python3.12/site-packages/nvidia/cufft/lib\
:/usr/local/lib/python3.12/site-packages/nvidia/cufile/lib\
:/usr/local/lib/python3.12/site-packages/nvidia/curand/lib\
:/usr/local/lib/python3.12/site-packages/nvidia/cusolver/lib\
:/usr/local/lib/python3.12/site-packages/nvidia/cusparse/lib\
:/usr/local/lib/python3.12/site-packages/nvidia/nccl/lib\
:/usr/local/lib/python3.12/site-packages/nvidia/nvjitlink/lib\
:/usr/local/lib/python3.12/site-packages/nvidia/nvtx/lib"

# Deps for ComfyUI & custom nodes
COPY builder-scripts/.  /builder-scripts/

RUN --mount=type=cache,target=/root/.cache/pip \
    pip install \
        -r /builder-scripts/pak3.txt

RUN --mount=type=cache,target=/root/.cache/pip \
    pip install \
        -r /builder-scripts/pak5.txt

RUN --mount=type=cache,target=/root/.cache/pip \
    pip install \
        -r /builder-scripts/pak7.txt

# Make sure the deps fit the needs for ComfyUI & Manager
RUN --mount=type=cache,target=/root/.cache/pip \
    pip install \
        -r https://github.com/comfyanonymous/ComfyUI/raw/refs/heads/master/requirements.txt \
        -r https://github.com/ltdrdata/ComfyUI-Manager/raw/refs/heads/main/requirements.txt \
    && pip list

################################################################################

RUN du -ah /root \
    && rm -rf /root/* \
    && rm -rf /root/.[^.]* /root/.??*

COPY runner-scripts/.  /runner-scripts/

USER root
VOLUME /root
WORKDIR /root
EXPOSE 8188
ENV CLI_ARGS=""
CMD ["bash","/runner-scripts/entrypoint.sh"]
