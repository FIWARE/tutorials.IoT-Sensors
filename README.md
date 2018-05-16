![FIWARE Banner](https://fiware.github.io/tutorials.IoT-Sensors//img/fiware.png)

[![NGSI v2](https://img.shields.io/badge/Ultralight-2.0-pink.svg)](http://fiware-iotagent-ul.readthedocs.io/en/latest/usermanual/index.html#user-programmers-manual)

This tutorial is an introduction to IoT devices and the usage of the 
[UltraLight 2.0](http://fiware-iotagent-ul.readthedocs.io/en/latest/usermanual/index.html#user-programmers-manual) Protocol for 
constrained devices. The tutorial introduces a series of dummy IoT devices  which are displayed within the browser and
allows a user to interact with them. A complete understanding of all the terms and concepts defined in this
tutorial is necessary before proceeding to connect the IoT devices to the Orion Context Broker via an IoT Agent. 

The tutorial uses [cUrl](https://ec.haxx.se/) commands throughout, but is also available as [Postman documentation](http://fiware.github.io/tutorials.Getting-Started/)

[![Run in Postman](https://run.pstmn.io/button.svg)](https://app.getpostman.com/run-collection/d6671a59a7e892629d2b)

# Contents

- [What are IoT devices?](#what-are-iot-devices)
- [Architecture](#architecture)
- [Prerequisites](#prerequisites)
  * [Docker](#docker)
  * [Cygwin](#cygwin)
- [Start Up](#start-up)
- [Ultralight](#ultralight)
  * [What is Ultralight 2.0?](#what-is-ultralight-20)
  * [Southbound Traffic (Commands)](#southbound-traffic-commands)
  * [Northbound Traffic (Measurements)](#northbound-traffic-measurements)
    + [Measurement using HTTP GET](#measurement-using-http-get)
    + [Measurement using HTTP POST](#measurement-using-http-post)


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

The demo application will only make use of a single custom component acting as an IoT message broker for connected devices.
The IoT message broker will be using the [UltraLight 2.0](http://fiware-iotagent-ul.readthedocs.io/en/latest/usermanual/index.html#user-programmers-manual) protocol running over HTTP.
Since all interactions are initiated by HTTP requests, the entities can be containerized and run from exposed ports. 

![](https://fiware.github.io/tutorials.IoT-Sensors/img/architecture.png)

When describing the messages being passed through a working smart solution we will refer to two further components which
are not used in this tutorial, but will be needed to complete the system subsequently.

* The Orion Context Broker server is used for holding the context data of the smart solution. As you know all 
  interactions with the context broker must be made using [NGSI](https://swagger.lab.fiware.org/?url=https://raw.githubusercontent.com/Fiware/specifications/master/OpenAPI/ngsiv2/ngsiv2-openapi.json)
* An IoT Agent acts as a middleware component converting [NGSI](https://swagger.lab.fiware.org/?url=https://raw.githubusercontent.com/Fiware/specifications/master/OpenAPI/ngsiv2/ngsiv2-openapi.json) 
  requests (from the context broker) into a protocol 
  (such as [UltraLight 2.0](http://fiware-iotagent-ul.readthedocs.io/en/latest/usermanual/index.html#user-programmers-manual))
  usable by the IoT devices themselves.

It is therefore necessary to understand a sample device protocol first, and comprehend how messages pass 
through an IoT message broker to subsequently understand the purpose of the IoT agent middleware.

# Prerequisites

## Docker

To keep things simple all components will be run using [Docker](https://www.docker.com). **Docker** is a container technology
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

To follow the tutorial correctly please ensure you have the device monitor page available in your browser before you enter any cUrl commands. The device monitor displays the current state of an array of dummy devices using Ultralight 2.0 syntax

#### Device Monitor

The device monitor can be found at: `http://localhost:3000/device/monitor`

## What is Ultralight 2.0?

Ultralight 2.0 is a lightweight text based protocol for constrained devices and communications where
bandwidth and device memory resources are limited. The payload for information update requests is
composed of a list of key-value pairs separated by the pipe `|` character. 

For example a payload such as:

```
t|15|k|abc
```

Contains two attributes, one named "t" with value "15" and another named "k" with value "abc" are transmitted. 
Values in Ultralight 2.0 are not typed (everything is treated as a string).

Ultralight 2.0 defines a payload describing measures and commands to share between devices and servers but, 
does not specify a single transport protocol. Instead, different transport protocol bindings (such as  HTTP,
MQTT and AMQP) can be used for different scenarios. For this tutorial we will be using HTTP as a transport protocol.




## Southbound Traffic (Commands)

HTTP requests generated by the from the Context Broker and passed downwards towards an IoT device (via
an IoT agent) is known as southbound traffic. Southbound traffic consists of **commands** made to 
actuator devices which alter the state of the real world by their actions. For example a command
to alter the state of a lamp to `ON` would switch on the lamp in real life. This in turn could alter
the readings of other sensors nearby.


## Northbound Traffic (Measurements)

Requests generated from an IoT device and passed back upwards towards the Context Broker (via an 
IoT agent) is known as northbound traffic. Northbound traffic consists of **measurements** made
by sensor devices and relays the state of the real world into the context data of the system.
For example a measurement from a humidity sensor could be relayed back into the context broker
to indicate that the moisture level of the entity has changed. A subscription could be made
to be informed of such changes and there provoke further actions (such as turning on a sprinkler) 


###  Measurement using HTTP GET 


A device can report new measures to the IoT Platform using an HTTP GET request to a "well-known" endpoint
(the path `/iot/d`) along with the following query parameters:

* `i` (device ID): Device ID (unique for the API Key).
* `k` (API Key): API Key for the service the device is registered on.
* `t` (timestamp): Timestamp of the measure. Will override the automatic IoTAgent timestamp (optional).
* `d` (Data): Ultralight 2.0 payload.

The `i` and `k`  parameters are mandatory.

For example the request:

```
<iot-agent>/iot/d?i=motion001&c=12
```

Would indicate that the device `id=motion001` wishes to inform the IoT Agent  that is  has made a real-world measurement `c` with
the value `12`. This would eventually be passed up

###  Measurement using HTTP POST 

HTTP POST can also be used. Again the path will be `/iot/d`, but in this case, `d` (Data) is not necessary - 
the key-value pairs of the measurement are passed as the body of the request. '`i` and `k` query parameters are
still mandatory:

* `i` (device ID): Device ID (unique for the API Key).
* `k` (API Key): API Key for the service the device is registered on.
* `t` (timestamp): Timestamp of the measure. Will override the automatic IoTAgent timestamp (optional).

Once again the `i` and `k`  parameters are mandatory.




