# Tensorflow_pine64

## Goal
This repositery aims at giving you directions and how to compile Tensorflow from source on the pine64. The pine64 is a single board computer (sbc) with an AARCH64 architecture. This might extend to any board with this architecture but I can't confirm.

I am running Ubuntu 16_04.

## Create SWAP

The first thing that we'll do is create a SWAP file. This may not be needed if you have a board with at least 4Go of RAM. In my case, I only have 2, and it was a problem at some point.

Check to see if you already have swap enabled:
   
    sudo swapon --show

If the result is empty, then you don't have a SWAP file and can proceed with the following command. Note that it will create a 2Go swapfile. Replace by how much you want (the same size or double the size of the RAM is usually good).

    sudo fallocate -l 2G /swapfile

We then change the permission to only root user

    sudo chmod 600 /swapfile

Then use *mkswap* to actually create swap space on the file

    sudo mkswap /swapfile

Activate the swap file

    sudo swapon /swapfile

Open */etc/fstab* to make the change permanent

    sudo nano /etc/fstab

and paste the following at the end of the file

    /swapfile   swap    swap    defaults    0   0

Verify that it's working

    sudo swapon --show

Something like this should appear

    NAME       TYPE        SIZE   USED PRIO
    /swapfile  file          4G   1.7G   -1

## Install OpenJDK 8

Ok, now that we have some swap ready, it's time to isntall the OpenJDK 8. You can use any JDK you want, however, I only tested with this one.

    wget ...
    tar -xvf jdk8u-server-release-1708.tar.xz
    cd jdk8u-server-release-1708
    export JAVA_HOME=$PWD
    cd jdk8u-server-release-1708/bin
    export PATH=$PWD:$PATH

## Install Python dependencies

You should have python install on your computer. I recommend using virtual environment. You'll need to have **numpy** installed, **dev**, **pip** and **wheel**

Install Numpy with

    pip install numpy

If you have a virtual environment, simply activate the **global** environment and then install numpy

Install the other dependencies

    sudo apt-get update
    sudo apt-get install python-numpy python-dev python-pip python-wheel

## Build Bazel

It's time to build Bazel! It's used to build Tensorflow later on.

Install the pre-requisites for Bazel

    sudo apt-get isntall pkg-config zip g++ zliblg-dev unzip

Grab the bazel version that you want by replacing \<version> in the following command

    wget ...
    mkdir bazel-<version>
    unzip bazel-<version>-dist.zip -d bazel-<version>
    cd bazel-<version>

If you use a fairly new bazel version, no file should be modified. (I tested back to bazel 0.18)

Use the following command to export optionnal parameters for the JVM and compile bazel. This is to avoid the error of java not having enough heap space.

    env BAZEL_JAVAC_OPTS="-J-Xms512m -J-Xmx1024m" ./compile.sh

If you still have the java heap space error, increase the value, for example from 512 to 1024. But make sure you have enough RAM and SWAP!

Once bazel is compiled you'll have a message telling you the location of the binary. Copy that binary to /usr/local/bin/

    sudo cp /output/bazel /usr/local/bin/

## Build Tensorflow

Finally, we can build Tensorflow!

    git clone https://github.com/tensorflow/tensorflow.git

If you don't change branch, by default you're in the master branch, so it should be the latest stable release. Though, if you want to build a specific version, the change branch with

    git checkout rX.X

replace X.X with the version you want.

You'll now configure your build, say no to everything but **jemalloc**

    ./configure

This should go without any error. You can now finally launch the actual tensorflow build. If you are using SSH to do that you'll want to use the **nohup** command, that will detach the process from your ssh session. This is crucial because if you lose the connection the build will stop !

    nohup bazel build -c opt --copt="-funsafe-math-optimizations" --copt="-ftree-vectorize" --copt="-fomit-frame-pointer" --verbose_failures tensorflow/tools/pip_package:build_pip_package &

This will launch the building process and put it in the background. You'll see something like this 

    [1] 10736
    user@pine64:~/tensorflow$ nohup: ignoring input and appending output to 'nohup.out'

Simply press enter and you have the hand again. If   you want to monitor the output of the building process simply do

    tail -f nohup.out

And you'll see the output of the building process



