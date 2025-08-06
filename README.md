## OpenWRT build of rtl_tcp with patches for low-power devices 

This is a patched version of rtlsdr-2.02 containing modifications designed for an RPI from the user `blinick`

I thought it would be worth comparing this patch with the OpenWRT version (routers are pretty low powered after all!)

Taken from [https://github.com/osmocom/rtl-sdr/pull/6/] `Perf: Replace rtl_tcp linked list with ring buffer #6` 

### Setting up the OpenWRT Buildsystem
---------------------------
First we need to setup the OpenWRT build system, these are the basic steps to build your own OpenWrt images/packages.

Install the build prerequisites: (Ubuntu 24.04)

    sudo apt update
    sudo apt install build-essential clang flex bison g++ gawk \
    gcc-multilib g++-multilib gettext git libncurses5-dev libssl-dev \
    python3-setuptools rsync swig unzip zlib1g-dev file wget

Download the sources:

    git clone https://git.openwrt.org/openwrt/openwrt.git
    cd openwrt

View the available releases:

    git branch -a

Then pick a version to checkout:

    git checkout openwrt-24.10


### replacing rtlsdr version in OpenWRT
-------------------------------------

Retrieve the software feeds:

    ./scripts/feeds update -a && ./scripts/feeds install -a

Remove rtl-sdr from `feeds/packages/utils/`

    cd feeds/packages/utils/
    rm -r rtl-sdr

Add our patched rtl-sdr in its place:
    
    git clone git@github.com:thecosmicslug/rtl_tcp_low_power_devices_openwrt.git
    mv rtl_tcp_low_power_devices_openwrt rtl-sdr

Return to the main openwrt directory:

    cd ../../../

Update Feeds again to include rtl_fm_streamer in menuconfig

    ./scripts/feeds update -a && ./scripts/feeds install -a
    
Configure the build by selecting target router and the our package at `utils/rtl-sdr`

    make defconfig
    make menuconfig

Save the configuration as `.config` to exit menuconfig.

Download needed files:

    make download

### Building rtl_tcp
-------------------------------------

Build the packages, log output to `build.log`: (Grab a coffee, this will take a while!)

    make V=s 2>&1 | tee build.log | grep -i -E "^make.*(error|[12345]...Entering dir)"


The package will be located in:

    bin/packages/<TARGET-ARCH>/packages/
