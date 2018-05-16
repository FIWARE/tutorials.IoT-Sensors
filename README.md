![FIWARE Banner](https://fiware.github.io/tutorials.Subscriptions/img/fiware.png)

[![NGSI v2](https://img.shields.io/badge/Ultralight-2.0-pink.svg)](http://fiware-iotagent-ul.readthedocs.io/en/latest/usermanual/index.html#user-programmers-manual)

This tutorial is an introduction to IoT devices and the usage of the UltraLight 2.O Protocol for constrained
devices. The tutorial introduces a series of dummy IoT devices  which are displayed within the browser and
allows a user to interact with them. A complete understanding of all the terms and concepts defined in this
tutorial is necessary before proceeding to connect the IoT devices to the Orion Context Broker via an IoT Agent. 

The tutorial uses [cUrl](https://ec.haxx.se/) commands throughout, but is also available as [Postman documentation](http://fiware.github.io/tutorials.Getting-Started/)

[![Run in Postman](https://run.pstmn.io/button.svg)](https://app.getpostman.com/run-collection/d6671a59a7e892629d2b)

# Contents

TBD...


# What are IoT devices?

The [Internet of Things](https://www.linux.com/news/who-needs-internet-things) (IoT) is a network of physical
devices which are able to connect to a network and exchange data. Each "thing"  or "smart device" is a gadget
with embedded electronics and sofware which can act as a sensor or actuator. Sensors are able to report the
state of the real-world around them. Actuators are responsible for altering the state of the system, by responding
to a control signal.

Each device is uniquely identifiable through its embedded computing system but is able to inter-operate within the
existing Internet infrastructure.

FIWARE is a system for managing context information. For a smart solution based on the Internet of Things,
the context is provided by the array of attached IoT devices. Since each IoT device is a physical object which
exists in the real world, it will eventually be represented as a unique entity within the context.

IoT devices can range from simple to complex. Here are some examples of IoT devices which will be used within this tutorial:

* A **Smart Door** is an electronic door which can be sent commands to be locked or unlocked remotely.
  It can also report on its current state (`OPEN`, `CLOSED` or `LOCKED`), 
* A **Bell** can be sent a command to activate and ring for a short period
* A **Motion Sensor** can be queried to return the number of people who have passed by since it was last reset
* A **Smart Lamp** can be switched on or off remotely. It can also report on its current state (`ON` or `OFF`).
  When switched on, a motion sensor within the device checks to see if light is is needed and will dim if no-one is nearby.
  Furthermore the device can be report on the current luminocity of the bulb.

As you can see, the **Bell** is an example of a pure actuator, as it only reacts to the given commands. Meanwhile the 
**Motion Sensor** is an example of a pure sensor, since it will only report on the state of the world as it sees it.
The other two devices are able to both respond to commands and report on state in a meaningful way.

The state information held within each device, as it will eventually be seen within the Context Broker is defined in in the diagram below:

![](https://fiware.github.io/tutorials.IoT-Sensors/img/entities.png)


#### Device Monitor

For the purpose of this tutorial, a series of dummy IoT devices have been created, which will eventually be attached to the context broker.
The state of each device can be seen on the UltraLight device monitor web-page found at: `http://localhost:3000/device/monitor`

![FIWARE Monitor](https://fiware.github.io/tutorials.IoT-Sensors/img/device-monitor.png)


# Architecture


The demo application will only make use of a single custom component acting as a message broker for connected devices.
Since all interactions are initiated by HTTP requests, the entities can be containerized and run from exposed ports. 

![](https://fiware.github.io/tutorials.IoT-Sensors/img/architecture.png)


# Prerequisites

## Docker

To keep things simple both components will be run using [Docker](https://www.docker.com). **Docker** is a container technology
which allows to different components isolated into their respective environments. 

* To install Docker on Windows follow the instructions [here](https://docs.docker.com/docker-for-windows/)
* To install Docker on Mac follow the instructions [here](https://docs.docker.com/docker-for-mac/)
* To install Docker on Linux follow the instructions [here](https://docs.docker.com/install/)

**Docker Compose** is a tool for defining and running multi-container Docker applications. A 
[YAML file](https://raw.githubusercontent.com/Fiware/tutorials.Entity-Relationships/master/docker-compose.yml) is used
configure the required services for the application. This means all container sevices can be brought up in a single 
commmand. Docker Compose is installed by default  as part of Docker for Windows and  Docker for Mac, however Linux users 
will need to follow the instructions found  [here](https://docs.docker.com/compose/install/)

## Cygwin 

We will start up our services using a simple bash script. Windows users should download [cygwin](http://www.cygwin.com/) to provide a
command line functionality similar to a Linux distribution on Windows. 

# Start Up

All services can be initialised from the command line by running the bash script provided within the repository:

```console
./services create; ./services start;
```

This command will also import seed data from the previous [Stock Management example](https://github.com/Fiware/tutorials.Context-Providers) on startup.

>:information_source: **Note:** If you want to clean up and start over again you can do so with the following command:
>
>```console
>./services stop
>``` 
>

#  Ultralight

To follow the tutorial correctly please ensure you have the follow pages available on tabs in your browser before you enter any cUrl commands.

#### Device Monitor

The device monitor can be found at: `http://localhost:3000/device/monitor`




