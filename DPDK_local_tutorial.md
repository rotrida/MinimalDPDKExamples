# DPDK on local machine (6 cores and 2 compatible NICs)

## Follow F-Stack install process

    # clone F-Stack
    mkdir -p /data/f-stack
    git clone https://github.com/F-Stack/f-stack.git /data/f-stack

    # Install libnuma-dev
    yum install numactl-devel          # on Centos
    #sudo apt-get install libnuma-dev  # on Ubuntu

    pip3 install pyelftools --upgrade
    # Install python and modules for running DPDK python scripts
    pip3 install pyelftools --upgrade # RedHat/Centos
    sudo apt install python # On ubuntu
    #sudo pkg install python # On FreeBSD

    # Install dependencies (FreeBSD only)
    #pkg install meson pkgconf py38-pyelftools

    cd f-stack
    # Compile DPDK
    cd dpdk/
    # re-enable kni now, to remove kni later
    meson -Denable_kmods=true -Ddisable_libs=flow_classify build
    ninja -C build
    ninja -C build install

    # Set hugepage (Linux only)
    # single-node system
    echo 1024 > /sys/kernel/mm/hugepages/hugepages-2048kB/nr_hugepages

    # or NUMA (Linux only)
    echo 1024 > /sys/devices/system/node/node0/hugepages/hugepages-2048kB/nr_hugepages
    echo 1024 > /sys/devices/system/node/node1/hugepages/hugepages-2048kB/nr_hugepages

    # Using Hugepage with the DPDK (Linux only)
    mkdir /mnt/huge
    mount -t hugetlbfs nodev /mnt/huge

    # Close ASLR; it is necessary in multiple process (Linux only)
    echo 0 > /proc/sys/kernel/randomize_va_space

    # Offload NIC
    # For Linux:
    modprobe uio
    insmod /data/f-stack/dpdk/build/kernel/linux/igb_uio/igb_uio.ko
    insmod /data/f-stack/dpdk/build/kernel/linux/kni/rte_kni.ko carrier=on # carrier=on is necessary, otherwise need to be up `veth0` via `echo 1 > /sys/class/net/veth0/carrier`
    python dpdk-devbind.py --status
    ifconfig eth0 down
    python dpdk-devbind.py --bind=igb_uio eth0 # assuming that use 10GE NIC and eth0

    # For FreeBSD:
    # Refer DPDK FreeBSD guide to set tunables in /boot/loader.conf
    # Below is an example used for our testing machine
    #echo "hw.nic_uio.bdfs=\"2:0:0\"" >> /boot/loader.conf
    #echo "hw.contigmem.num_buffers=1" >> /boot/loader.conf
    #echo "hw.contigmem.buffer_size=1073741824" >> /boot/loader.conf
    #kldload contigmem
    #kldload nic_uio

    # On Ubuntu, use gawk instead of the default mawk.
    #sudo apt-get install gawk  # or execute `sudo update-alternatives --config awk` to choose gawk.

    # Install dependencies for F-Stack
    sudo apt install gcc make libssl-dev                            # On ubuntu
    #sudo pkg install gcc gmake openssl pkgconf libepoll-shim       # On FreeBSD

    # Upgrade pkg-config while version < 0.28
    #cd /data
    #wget https://pkg-config.freedesktop.org/releases/pkg-config-0.29.2.tar.gz
    #tar xzvf pkg-config-0.29.2.tar.gz
    #cd pkg-config-0.29.2
    #./configure --with-internal-glib
    #make
    #make install
    #mv /usr/bin/pkg-config /usr/bin/pkg-config.bak
    #ln -s /usr/local/bin/pkg-config /usr/bin/pkg-config
 
    # Compile F-Stack
    export FF_PATH=/data/f-stack
    export PKG_CONFIG_PATH=/usr/lib64/pkgconfig:/usr/local/lib64/pkgconfig:/usr/lib/pkgconfig
    cd /data/f-stack/lib/
    make    # On Linux
    #gmake   # On FreeBSD

    # Install F-STACK
    # libfstack.a will be installed to /usr/local/lib
    # ff_*.h will be installed to /usr/local/include
    # start.sh will be installed to /usr/local/bin/ff_start
    # config.ini will be installed to /etc/f-stack.conf
    make install    # On Linux
    #gmake install   # On FreeBSD

    # Build minimal_tx and minimal_rx with make
    cd minimal_tx
    make

    cd minimal_rx
    make

    #execute on the same machine. Be careful with the cores
    sudo build/minimal_tx -l 3-5 -n 4 --proc-type auto -- -m 98:b7:85:01:e0:6e -s 10.0.0.1 -d 10.0.0.2
    sudo build/minimal_rx -l 0-2 -n 4 --proc-type auto