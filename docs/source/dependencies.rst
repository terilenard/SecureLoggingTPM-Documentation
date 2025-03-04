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

    mkdir libraries


1.2 Trusted Platform Module dependencies
````````````````````````````````````````

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
