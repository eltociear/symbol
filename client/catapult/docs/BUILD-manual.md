# Building manually

This guide should mostly work for Linux and OS X operating systems.
It does not include instructions for Windows.
For Windows, [Build with CONAN](BUILD-conan.md) is strongly encouraged.

## Prerequisites

- ``git``.
- ``python`` 3.7+.
- ``openssl`` 3.0.8+.
- About 15 GB of free disk space.

These instructions have been verified to work on Ubuntu 20.04 with 8 GB of RAM and 4 CPU cores.
On an Ubuntu system, all prerequisites can be installed with the following commands:

```sh
apt update
apt -y upgrade
apt -y install git gcc g++ cmake curl libssl-dev libgtest-dev ninja-build pkg-config python3-pip
```

## Step 1: Clone catapult-client

As usual, clone the git repository:

```sh
git clone https://github.com/symbol/symbol.git
cd symbol/client/catapult
```

## Step 2: Download, build and install all dependencies from source

Type this into a terminal:

> **NOTE**:
> If you want to build with clang, add the `--use-clang` flag.

```sh
PYTHONPATH="../../jenkins/catapult/" python3 "../../jenkins/catapult/installDepsLocal.py" \
	--target "./_deps" \
	--versions "../../jenkins/catapult/versions.properties" \
	--download \
	--build
```

## Step 3: Build catapult

To create or update the ``_build`` directory type into a terminal:

> **NOTE**:
> When building with clang, add `-DENABLE_FUZZ_BUILD=ON` to build with UBSAN and ASAN (recommended for development).

```sh
mkdir -p _build && cd _build
BOOST_ROOT="$(realpath ../_deps/boost)" cmake .. \
	-DCMAKE_BUILD_TYPE=RelWithDebInfo \
	-DCMAKE_PREFIX_PATH="$(realpath ../_deps/facebook);$(realpath ../_deps/google);$(realpath ../_deps/mongodb);$(realpath ../_deps/zeromq)" \
	-GNinja
ninja publish
ninja
```

Once the build finishes successfully, the tools in ``_build/bin`` are ready to use. However, the dependencies in ``_deps/boost/lib`` and ``_build/lib`` must be accessible so make sure to add these folders to the ``LD_LIBRARY_PATH`` environment variable.

One way of doing this is by running this from the ``symbol/client/catapult`` directory:

  ```sh
  export LD_LIBRARY_PATH=$PWD/_deps/boost/lib:$PWD/_build/lib
  ```

You will need to run this line every new session, unless you add it at the end of your ``~/.bashrc`` or ``~/.profile`` files.

## Step 4: Verification

Verify that the tools are working correctly by running (from ``_build`` directory):

```sh
./bin/catapult.tools.address --help
```
