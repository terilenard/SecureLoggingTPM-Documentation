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

TPM2-ABRMD is a Access Broker and Resource Manager daemon is a service that controls the access and manages the resources loaded and de-loaded from a Trusted Platform Module. To install and setup TPM2-ABRMD, be sure you are in your libraries directory:

.. code-block:: bash

    cd <your-path>/libraries

Clone the project:

.. code-block:: bash

    git clone https://github.com/tpm2-software/tpm2-abrmd.git

Cd into the project directory:

.. code-block:: bash

    cd tpm2-abrmd

Install possible missing dependencies:

.. code-block:: bash

    sudo apt install libglib2.0-dev

And proceed to configure and build the project

.. code-block:: bash

    ./bootstrap

.. code-block:: bash

    ./configure --with-dbuspolicydir=/etc/dbus-1/system.d --with-udevrulesdir=/usr/lib/udev/rules.d --with-systemdsystemunitdir=/usr/lib/systemd/system --libdir=/usr/lib64 --prefix=/usr

.. code-block:: bash

    make -j$(nproc)

.. code-block:: bash

    sudo make install

Afterwards enable the *tpm2-abrmd* service and restart it:

.. code-block:: bash

    sudo systemctl enable tpm2-abrmd

.. code-block:: bash

    sudo systemctl restart tpm2-abrmd

You can check the service status with:

.. code-block:: bash

    sudo systemctl status tpm2-abrmd


1.3 Virtual Trusted Platform Module instalation
````````````````````````````````````````````````

Now that we have installed our TPM dependencies, we need to setup the virtual TPM, and configure the tpm2-abrmd to connect to it. For the virtual TPM we will use th `IBM's TPM 2.0 TSS <https://sourceforge.net/projects/ibmtpm20tss/>`_ project.

Go to your libraries directory:

.. code-block:: bash

    cd <your-path>/libraries

Download the latest version of the project:

.. code-block:: bash

    wget https://sourceforge.net/projects/ibmswtpm2/files/latest/download -O ibmtpm.tar.gz

Create a directory where to extract the archive

.. code-block:: bash

    mkdir ibmtpm

Untar the archive:

.. code-block:: bash

    tar -zxvf ibmtpm.tar.gz ./ibmtpm/

Go to the source directory of the project

.. code-block:: bash

    cd ibmtpm/src/

And compile

.. code-block:: bash

    make -j$(nproc)

Now, we will create a service that will start on system boot and execute the virtual TPM. To do this, we create first a *.service* file in */etc/systemd/system/*.

Cd to */etc/systemd/system/*:

.. code-block:: bash

    cd /etc/systemd/system/

Create and open a new file named *ibmtss.service*:

.. code-block:: bash

    sudo nano ibmtss.service

And paste the following content into *ibmtss.service*

.. code-block:: bash

    [Unit]
    Description=IBM Virtual TPM2.0
    Before=tpm2-abrmd.service
    
    [Service]
    ExecStart=<your-path>/ibmtpm/src/tpm_server
    
    [Install]
    WantedBy=multi-user.target

Note that *ExecStart* should point to the executable *tpm_server* that was generated through compiling the IBM virtual TPM source code.

After run:

.. code-block:: bash

    sudo systemctl daemon-reload

To enable the service we must run:

.. code-block:: bash

    sudo systemctl enable ibmtss

And now restart:

.. code-block:: bash

    sudo systemctl restart ibmtss

Lastly, we have to modify the *.service* of the *tpm2-abrmd* to start after *ibmtss* and to connect to it, instead of physical TPM.

Open to edit *tpm2-abrmd.service*:

.. code-block:: bash

    sudo nano /usr/local/lib/systemd/system/tpm2-abrmd.service

Modify the file to look like this:

.. code-block:: bash

    [Unit]
    #Description=TPM2 Access Broker and Resource Management Daemon
    # These settings are needed when using the device TCTI. If the
    # TCP mssim is used then the settings should be commented out.
    #After=dev-tpm0.device
    After=ibmtss.service
    Requires=dev-tpm0.device
    
    [Service]
    Type=dbus
    BusName=com.intel.tss2.Tabrmd
    ExecStart=/usr/local/sbin/tpm2-abrmd --tcti=mssim:host=localhost,port=2321
    User=tss
    
    [Install]
    WantedBy=multi-user.target

After run:

.. code-block:: bash

    sudo systemctl daemon-reload

And restart the service:

.. code-block:: bash

    sudo systemctl restart tpm2-abrmd

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

