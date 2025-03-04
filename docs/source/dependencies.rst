1. Environment and dependencies installation
============================================

As a virtual environment an Ubuntu 24.04.01 LTS server was used, running in VMware. For VMware we recommend the community edition. Once the usual installation of the virtual machine is completed, proceed with the usual: 

.. code-block:: bash

    sudo apt update && sudo apt upgrade

Afterwards we need to install git:

.. code-block:: bash

    sudo apt install git

and create a directory where to pull locally and compile required libraries and dependencies:

.. code-block:: bash

    mkdir <your-path>/libraries


1.2 Trusted Platform Module dependencies
````````````````````````````````````````

1.2.1 TPM2-TSS library installation
+++++++++++++++++++++++++++++++++++

To install the Trusted Platform Module dependencies, we will use the `tpm2-tss <https://github.com/tpm2-software/tpm2-tss/blob/master/INSTALL.md>`_ install guide.

First, cd to your working libraries directory

.. code-block:: bash

    cd <your-path>/libraries

Install dependencies for the *tpm2-tss* library:

.. code-block:: bash

    sudo apt -y install autoconf-archive libcmocka0 libcmocka-dev procps iproute2 build-essential git pkg-config   gcc   libtool automake libssl-dev uthash-dev autoconf doxygen libjson-c-dev libini-config-dev libcurl4-openssl-dev uuid-dev   libltdl-dev libusb-1.0-0-dev libftdi-dev

Clone the git project:

.. code-block:: bash

    git clone https://github.com/tpm2-software/tpm2-tss.git

Cd into the project directory

.. code-block:: bash

    cd tpm2-tss

And proceed with the project configuration and compilation

.. code-block:: bash

    ./bootstrap

.. code-block:: bash

    ./configure

.. code-block:: bash

    make -j$(nproc)

.. code-block:: bash

    sudo make install

1.2.2 TPM2-TOOLS installation
+++++++++++++++++++++++++++++

TPM2-TOOLS is a set of utilities that can be usefull to test and interact with the TPM from a command line. The installation process is similar with the previous one.

Be sure you are in the libraries directory:

.. code-block:: bash

    cd <your-path>/libraries

Clone the project:

.. code-block:: bash

    git clone https://github.com/tpm2-software/tpm2-tools.git

Cd into the project directory:

.. code-block:: bash

    cd tpm2-tools

Configure and compile with the following sequence of commands:

.. code-block:: bash

    ./bootstrap

.. code-block:: bash

    ./configure

.. code-block:: bash

    make -j$(nproc)

.. code-block:: bash

    sudo make install


1.2.3 TPM2-ABRMD installation
+++++++++++++++++++++++++++++


1.3 Virtual Trusted Platform Module instalation
````````````````````````````````````````````````

1.4 MQTT Broker
```````````````
MQTT is used for internal and external communication to and from the Secure Logging Service. As a message broker, we will use Eclipse's mosquitto with the paho python client:

Install mosquitto:

.. code-block:: bash

        sudo apt install mosquitto

Enable the mosquitto service via systemctl:

.. code-block:: bash

    sudo systemctl enable mosquitto

Check status:

.. code-block:: bash

    sudo systemctl status mosquitto

If needed, restart the service:

.. code-block:: bash

    sudo systemctl restart mosquitto

Now we can install the paho python3 module:

.. code-block:: bash

    pip3 install paho-mqtt --break-system-packages

The argument *--break-system-packages* is used to install the module system-wide. For testing the MQTT communication, it is usefull to have the mosquitto-clients installed as well, as it allows us to test PUB/SUB messaging

.. code-block:: bash
  
    sudo apt install mosquitto-clients




1.5 Virtual Controller Area Network configuration
`````````````````````````````````````````````````

To setup a virtual Controller Area Network interface we will create a service that starts on system boot and keeps the vcan interface alive. To do this, first go in your *libraries* directory and clone the repository below:

Go to your libraries/dependencies directory:

.. code-block:: bash
  
    cd libraries

Clone the repository:

.. code-block:: bash

    git clone https://github.com/terilenard/vcan-setup.git

This repository has the neccesary scripts and configs to setup the virtual CAN interface. Prior to running the *vcan.sh* script, we also need to install *can-utils*:

.. code-block:: bash
    sudo apt install can-utils

Afterwards, got to the *vcan-setup* directory:

.. code-block:: bash

    cd vcan-setup

And run:

.. code-block:: bash

    sudo ./vcan.sh

After running the script, a new interface should be visible named vcan0 in the output of the following command:

.. code-block:: bash

    ip link

