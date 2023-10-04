# MinimalDPDKExamples
Minimal examples of DPDK

Original Project Lead: Thomas Edwards (thomas.edwards@disney.com)

This project was modified to send only through one port and also receive only through one port

## minimal_tx
Minimal code to send UDP packets with DPDK

## minimal_rx
Minimal code to receive packets with DPDK

Loops and prints hex dump of received packets on all DPDK interfaces

## Building
Requires DPDK installation and RTE_SDK environment variable set properly for Makefiles.

## Local Test using 2 ports and cross connection
DPDK_local_tutorial.md (https://github.com/rotrida/MinimalDPDKExamples/blob/master/DPDK_local_tutorial.md)

## AWS Testing
These programs have been tested on AWS EC2 c5.9xlarge with RHEL AMI and added ENA interfaces.  See [DPDK on EC2 Tutorial](https://github.com/FOXNEOAdvancedTechnology/MinimalDPDKExamples/blob/master/DPDK_EC2_Tutorial.md) to rapidly install DPDK on AWS EC2, and to add ENA interfaces with DPDK drivers.