---
title: 'TideSense Development Environment Setup: Docker, Podman and Ubuntu'
description: Prepare the TideSense build environment using Docker or Podman, or install Ubuntu 24.04 dependencies, ROS 2, Boost headers, Node.js and TideConfigurator tools directly.
---

# Development Environment Setup

The following two approaches provide the build dependencies required by the project. After obtaining the source code, run the commands from the repository root.

## Option 1: Use Docker or Podman

Docker:

```bash
docker pull ghcr.io/humpbacklab/tidesense-dev:ubuntu24.04
SDK_ROOT=$(pwd -P)
docker run --rm -it --privileged --network=host \
  -v "$SDK_ROOT:$SDK_ROOT" -w "$SDK_ROOT" \
  -v tidesense-calibration:/var/lib/containers/storage \
  ghcr.io/humpbacklab/tidesense-dev:ubuntu24.04 bash
```

Podman:

```bash
podman pull ghcr.io/humpbacklab/tidesense-dev:ubuntu24.04
TIDESENSE_DEV_IMAGE=ghcr.io/humpbacklab/tidesense-dev:ubuntu24.04 ./scripts/dev-container.sh shell
```

## Option 2: Install Dependencies Directly (Ubuntu 24.04 x86_64)

Install the same system tools used by the [SDK base image recipe](https://github.com/HumpbackLab/tidesense/blob/master/scripts/dev-image/Containerfile.base):

```bash
sudo apt update
sudo apt install -y --no-install-recommends \
  autoconf automake bc binutils bison build-essential bzip2 ccache cmake \
  cpio device-tree-compiler dosfstools e2fsprogs file flex gawk gettext \
  git git-lfs libelf-dev libncurses-dev libssl-dev libtool libudev-dev \
  libusb-1.0-0-dev libxml2-dev lz4 mtools ninja-build openssh-client \
  patch pkg-config python3 python3-dev python3-pip python3-setuptools \
  python3-venv python3-yaml python3-jinja2 python3-colorama \
  python-is-python3 rsync sshpass swig tar texinfo unzip wget xz-utils zip \
  zlib1g-dev kmod libpcre3 squashfs-tools podman catatonit xxd ca-certificates curl gnupg
```

Add the ROS 2 package repository and install the development packages for your chosen distribution. The SDK does not require a specific ROS 2 distribution:

```bash
sudo install -d -m 0755 /usr/share/keyrings
curl -fsSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key | \
  sudo tee /usr/share/keyrings/ros-archive-keyring.gpg > /dev/null
echo 'deb [arch=amd64 signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu noble main' | \
  sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null
sudo apt update
ROS_DISTRO=jazzy  # Change to another ROS 2 distribution available for Ubuntu 24.04 if needed
sudo apt install -y --no-install-recommends \
  "ros-$ROS_DISTRO-ros-base" "ros-$ROS_DISTRO-rmw-cyclonedds-cpp" ros-dev-tools
export TIDESENSE_ROS_SETUP="/opt/ros/$ROS_DISTRO/setup.bash"
```

SchurVINS uses Boost 1.74 headers. On Ubuntu 24.04, extract them separately as the development image does:

```bash
BOOST_DIR="$HOME/.local/share/tidesense-boost-1.74"
mkdir -p "$BOOST_DIR"
curl -fsSL https://archive.ubuntu.com/ubuntu/pool/main/b/boost1.74/libboost1.74-dev_1.74.0-14ubuntu3_amd64.deb \
  -o "$BOOST_DIR/libboost1.74-dev.deb"
dpkg-deb --fsys-tarfile "$BOOST_DIR/libboost1.74-dev.deb" | \
  tar --no-same-owner --no-same-permissions --no-overwrite-dir -xf - -C "$BOOST_DIR"
export BOOST_HEADERS="$BOOST_DIR/usr/include/boost"
```

Install [Node.js 22](https://nodejs.org/en/download/archive/v22) using the version from the development image, then install TideConfigurator's Python and frontend dependencies:

```bash
NODE_VERSION=v22.23.4
NODE_DIR="$HOME/.local/opt/node-$NODE_VERSION"
mkdir -p "$NODE_DIR"
curl -fsSL "https://nodejs.org/dist/$NODE_VERSION/node-$NODE_VERSION-linux-x64.tar.xz" | \
  tar -xJ --strip-components=1 -C "$NODE_DIR"
export PATH="$NODE_DIR/bin:$PATH"
python3 -m venv --system-site-packages tide_configurator/.venv
source tide_configurator/.venv/bin/activate
python3 -m pip install pycryptodome -r tide_configurator/requirements.txt
npm --prefix tide_configurator/frontend ci
npm --prefix tide_configurator/frontend run build
```

When installing dependencies directly, make sure `TIDESENSE_ROS_SETUP`, `BOOST_HEADERS` and the Node.js path remain set in the current shell before building. After completing either approach, follow the same commands in the build section of the [main README](https://github.com/HumpbackLab/tidesense#readme). Follow each subproject's README for its own instructions.
