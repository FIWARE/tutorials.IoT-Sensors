[![FIWARE Banner](https://fiware.github.io/tutorials.IoT-Sensors/img/fiware.png)](https://www.fiware.org/developers)
[![NGSI v2](https://img.shields.io/badge/NGSI-v2-5dc0cf.svg)](https://fiware-ges.github.io/orion/api/v2/stable/)

[![FIWARE IoT Agents](https://nexus.lab.fiware.org/repository/raw/public/badges/chapters/iot-agents.svg)](https://github.com/FIWARE/catalogue/blob/master/iot-agents/README.md)
[![License: MIT](https://img.shields.io/github/license/fiware/tutorials.IoT-Sensors.svg)](https://opensource.org/licenses/MIT)
[![Support badge](https://img.shields.io/badge/tag-fiware-orange.svg?logo=stackoverflow)](https://stackoverflow.com/questions/tagged/fiware)
[![UltraLight 2.0](https://img.shields.io/badge/Payload-Ultralight-27ae60.svg)](https://fiware-iotagent-ul.readthedocs.io/en/latest/usermanual/index.html#user-programmers-manual)
<br/> [![Documentation](https://img.shields.io/readthedocs/fiware-tutorials.svg)](https://fiware-tutorials.rtfd.io)

This tutorial is an introduction to IoT devices and the usage of the
[UltraLight 2.0](https://fiware-iotagent-ul.readthedocs.io/en/latest/usermanual/index.html#user-programmers-manual)
Protocol for constrained devices. The tutorial introduces a series of dummy IoT devices which are displayed within the
browser and allows a user to interact with them. A complete understanding of all the terms and concepts defined in this
tutorial is necessary before proceeding to connect the IoT devices to the Orion Context Broker via a real IoT Agent.

The tutorial uses [cUrl](https://ec.haxx.se/) commands throughout, but is also available as
[Postman documentation](https://fiware.github.io/tutorials.IoT-Sensors/)

[![Run in Postman](https://run.pstmn.io/button.svg)](https://app.getpostman.com/run-collection/32975e01a2c250698149)

🇯🇵 このチュートリアルは[日本語](README.ja.md)でもご覧いただけます。<br/> 🇪🇸 Este tutorial también está disponible en
[español](README.es.md)

## Contents

<details>
<summary><strong>Details</strong></summary>

-   [What are IoT devices?](#what-are-iot-devices)
-   [What is Ultralight 2.0?](#what-is-ultralight-20)
    -   [Southbound Traffic (Commands)](#southbound-traffic-commands)
        -   [Push Command using HTTP POST](#push-command-using-http-post)
    -   [Northbound Traffic (Measurements)](#northbound-traffic-measurements)
        -   [Measurement using HTTP GET](#measurement-using-http-get)
        -   [Measurement using HTTP POST](#measurement-using-http-post)
-   [Architecture](#architecture)
-   [Prerequisites](#prerequisites)
    -   [Docker](#docker)
    -   [Cygwin](#cygwin)
-   [Start Up](#start-up)
-   [Communicating with IoT Devices](#communicating-with-iot-devices)
    -   [Bell Commands](#bell-commands)
        -   [Ring a bell](#ring-a-bell)
    -   [Smart Lamp Commands](#smart-lamp-commands)
        -   [Switch on a Smart Lamp](#switch-on-a-smart-lamp)
        -   [Switch off a Smart Lamp](#switch-off-a-smart-lamp)
    -   [Smart Door Commands](#smart-door-commands)
        -   [Unlock a Door](#unlock-a-door)
        -   [Open a Door](#open-a-door)
        -   [Close a Door](#close-a-door)
        -   [Lock a Door](#lock-a-door)
-   [Next Steps](#next-steps)

</details>

# What are IoT devices?

> "All our knowledge begins with the senses."
>
> — Immanuel Kant (Critique of Pure Reason)

The [Internet of Things](https://www.linux.com/news/who-needs-internet-things) (IoT) is a network of physical devices
which are able to connect to a network and exchange data. Each "thing" or "smart device" is a gadget with embedded
electronics and software which can act as a sensor or actuator. Sensors are able to report the state of the real-world
around them. Actuators are responsible for altering the state of the system, by responding to a control signal.

Each device is uniquely identifiable through its embedded computing system but is able to inter-operate within the
existing internet infrastructure.

FIWARE is a system for managing context information. For a smart solution based on the internet of Things, the context
is provided by the array of attached IoT devices. Since each IoT device is a physical object which exists in the real
world, it will eventually be represented as a unique entity within the context.

IoT devices can range from simple to complex. Here are some examples of IoT devices which will be used within this
tutorial:

-   A **Smart Door** is an electronic door which can be sent commands to be locked or unlocked remotely. It can also
    report on its current state (`OPEN`, `CLOSED` or `LOCKED`),
-   A **Bell** can be sent a command to activate and ring for a short period
-   A **Motion Sensor** can be queried to return the number of people who have passed by since it was last reset
-   A **Smart Lamp** can be switched on or off remotely. It can also report on its current state (`ON` or `OFF`). When
    switched on, a Motion Sensor within the device checks to see if light is needed and will dim if no-one is nearby.
    Furthermore the device itself can measure the current luminosity of the bulb.

As you can see, the **Bell** is an example of a pure actuator, as it only reacts to the given commands. Meanwhile the
**Motion Sensor** is an example of a pure sensor, since it will only report on the state of the world as it sees it. The
other two devices are able to both respond to commands and report on state in a meaningful way.

The state information held within each device, as it will eventually be seen within the Context Broker is defined in the
diagram below:

![](https://fiware.github.io/tutorials.IoT-Sensors/img/entities.png)

# What is Ultralight 2.0?

[UltraLight 2.0](https://fiware-iotagent-ul.readthedocs.io/en/latest/usermanual/index.html#user-programmers-manual) is a
lightweight text based protocol for constrained devices and communications where bandwidth and device memory resources
are limited. The payload for measurement requests is a list of key-value pairs separated by the pipe `|` character.

e.g.

```
<key>|<value>|<key>|<value>|<key>|<value> etc..
```

For example a payload such as:

```
t|15|k|abc
```

Contains two attributes, one named "t" with value "15" and another named "k" with value "abc" are transmitted. Values in
Ultralight 2.0 are not typed (everything is treated as a string).

Ultralight 2.0 defines a payload describing measures and commands to share between devices and servers but, does not
specify a single transport protocol. Instead, different transport protocol bindings (such as HTTP, MQTT and AMQP) can be
used for different scenarios. For this tutorial we will be using HTTP as a transport protocol.

## Southbound Traffic (Commands)

HTTP requests generated by the from the Context Broker and passed downwards towards an IoT device (via an IoT Agent) are
known as southbound traffic. Southbound traffic consists of **commands** made to actuator devices which alter the state
of the real world by their actions. For example a command to alter the state of a lamp to `ON` would switch on the lamp
in real life. This in turn could alter the readings of other sensors nearby.

### Push Command using HTTP POST

Setting up the southbound communication between an IoT Agent and IoT devices is known as provisioning. This ensures that
the IoT Agent holds sufficient information to be able to contact each IoT device. In other words it knows where to send
commands and which commands are supported. In order to send a command to a device, the IoT Agent sends a POST request to
the endpoint supplied by the device. The body of the POST request holds the command.

The payload for Ultralight commands has the following format:

```
<device name>@<command>|<param|<param>
```

Where `<device_name>` is the entity `id` as held in the context broker, `<command>` is one of the supported commands and
any additional required values are passed in subsequent parameters for example

```
urn:ngsi-ld:Robot:001@turn|left|30
```

Will tell a device _"I am known as `id="urn:ngsi-ld:Robot:001"` within the Context Broker. I would like the device
listening on this endpoint to perform the `turn` command. I have supplied the parameters `left` and '`30` (degrees) as
required for the device to be able to perform the maneuver"_.

The defined Northbound response to an IoT Agent is as follows:

```
urn:ngsi-ld:Robot:001@turn|Turn ok
```

Which is saying _"I have complied with a request from the entity known as `id="urn:ngsi-ld:Robot:001"` within the
Context Broker. The command I have performed was a `turn` command. The result was `Turn ok`"_.

As you can see, because the Southbound command defines the `id` used within the interaction, and the same data is also
returned, every response can always be associated to the appropriate entity held within the Context Broker.

Push commands can only be used if the device is able to supply a separate endpoint for listening to southbound traffic,
an alternative polling mechanism can be used when all interactions are initiated from the device itself, but this is
beyond the scope of this tutorial.

## Northbound Traffic (Measurements)

Requests generated from an IoT device and passed back upwards towards the Context Broker (via an IoT Agent) are known as
northbound traffic. Northbound traffic consists of **measurements** made by sensor devices and relays the state of the
real world into the context data of the system. For example a measurement from a humidity sensor could be relayed back
into the context broker to indicate that the moisture level of the entity has changed. A subscription could be made to
be informed of such changes and there provoke further actions (such as turning on a sprinkler)

### Measurement using HTTP GET

A device can report new measures to an IoT Agent using an HTTP GET request to a "well-known" endpoint (the path
`/iot/d`) along with the following query parameters:

-   `i` (device ID): Device ID (unique for the API Key).
-   `k` (API Key): API Key for the service the device is registered on.
-   `t` (timestamp): Timestamp of the measure. Will override the automatic IoTAgent timestamp (optional).
-   `d` (Data): Ultralight 2.0 payload.

The `i` and `k` parameters are mandatory.

For example the request:

```
<iot-agent>/iot/d?i=motion001&d=c|12
```

Would indicate that the device `id=motion001` wishes to inform the IoT Agent that is has made a real-world measurement
`c` with the value `12`. This would eventually be passed up into the Context Broker.

### Measurement using HTTP POST

HTTP POST can also be used. Again the path will be `/iot/d`, but in this case, `d` (Data) is not necessary - the
key-value pairs of the measurement are passed as the body of the request. `i` and `k` query parameters are still
mandatory:

-   `i` (device ID): Device ID (unique for the API Key).
-   `k` (API Key): API Key for the service the device is registered on.
-   `t` (timestamp): Timestamp of the measure. Will override the automatic IoTAgent timestamp (optional).

Once again the `i` and `k` parameters are mandatory.

#### Device Monitor

For the purpose of this tutorial, a series of dummy IoT devices have been created, which will eventually be attached to
the context broker. The state of each device can be seen on the UltraLight device monitor web page found at:
`http://localhost:3000/device/monitor`

![FIWARE Monitor](https://fiware.github.io/tutorials.IoT-Sensors/img/device-monitor.png)

# Architecture

The demo application will only make use of a single custom component acting as a set of dummy IoT devices. Every IoT
device will be using the
[UltraLight 2.0](https://fiware-iotagent-ul.readthedocs.io/en/latest/usermanual/index.html#user-programmers-manual)
protocol running over HTTP. Since all interactions are initiated by HTTP requests, the entities can be containerized and
run from exposed ports.

![](https://fiware.github.io/tutorials.IoT-Sensors/img/architecture.png)

The necessary configuration information can be seen in the services section of the associated `docker-compose.yml` file:

```yaml
tutorial:
    image: fiware/tutorials.context-provider
    hostname: iot-sensors
    container_name: fiware-tutorial
    networks:
        - default
    expose:
        - "3000"
        - "3001"
    ports:
        - "3000:3000"
        - "3001:3001"
    environment:
        - "DEBUG=tutorial:*"
        - "PORT=3000"
        - "IOTA_HTTP_HOST=iot-agent"
        - "IOTA_HTTP_PORT=7896"
        - "DUMMY_DEVICES_PORT=3001" # Port used by the dummy IOT devices to receive commands
        - "DUMMY_DEVICES_API_KEY=4jggokgpepnvsb2uv4s40d59ov"
```

The `tutorial` container is listening on two ports:

-   Port `3000` is exposed so we can see the web page displaying the Dummy IoT devices.
-   Port `3001` is exposed purely for tutorial access - so that cUrl or Postman can make UltraLight commands without
    being part of the same network.

The `tutorial` container is driven by environment variables as shown:

| Key                   | Value                        | Description                                                                                                                                                                        |
| --------------------- | ---------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| DEBUG                 | `tutorial:*`                 | Debug flag used for logging                                                                                                                                                        |
| WEB_APP_PORT          | `3000`                       | Port used by web-app which displays the dummy device data                                                                                                                          |
| IOTA_HTTP_HOST        | `iot-agent`                  | The hostname of the missing IoT Agent - used in a later tutorial                                                                                                                   |
| IOTA_HTTP_PORT        | `7896`                       | The port that the missing IoT Agent will be listening on. `7896` is a common default for UltraLight over HTTP                                                                      |
| DUMMY_DEVICES_PORT    | `3001`                       | Port used by the dummy IoT devices to receive commands                                                                                                                             |
| DUMMY_DEVICES_API_KEY | `4jggokgpepnvsb2uv4s40d59ov` | Random security key used for UltraLight interactions - this will be used in a later tutorial to ensure the integrity of interactions between the devices and the missing IoT Agent |

The other `tutorial` container configuration values described in the YAML file are not used in this tutorial.

When describing the messages being passed through a working smart solution we will refer to two further components which
are not used in this tutorial, but will be needed to complete the system subsequently.

-   The [Orion Context Broker](https://fiware-orion.readthedocs.io/en/latest/) is used for holding the context data of
    the smart solution. As you know all interactions with the context broker must be made using
    [NGSI-v2](https://fiware.github.io/specifications/OpenAPI/ngsiv2)
-   An IoT Agent acts as a middleware component converting
    [NGSI-v2](https://fiware.github.io/specifications/OpenAPI/ngsiv2) requests (from the context broker) into a protocol
    (such as
    [UltraLight 2.0](https://fiware-iotagent-ul.readthedocs.io/en/latest/usermanual/index.html#user-programmers-manual))
    usable by the IoT devices themselves.

It is therefore necessary to understand a sample device protocol first, and comprehend how messages are passed through
the system to subsequently understand the purpose of the IoT Agent middleware. In this tutorial you will be playing the
role of an IoT Agent making commands to devices and receiving measurements from them.

# Prerequisites

## Docker

To keep things simple all components will be run using [Docker](https://www.docker.com). **Docker** is a container
technology which allows to different components isolated into their respective environments.

-   To install Docker on Windows follow the instructions [here](https://docs.docker.com/docker-for-windows/)
-   To install Docker on Mac follow the instructions [here](https://docs.docker.com/docker-for-mac/)
-   To install Docker on Linux follow the instructions [here](https://docs.docker.com/install/)

**Docker Compose** is a tool for defining and running multi-container Docker applications. A
[YAML file](https://raw.githubusercontent.com/Fiware/tutorials.Entity-Relationships/master/docker-compose.yml) is used
configure the required services for the application. This means all container services can be brought up in a single
command. Docker Compose is installed by default as part of Docker for Windows and Docker for Mac, however Linux users
will need to follow the instructions found [here](https://docs.docker.com/compose/install/)

You can check your current **Docker** and **Docker Compose** versions using the following commands:

```console
docker-compose -v
docker version
```

Please ensure that you are using Docker version 20.10 or higher and Docker Compose 1.29 or higher and upgrade if
necessary.

## Cygwin

We will start up our services using a simple bash script. Windows users should download [cygwin](http://www.cygwin.com/)
to provide a command-line functionality similar to a Linux distribution on Windows.

# Start Up

All services can be initialized from the command-line by running the bash script provided within the repository. Please
clone the repository and create the necessary images by running the commands as shown:

```console
git clone https://github.com/FIWARE/tutorials.IoT-Sensors.git
cd tutorials.IoT-Sensors
git checkout NGSI-v2

./services create; ./services start;
```

This command will also import seed data from the previous
[Stock Management example](https://github.com/FIWARE/tutorials.Context-Providers) on startup.

> :information_source: **Note:** If you want to clean up and start over again you can do so with the following command:
>
> ```console
> ./services stop
> ```

# Communicating with IoT Devices

To follow the tutorial correctly please ensure you have the device monitor page available in your browser and click on
the page to enable audio before you enter any cUrl commands. The device monitor displays the current state of an array
of dummy devices using Ultralight 2.0 syntax

#### Device Monitor

The device monitor can be found at: `http://localhost:3000/device/monitor`

Within this tutorial you will be playing the role of the missing IoT Agent component, making Southbound commands to the
attached IoT devices and receiving Northbound measurements as the environment changes within the store. All the commands
are made as HTTP POST requests using Ultralight syntax and therefore are very simple. It is worthwhile keeping an eye on
the device monitor page as it shows all the Northbound traffic generated by the devices themselves.

## Bell Commands

A **Bell** is an example of an actuator. It can respond to commands, but the device does not supply any measurements
from the real world.

### Ring a bell

This example shows how a real IoT Agent would send commands to an actuator. The **Bell** has supplied an endpoint
`/iot/bell001` where it is listening for commands.

#### :one: Request:

```console
curl -iX POST \
  --url 'http://localhost:3001/iot/bell001' \
  --data urn:ngsi-ld:Bell:001@ring
```

#### Response:

```
urn:ngsi-ld:Bell:001@ring| ring OK
```

The body of the request is in Ultralight syntax and consists of the `id` of the device (`urn:ngsi-ld:Bell:001`) as held
in the Context Broker and the name of the command (`ring`) to invoke on the device.

The response returns the command and the result of the action.

If you are viewing the device monitor page, you can see the state of the bell change.

![](https://fiware.github.io/tutorials.IoT-Sensors/img/bell-ring.gif)

## Smart Lamp Commands

The **Smart Lamp** can be switched on and off remotely - it also registers luminosity. It contains a Motion Sensor
within it and will slowly dim as time passes (provided no movement is detected)

Measurements will be sent to the IoT Agent as the state and/or luminosity changes.

### Switch on a Smart Lamp

This example shows how a real IoT Agent would send an Ultralight command to a **Smart Lamp** to switch it on. The
**Smart Lamp** has already supplied an endpoint `/iot/lamp001` where it is listening for commands.

#### :two: Request:

```console
curl -iX POST \
  --url 'http://localhost:3001/iot/lamp001' \
  --data urn:ngsi-ld:Lamp:001@on
```

The body of the request consists of the `id` of the device (`urn:ngsi-ld:Lamp:001`) as held in the Context Broker and
the name of the command (`on`) to invoke on the device.

#### Response:

The response returns the command and the result of the action.

```
urn:ngsi-ld:Lamp:001@on| on OK
```

Once the lamp is switched on the luminosity level will alter dependent upon whether the internal motion sensor detects
movement. The measurement is actively reported and requests to the IoT Broker can be seen on the device monitor page.

### Switch off a Smart Lamp

This example shows how a real IoT Agent would send an Ultralight command to a **Smart Lamp** to switch it off. The
**Smart Lamp** has already supplied an endpoint `/iot/lamp001` where it is listening for commands.

#### :three: Request:

```console
curl -iX POST \
  --url 'http://localhost:3001/iot/lamp001' \
  --data urn:ngsi-ld:Lamp:001@off
```

The body of the request consists of the `id` of the device (`urn:ngsi-ld:Lamp:001`) as held in the Context Broker and
the name of the command (`off`) to invoke on the device.

#### Response:

The response returns the command and the result of the action.

```
urn:ngsi-ld:Lamp:001@off| off OK
```

Once the lamp is switched off the luminosity level does not alter. The latest Ultralight measurement (`s|OFF|l|0`) as
sent to the IoT Broker can be seen on the device monitor page.

To turn the **Smart Lamp** back on again repeat the following command:

#### :four: Request:

```console
curl -iX POST \
  --url 'http://localhost:3001/iot/lamp001' \
  --data urn:ngsi-ld:Lamp:001@on
```

#### Response:

```
urn:ngsi-ld:Lamp:001@on| on OK
```

## Smart Door Commands

A **Smart Door** is an electronic door which can be sent commands to be locked or unlocked remotely. It can also report
on its current state (`OPEN`, `CLOSED` or `LOCKED`),

Measurements will be sent to the IoT Agent as the state changes.

### Unlock a Door

This example shows how a real IoT Agent would send an Ultralight command to a **Smart Door** to unlock the door. The
**Smart Door** has already supplied an endpoint `/iot/door001` where it is listening for commands.

#### :five: Request:

```console
curl -iX POST \
  --url 'http://localhost:3001/iot/door001' \
  --data urn:ngsi-ld:Door:001@unlock
```

The body of the request consists of the `id` of the device (`urn:ngsi-ld:Door:001`) as held in the Context Broker and
the name of the command (`unlock`) to invoke on the device.

#### Response:

The response returns the command and the result of the action.

```
urn:ngsi-ld:Door:001@unlock| unlock OK
```

Once the **Smart Door** is unlocked, it will automatically open and close as customers enter. The changes of state are
actively reported to the IoT Broker, and the state of the **Smart Door** can be seen on the device monitor page.

The **Motion Sensor** within the store is not an actuator - it does not respond to commands, however it does actively
measure the number of customers passing by. If the door is unlocked, the **Motion Sensor** will detect movement and send
Ultralight measurements back up to the IoT Broker.

The Northbound HTTP requests generated by the **Motion Sensor** can be also viewed on the device monitor page.

![](https://fiware.github.io/tutorials.IoT-Sensors/img/door-open.gif)

### Open a Door

This example shows how a real IoT Agent would send a command to a **Smart Door** to open the door. The **Smart Door**
has already supplied an endpoint `/iot/door001` where it is listening for commands.

#### :six: Request:

```console
curl -iX POST \
  --url 'http://localhost:3001/iot/door001' \
  --data urn:ngsi-ld:Door:001@open
```

The body of the request consists of the `id` of the device (`urn:ngsi-ld:Door:001`) as held in the Context Broker and
the name of the command (`open`) to invoke on the device.

#### Response:

The response returns the command and the result of the action.

```
urn:ngsi-ld:Door:001@open| open OK
```

The state of the **Smart Door** can be seen on the device monitor page. Customers may now enter and the **Motion
Sensor** may pick up movement and send measurements to the IoT Broker.

The Northbound HTTP requests generated by the **Smart Door** and the **Motion Sensor** can also be viewed on the device
monitor page.

### Close a Door

This example shows how a real IoT Agent would send a command to a **Smart Door** to close the door. The **Smart Door**
has already supplied an endpoint `/iot/door001` where it is listening for commands.

#### :seven: Request:

```console
curl -iX POST \
  --url 'http://localhost:3001/iot/door001' \
  --data urn:ngsi-ld:Door:001@close
```

The body of the request consists of the `id` of the device (`urn:ngsi-ld:Door:001`) as held in the Context Broker and
the name of the command (`close`) to invoke on the device.

#### Response:

The response returns the command and the result of the action.

```
urn:ngsi-ld:Door:001@close| close OK
```

Since the door is currently unlocked, customers will continue to enter, and re-open the door themselves. If motion is
detected, the **Motion Sensor** will send measurements to the IoT Broker.

The Northbound HTTP requests generated by the **Motion Sensor** can also be viewed on the device monitor page.

### Lock a Door

This example shows how a real IoT Agent would send an Ultralight command to a Smart Door to lock the door. The Smart
Door has already supplied an endpoint `/iot/door001` where it is listening for commands.

#### :eight: Request:

```console
curl -iX POST \
  --url 'http://localhost:3001/iot/door001' \
  --data urn:ngsi-ld:Door:001@lock
```

The body of the request consists of the `id` of the device (`urn:ngsi-ld:Door:001`) as held in the Context Broker and
the name of the command (`lock`) to invoke on the device.

#### Response:

The response returns the command and the result of the action.

```
urn:ngsi-ld:Door:001@lock| lock OK
```

Once the door is locked, no further customers may enter. The **Motion Sensor** will report no further movement detected,
the **Smart Door** cannot be opened manually and the **Smart Lamp** will slowly return to the ambient lighting level.

The Northbound HTTP requests generated by the **Smart Lamp** can be viewed on the device monitor page.

![](https://fiware.github.io/tutorials.IoT-Sensors/img/door-lock.gif)

# Next Steps

Want to learn how to add more complexity to your application by adding advanced features? You can find out by reading
the other [tutorials in this series](https://fiware-tutorials.rtfd.io)

---

## License

[MIT](LICENSE) © 2018-2020 FIWARE Foundation e.V.
