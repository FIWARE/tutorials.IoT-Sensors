[![FIWARE Banner](https://fiware.github.io/tutorials.IoT-Sensors/img/fiware.png)](https://www.fiware.org/developers) [<img src="https://raw.githubusercontent.com/FIWAREZone/misc/master/Group%400%2C36x.png" align="right">](http://www.fiware.zone)

[![FIWARE IoT Agents](https://nexus.lab.fiware.org/repository/raw/public/badges/chapters/iot-agents.svg)](https://github.com/FIWARE/catalogue/blob/master/iot-agents/README.md)
[![License: MIT](https://img.shields.io/github/license/fiware/tutorials.IoT-Sensors.svg)](https://opensource.org/licenses/MIT)
[![NGSI v2](https://img.shields.io/badge/Payload-Ultralight-27ae60.svg)](https://fiware-iotagent-ul.readthedocs.io/en/latest/usermanual/index.html#user-programmers-manual)
[![Support badge](https://nexus.lab.fiware.org/repository/raw/public/badges/stackoverflow/fiware.svg)](https://stackoverflow.com/questions/tagged/fiware)
<br/> [![Documentation](https://img.shields.io/readthedocs/fiware-tutorials.svg)](https://fiware-tutorials.rtfd.io)

Este tutorial es una introducción a los dispositivos IoT y al uso del protocolo
[UltraLight 2.0](https://fiware-iotagent-ul.readthedocs.io/en/latest/usermanual/index.html#user-programmers-manual)
para dispositivos con capacidades limitadas. El tutorial presenta una serie de dispositivos ficticios que se muestran en el navegador y permiten interactuar con ellos. Se necesita una comprensión completa de todos los términos y conceptos en este tutorial antes de proceder a conectar un dispositivo IoT al Orion Context Broker por medio de un agente IoT real.

A lo largo de este tutorial se usan comandos [cUrl](https://ec.haxx.se/), pero también está disponible como
[documentación Postman](https://fiware.github.io/tutorials.IoT-Sensors/)

[![Run in Postman](https://run.pstmn.io/button.svg)](https://app.getpostman.com/run-collection/32975e01a2c250698149)

> Este tutorial ha sido traducido por **FIWARE ZONE**, una iniciativa sin ánimo de lucro entre **Telefónica** y la **Junta de Andalucía** cuyo fin es la divulgación, promoción y difusión de la tecnología *FIWARE*, para hacer crecer el ecosistema y generar conocimiento y oportunidades a los desarrolladores y empresas andaluzas. **FIWARE ZONE**, como *iHub* de 3 estrellas ofrece servicios de alto nivel en formación, mentorización y consultoría de forma totalmente **gratuita**. Si necesitas cualquier tipo de ayuda o quieres contarnos tu proyecto, puedes ponerte en contacto con nosotros a través de la dirección [fiware.zone@fiware.zone](mailto:fiware.zone@fiware.zone), por nuestro [formulario web](https://fiware.zone/contacto/), en cualquiera de nuestras redes sociales o físicamente en [nuestros centros en Málaga y Sevilla](https://fiware.zone/contacto/)
>
>![3 stars iHub](https://badgen.net/badge/iHub/3%20stars/yellow)
>
>[![Twitter](https://raw.githubusercontent.com/FIWAREZone/misc/master/twitter.png)](https://twitter.com/FIWAREZone)
[![Linkedin](https://raw.githubusercontent.com/FIWAREZone/misc/master/linkedin.png)](https://www.linkedin.com/company/fiware-zone)
[![Instagram](https://raw.githubusercontent.com/FIWAREZone/misc/master/instagram.png)](https://www.instagram.com/fiwarezone/)
[![Github](https://raw.githubusercontent.com/FIWAREZone/misc/master/github.png)](https://github.com/FIWAREZone)
[![Facebook](https://raw.githubusercontent.com/FIWAREZone/misc/master/facebook.png)](https://www.facebook.com/FIWAREZone/)

## Contenido

<details>
<summary><strong>Details</strong></summary>

-   [¿Que son dispositivos IoT?](#¿que-son-dispositivos-iot)
-   [¿Que es Ultralight 2.0?](#¿que-es-ultralight-20)
    -   [Trafico hacia el sur (Comandos)](#trafico-hacia-el-sur-comandos)
        -   [Envío de comandos usando HTTP POST](#envio-de-comandos-usando-http-post)
    -   [Trafico hacia el norte (Medidas)](#trafico-hacia-el-norte-medidas)
        -   [Envió de medidas usando HTTP GET](#envio-de-medidas-usando-http-get)
        -   [Envío de medidas usando HTTP POST](#envio-de-medidas-usando-http-post)
-   [Arquitectura](#arquitectura)
-   [Prerequisitos](#prerequisitos)
    -   [Docker](#docker)
    -   [Cygwin](#cygwin)
-   [Inicio](#inicio)
-   [Comunicando con los dispositivos IoT](#comunicando-con-los-dispositivos-iot)
    -   [Comandos de campana](#comandos-de-campana)
        -   [Tocar una campana](#tocar-una-campana)
    -   [Comandos de bombilla inteligente](#comandos-de-bombilla-inteligente)
        -   [Encender una bombilla inteligente](#encender-una-bombilla-inteligente)
        -   [Apagar una bombilla inteligente](#apagar-una-bombilla-inteligente)
    -   [Comandos de puerta inteligente](#comandos-de-puerta-inteligente)
        -   [Desbloquear una puerta](#desbloquear-una-puerta)
        -   [Abrir una puerta](#abrir-una-puerta)
        -   [Cerrar una puerta](#cerrar-una-puerta)
        -   [Bloquear una puerta](#bloquear-una-puerta)
-   [Próximos pasos](#próximos-pasos)

</details>

# ¿Que son dispositivos IoT?

> "All our knowledge begins with the senses."
>
> — Immanuel Kant (Critique of Pure Reason)

El internet de las cosas o [Internet of Things](https://www.linux.com/news/who-needs-internet-things) (IoT) es una red de dispositivos físicos que son capaces
de conectarse a la red e intercambiar datos. Cada "cosa" o "dispositivo inteligente" es un dispositivo con electrónica integrada y software que puede actuar como sensor o como actuador. Los sensores son capaces de informar del estado del mundo real alrededor de ellos. Los actuadores son responsables de modificar el estado de los sistema, de acuerdo a las señales de control.

Cada dispositivo es identificable unicamente por medio de su sistema de computación embebido siendo capaz de interoperar con la infraestructura de red existente

FIWARE es un sistema de gestión de la información de contexto. Para una solución inteligente basada en la Internet de las Cosas, el contexto es proporcionado por el conjunto de dispositivos de IoT conectados. Dado que cada dispositivo de IoT es un objeto físico que existe en el mundo real, con el tiempo se representará como una entidad única dentro del contexto.

Podemos encontrar una gran variedad de dispositivos de acuerdo a su complejidad. A continuación se presentan algunos ejemplos de dispositivos de IO que se utilizarán en este tutorial:

-   Una puerta inteligente o **Smart Door** es una puerta electrónica que puede ser enviada con comandos para ser bloqueada o desbloqueada remotamente. También puede informar sobre su estado actual (`OPEN`, `CLOSED` or `LOCKED`),
-   Una campana o **Bell**, a la cual se le puede enviar un comando para activar y hacerla sonar por un corto periodo de tiempo
-   Un sensor de movimiento o **Motion Sensor** puede ser consultado para devolver el número de personas que han pasado desde la última vez que fue reseteado
-   Una lámpara inteligente o **Smart Lamp** puede ser encendida o apagada de forma remota. También puede informar de su estado actual, encendida (`ON`) o apagada (`OFF`). un sensor de movimiento dentro del dispositivo comprueba si se necesita luz y se atenúa si no hay nadie cerca. Además, el dispositivo puede informar sobre la luminosidad actual de la bombilla.

Como puedes ver, la campana **Bell** es un ejemplo de un actuador puro, ya que sólo reacciona a las órdenes dadas. Mientras tanto, el Sensor de Movimiento **Motion Sensor** es un ejemplo de un sensor puro, ya que sólo informará sobre el estado del mundo tal y como lo ve. Los otros dos dispositivos son capaces de responder a las órdenes y de informar sobre el estado de una manera significativa.

La información de estado que se mantiene dentro de cada dispositivo, como se verá posteriormente en el Context Broker se define en el siguiente diagrama:
![](https://fiware.github.io/tutorials.IoT-Sensors/img/entities.png)

# ¿Que es Ultralight 2.0?

[UltraLight 2.0](https://fiware-iotagent-ul.readthedocs.io/en/latest/usermanual/index.html#user-programmers-manual) es un protocolo ligero basado en texto para dispositivos y comunicaciones limitados en los que el ancho de banda y los recursos de memoria del dispositivo son limitados. El mensaje de los envíos de las medidas es una lista de pares clave-valor separados por el carácter `|` o tubería.

```
<key>|<value>|<key>|<value>|<key>|<value> etc..
```

Un ejemplo sería el siguiente mensaje:

```
t|15|k|abc
```

Que contiene dos atributos, uno llamado "t" con valor "15" y otro llamado "k" con valor "abc". Los valores en UltraLight 2.0 no van tipificados (todo se trata como una cadena).

Ultralight 2.0 define una mensaje o payload describiendo medidas y comandos para compartir entre dispositivos y servidores pero, no especifica un único protocolo de transporte. En su lugar, se pueden utilizar diferentes protocolos de transporte (como HTTP, MQTT y AMQP) para diferentes escenarios. En este tutorial utilizaremos HTTP como protocolo de transporte.

## Trafico hacia el sur (Comandos)

Las peticiones HTTP generadas por el Context Broker y pasadas hacia abajo hacia un dispositivo de IoT (a través de un agente de IO) se conocen como tráfico hacia el sur. El tráfico hacia el sur consiste en **comandos** enviados a dispositivos actuadores que alteran el estado del mundo real por sus acciones. Por ejemplo, un comando para alterar el estado de una lámpara a "ON" encendería la lámpara en la vida real. Esto a su vez podría alterar las lecturas de otros sensores cercanos

### Envío de comandos usando HTTP POST

Establecer la comunicación hacia el sur entre un agente IoT y los dispositivos se conoce como provisionamiento. Esto asegura que el agente IoT tenga suficiente información para poder comunicarse con cada dispositivo IoT. En otras palabras, sabe dónde enviar los comandos y qué comandos están soportados. Para enviar un comando a un dispositivo, el agente de IoT envía una petición POST a la dirección facilitada por el dispositivo. El cuerpo de la petición HTTP contiene el comando.

Para el protocolo Ultralight 2.0, el mensaje sigue el siguiente formato:

```
<device name>@<command>|<param|<param>
```
Donde `<device_name>` es el  `id` de la entidad en el context broker, `<command>` es uno de los comandos soportados y cualquier valor adicional requerido  pasa en los parámetros subsiguientes, por ejemplo:
```
urn:ngsi-ld:Robot:001@turn|left|30
```

Le dirá a un dispositivo _"Soy conocido como `id="urn:ngsi-ld:Robot:001"` dentro del Context Broker. Me gustaría que el dispositivo que está escuchando en esta dirección realice el comando `turn`. He suministrado los parámetros izquierda `left` y `30` (grados) necesarios para que el dispositivo pueda realizar la maniobra"_.

La respuesta definida hacia el norte de un agente IoT es la siguiente:

```
urn:ngsi-ld:Robot:001@turn|Turn ok
```
Lo que significa: _"He cumplido con una solicitud de la entidad conocida como `id="urn:ngsi-ld:Robot:001"` dentro del Context Broker. El comando que he realizado fue un comando de giro - `turn`. El resultado fue giro satisfactorio - `Turn ok`"_.

Como se puede ver, debido a que el comando define el `id` utilizado dentro de la interacción, y también se devuelven los mismos datos, cada respuesta siempre puede ser asociada a la entidad apropiada del Context Broker.

Los comandos push sólo se pueden utilizar si el dispositivo es capaz de suministrar una dirección - o endpoint - separado para escuchar el tráfico en dirección sur. Esto supone que el dispositivo tenga una IP asignada y esta sea accesible, así como ser capaz de escuchar el tráfico entrante en ese puerto.

Alternativamente se puede usar otro método cuando todas las interacciones se inician desde el propio dispositivo, pero esto está fuera del alcance de este tutorial.

## Trafico hacia el norte (Medidas)

Las peticiones generadas por un dispositivo IoT y pasadas de nuevo (hacia arriba) al Context Broker (a través de un agente IoT) se conocen como tráfico hacia el norte. El tráfico en dirección norte consiste en mediciones - **measurements** - realizadas por sensores, que transmite el estado del mundo real a los datos de contexto del sistema. Por ejemplo, una medición de un sensor de humedad se envía al Context Broker para indicar que el nivel de humedad de la entidad ha cambiado. Se podría hacer una suscripción para ser informado de tales cambios y provocar otras acciones (como encender un aspersor)

### Envió de medidas usando HTTP GET

Un dispositivo puede informar de nuevas medidas a un agente IoT utilizando una petición HTTP GET a una dirección o endpoint "conocido" (la ruta `/iot/d`) junto con los siguientes parámetros en la consulta:

-   `i` (device ID): Identificador de dispositivo (único para cada API Key).
-   `k` (API Key): Es una contraseña para el servicio en el que el dispositivo está registrado.
-   `t` (timestamp): Marca temporal de la medida. Si existe, sobrescribirá de forma automática la marca temporal del agente IoT (opcional).
-   `d` (Data): El dato o mensaje en formato Ultralight 2.0.

Los parámetros `i` y `k` son obligatorios.

Por ejemplo, la petición:

```
<iot-agent>/iot/d?i=motion001&d=c|12
```

Indicaría que el dispositivo `id=motion001` desea informar al agente IoT que ha hecho una medición en el mundo real `c` con el valor `12`. Esto finalmente pasaría enviaría al Context Broker.

### Envió de medidas usando HTTP POST

Tambien se puede usar una petición HTTP POST para el envío de datos. Una vez más, la ruta sería `/iot/d`, pero en este caso el parámetro `d` (Data) no es necesario. - los pares de clave y valor de las medidas se pasan como cuerpo de la petición. Los parámetros `i` y `k` siguen siendo obligatorios:

-   `i` (device ID): Identificador de dispositivo (único para cada API Key).
-   `k` (API Key): Es una contraseña para el servicio en el que el dispositivo está registrado.
-   `t` (timestamp): Marca temporal de la medida. Si existe, sobrescribirá de forma automática la marca temporal del agente IoT (opcional).

#### Monitor de dispositivos

Para el propósito de este tutorial, se han creado una serie de dispositivos IoT ficticios, asociados al BCntext Broker. El estado de cada dispositivo se puede ver en la página web del monitor de dispositivos UltraLight que se encuentra en:
`http://localhost:3000/device/monitor`

![FIWARE Monitor](https://fiware.github.io/tutorials.IoT-Sensors/img/device-monitor.png)

# Arquitectura

La aplicación de demostración sólo hará uso de un único componente personalizado que actúa como un conjunto de dispositivos IoT ficticios. Cada dispositivo IoT utilizará el protocolo [UltraLight 2.0](https://fiware-iotagent-ul.readthedocs.io/en/latest/usermanual/index.html#user-programmers-manual) corriendo sobre HTTP. Dado que todas las interacciones se inician con peticiones HTTP, las entidades pueden ser contenedorizadas y ejecutadas desde los puertos expuestos.

![](https://fiware.github.io/tutorials.IoT-Sensors/img/architecture.png)

La información de configuración necesaria se puede ver en la sección de servicios del archivo asociado `docker-compose.yml`:

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

El contenedor `tutorial` está escuchando en dos puertos:

-   El puerto `3000` se expone para poder ver la página web que enseña los dispositivos IoT ficticios.
-   El puerto `3001` se expone únicamente para el acceso a los tutoriales - para que cUrl o Postman puedan hacer comandos de UltraLight sin ser parte de la misma red.

El contenedor `tutorial` está configurado por las variables de entorno que se muestra a continuación:

| Clave                   | Valor                        | Descripción                                                                                                                                                                        |
| --------------------- | ---------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| DEBUG                 | `tutorial:*`                 | Flag empleado para la depuración de la aplicación |
| WEB_APP_PORT          | `3000`                       | Puerto usado por la aplicación web que miuestra los dispositivos ficticios data |
| IOTA_HTTP_HOST        | `iot-agent`                  | El nombre de dominio del IoT Agent - used en tutoriales posteriores          |
| IOTA_HTTP_PORT        | `7896`                       | El puerto en el que el IoT Agent estará escuchando. `7896` es el puerto por defecto para UltraLight sobre HTTP  |
| DUMMY_DEVICES_PORT    | `3001`                       | El puerto usado por los dispositivos ficticios para recuperar comandos  |
| DUMMY_DEVICES_API_KEY | `4jggokgpepnvsb2uv4s40d59ov` | Clave de segurida aleatoria usado para las interacciones Ultralight - esta será usada en posteriores tutoriales para asgurar la integridad de las interacciones entre los dispositivos y el agente IoT |

Los otros valores de configuración del contenedor `tutorial` descritos en el archivo YAML no se utilizan en este tutorial.

Al describir los mensajes que se pasan a través de una solución inteligente de trabajo nos referiremos a dos componentes más que
no se utilizan en este tutorial, pero serán necesarios para completar el sistema posteriormente.

-   El [Orion Context Broker](https://fiware-orion.readthedocs.io/en/latest/) se utiliza para guardar los datos de contexto de la        solución inteligente. Como sabes, todas las interacciones con el Context Broker deben hacerse usando [NGSI-v2](https://fiware.github.io/specifications/OpenAPI/ngsiv2)
-   Un agente IoT actúa com intermediario o middleware convirtiendo peticiones [NGSI-v2](https://fiware.github.io/specifications/OpenAPI/ngsiv2) (desde el Context Broker) a otro protocolo (como por ejemplo [UltraLight 2.0](https://fiware-iotagent-ul.readthedocs.io/en/latest/usermanual/index.html#user-programmers-manual)) usable por los propios dispositivos IoT.

Por lo tanto, es necesario comprender primero un protocolo de dispositivo de ejemplo, y comprender cómo se pasan los mensajes a través del sistema para posteriormente comprender el propósito del middleware o Agente IoT. En este tutorial desempeñaremos el papel de un Agente IoT que envía comandos a los dispositivos y recibe mediciones de ellos.

# Prerequisitos

## Docker
Para mantener las cosas simples, ambos componentes se ejecutarán usando [Docker](https://www.docker.com). **Docker** es una tecnología de contenedores que permite aislar diferentes componentes en sus respectivos entornos.

- Para instalar Docker en Windows siga las instrucciones [aquí](https://docs.docker.com/docker-for-windows/)
- Para instalar Docker en Mac siga las instrucciones [aquí](https://docs.docker.com/docker-for-mac/)
- Para instalar Docker en Linux siga las instrucciones [aquí](https://docs.docker.com/install/)

## Docker Compose (Opcional)

**Docker Compose** es una herramienta para definir y ejecutar aplicaciones Docker multi-contenedor. A
Se utiliza el [archivo YAML](https://raw.githubusercontent.com/Fiware/tutorials.Getting-Started/master/docker-compose.yml) para configurar los servicios requeridos para la aplicación. Esto significa que todos los servicios de los contenedores pueden ser lanzados en un solo comando. Docker Compose se instala de forma predeterminada como parte de Docker para Windows y Docker para Mac, sin embargo los usuarios de Linux
tendrá que seguir las instrucciones que se encuentran [aquí](https://docs.docker.com/compose/install/)

Puede comprobar sus versiones actuales de **Docker** y **Docker Compose** usando los siguientes comandos:

```console
docker-compose -v
docker version
```

Por favor, asegúrese de que está utilizando la versión 18.03 o superior de Docker y la versión 1.21 o superior de Docker Compose y actualícela si es necesario.

## Cygwin

Comenzaremos nuestros servicios usando una simple consola de comando - o bash script. Los usuarios de Windows deben descargar [cygwin](http://www.cygwin.com/) para proporcionar una funcionalidad de línea de comandos similar a la de una distribución de Linux en Windows.

# Inicio

Todos los servicios pueden ser inicializados desde la línea de comandos ejecutando el script proporcionado en el repositorio. Por favor,
clone el repositorio y cree las imágenes necesarias ejecutando los comandos como se muestra a continuación:

```console
git clone https://github.com/FIWARE/tutorials.IoT-Sensors.git
cd tutorials.IoT-Sensors

./services create; ./services start;
```

Este comando también importará la información inicial del ejemplo previo de [Gestión de Stock](https://github.com/FIWARE/tutorials.Context-Providers) al inicio.

> :information_source: **Nota:** Si quieres limpiar y empezar de nuevo puedes hacerlo con el siguiente comando:
>
> ```console
> ./services stop
> ```

# Comunicando con los dispositivos IoT

Para seguir el tutorial correctamente, asegúrese de tener la página de monitorización del dispositivo en su navegador y haga clic en la página para habilitar el audio antes de introducir cualquier comando cUrl. El monitor de dispositivos muestra el estado actual de un conjunto de dispositivos ficticios usando la sintaxis de Ultralight 2.0

#### Monitor de dispositivos

El monitor de dispositivos se puede encontrar en la siguiente ruta: `http://localhost:3000/device/monitor`

Dentro de este tutorial jugarás el papel del componente del agente IoT que falta en el sistema, haciendo comandos hacia el sur a los dispositivos IoT asociados y recibiendo mediciones hacia el norte a medida que el entorno cambia dentro de la tienda. Todos los comandos se hacen como peticiones HTTP POST usando sintaxis Ultralight 2.0 y por lo tanto son muy simples. Merece la pena echar un vistazo a la página de monitorización del dispositivo ya que muestra todo el tráfico hacia el Norte generado por los propios dispositivos.

## Comandos de campana

Una campana - o **Bell** - es un ejemplo de actuador. Puede responder a comandos, pero no suministra ninguna medida del mundo real.

### Tocar una campana

Este ejemplo muestra cómo un verdadero agente IoT enviaría comandos a un actuador. La campana - **Bell** - ha provisto un endpoint, `/iot/bell001` donde escucha los comandos.

#### :one: Petición:

```console
curl -iX POST \
  --url 'http://localhost:3001/iot/bell001' \
  --data urn:ngsi-ld:Bell:001@ring
```

#### Respuesta:

```
urn:ngsi-ld:Bell:001@ring| ring OK
```
El cuerpo de la petición está en sintaxis Ultralight está formado por el `id` del dispositivo (`urn:ngsi-ld:Bell:001`) tal y como está representado en el Context Broker y el nombre del comando (`ring`) a invocar en el dispositivo.

La respuesta devuelve el comando y el resultado de la acción.

Si está viendo la página del monitor del dispositivo, puede ver el estado del cambio de la campana.

![](https://fiware.github.io/tutorials.IoT-Sensors/img/bell-ring.gif)

## Comandos de bombilla inteligente

La bombilla inteligente o **Smart Lamp** puede ser encendida y apagada a distancia - también registra la luminosidad. Contiene un sensor de movimiento en su interior y se atenuará lentamente a medida que pase el tiempo (siempre que no se detecte movimiento)

Las mediciones se enviarán al agente IoT a medida que cambie el estado y/o la luminosidad.

### Encender una bombilla inteligente

Este ejemplo muestra como un agente IoT real enviaría un comando Ultralight a una bombilla inteligente - **Smart Lamp** - para encenderla. La bombilla inteligente tiene un endpoint, `/iot/lamp001`, donde escucha para la recepción de comandos.

#### :two: Petición:

```console
curl -iX POST \
  --url 'http://localhost:3001/iot/lamp001' \
  --data urn:ngsi-ld:Lamp:001@on
```

El cuerpo de la petición está compuesto por el `id` del dispositivo (`urn:ngsi-ld:Lamp:001`) tal y como está representado en el Context Broker y el nombre del comando (`on`) a ejecutar en el dispositivo.

#### Respuesta:

La respuesta devuelve el comando y el resultado de la acción.

```
urn:ngsi-ld:Lamp:001@on| on OK
```

Una vez la lámpara se enciende, el nivel de luminosidad se altera dependiendo de si el sensor de movimiento interno detecta movimiento. La medición es enviada de forma activa y las peticiones al Context Broker pueden verse en la página del monitor del dispositivo.

### Apagar una bombilla inteligente

Este ejemplo muestra como un agente IoT real enviaría un comando Ultralight a una bombilla inteligente - **Smart Lamp** - para apagarla. La bombilla inteligente tiene un endpoint, `/iot/lamp001`, donde escucha para la recepción de comandos.

#### :three: Petición:

```console
curl -iX POST \
  --url 'http://localhost:3001/iot/lamp001' \
  --data urn:ngsi-ld:Lamp:001@off
```

El cuerpo de la petición está compuesto por el `id` del dispositivo (`urn:ngsi-ld:Lamp:001`) tal y como está representado en el Context Broker y el nombre del comando (`off`) a ejecutar en el dispositivo.

#### Respuesta:

La respuesta devuelve el comando y el resultado de la acción.

```
urn:ngsi-ld:Lamp:001@off| off OK
```

Una vez la lámpara es apagada, el nivel de luminosidad no se altera. La última medida Ultralight tal y como fue enviada enviada al Broker (`s|OFF|l|0`) se puede ver en la página de monitor de dispositivos.

Para encender la bombilla inteligente - **Smart Lamp** - de nuevo podemos repetir el siguiente comando:

#### :four: Petición:

```console
curl -iX POST \
  --url 'http://localhost:3001/iot/lamp001' \
  --data urn:ngsi-ld:Lamp:001@on
```

#### Respuesta:

```
urn:ngsi-ld:Lamp:001@on| on OK
```

## Comandos de puerta inteligente

Una puerta inteligente o **Smart Door** es una puerta electrónica a la que se le pueden enviar comandos para bloquearla y desbloqueaarla de forma remota. También puede reportar su propio estado (abierta - `OPEN`, cerrada - `CLOSED` o bloqueada -`LOCKED`),

Las mediciones se enviarán al agente IoT a medida que cambie el estado.

### Desbloquear una puerta inteligente

Este ejemplo enseña como un Agente IoT real enviaría un comando Ultralight a una puerta inteligente o **Smart Door** para desbloquear la puerta. Esta tiene un endpoint en `/iot/door001` donde escucha para la recepción de comandos.

#### :five: Petición:

```console
curl -iX POST \
  --url 'http://localhost:3001/iot/door001' \
  --data urn:ngsi-ld:Door:001@unlock
```

El cuerpo de la petición está compuesto por el `id` del dispositivo (`urn:ngsi-ld:Door:001`) tal y como está representado en el Context Broker y el nombre del comando (`unlock`) a ejecutar en el dispositivo.

#### Respuesta:

La respuesta devuelve el comando y el resultado de la acción.

```
urn:ngsi-ld:Door:001@unlock| unlock OK
```

Una vez la puerta inteligente - **Smart Door** - es desbloqueada, esta se abrirá y cerrará de forma automática conforme los clientes entren. Los cambios de estado son enviados de forma activa al broker IoT y el estado de la puerta inteligente se puede ver en la página de monitor de dispositivos.

El sensor de movimiento o **Motion Sensor** que hay en la tienda no es un actuador - no responde a ningún comando, sin embargo mide de forma activa el número de clientes que pasan por el. Si la puerta está desbloqueada, el sensor de movimiento detectará actividad y enviará medidas de vuelta al broker IoT.

Las peticiones enviadas hacia el norte, generadas por el sensor de movimiento, también se pueden ver en la página de monitor de dispositivos.

![](https://fiware.github.io/tutorials.IoT-Sensors/img/door-open.gif)

### Abrir una puerta

Este ejemplo enseña como un Agente IoT real enviaría un comando Ultralight a una puerta inteligente o **Smart Door** para abrir la puerta. Esta tiene un endpoint en `/iot/door001` donde escucha para la recepción de comandos.

#### :six: Petición:

```console
curl -iX POST \
  --url 'http://localhost:3001/iot/door001' \
  --data urn:ngsi-ld:Door:001@open
```

El cuerpo de la petición está compuesto por el `id` del dispositivo (`urn:ngsi-ld:Door:001`) tal y como está representado en el Context Broker y el nombre del comando (`open`) a ejecutar en el dispositivo.

#### Respuesta:

La respuesta devuelve el comando y el resultado de la acción.

```
urn:ngsi-ld:Door:001@open| open OK
```

El estado  de la puerta inteligente o **Smart Door** se puede ver a en la página de monitor de dispositivos. Los clientes pueden ahora entrar y el sensor de movimiento o **Motion Sensor** puede empezar a detectar clientes y enviar las medidas al broker IoT.

Las peticiones enviadas hacia el norte, generadas por la puerta inteligente - **Smart Door** - y el sensor de movimiento - **Motion Sensor** -, también se pueden ver en la página de monitor de dispositivos.

### Cerrar una puerta

Este ejemplo enseña como un Agente IoT real enviaría un comando Ultralight a una puerta inteligente o **Smart Door** para cerrar la puerta. Esta tiene un endpoint en `/iot/door001` donde escucha para la recepción de comandos.

#### :seven: Petición:

```console
curl -iX POST \
  --url 'http://localhost:3001/iot/door001' \
  --data urn:ngsi-ld:Door:001@close
```

El cuerpo de la petición está compuesto por el `id` del dispositivo (`urn:ngsi-ld:Door:001`) tal y como está representado en el Context Broker y el nombre del comando (`close`) a ejecutar en el dispositivo.

#### Respuesta:

La respuesta devuelve el comando y el resultado de la acción.

```
urn:ngsi-ld:Door:001@close| close OK
```

Dado que la puerta está todavía desbloqueada, los clientes pueden seguir entrado y abriendo la puerta por su cuenta. Si hay algún tipo de movimiento, el sensor de movimiento **Motion Sensor** enviará las medidas al broker IoT.

Las peticiones enviadas hacia el norte, generadas por el sensor de movimiento - **Motion Sensor** -, también se pueden ver en la página de monitor de dispositivos.

### Bloquear una puerta

Este ejemplo enseña como un Agente IoT real enviaría un comando Ultralight a una puerta inteligente o **Smart Door** para bloquear la puerta. Esta tiene un endpoint en `/iot/door001` donde escucha para la recepción de comandos.


#### :eight: Petición:

```console
curl -iX POST \
  --url 'http://localhost:3001/iot/door001' \
  --data urn:ngsi-ld:Door:001@lock
```

El cuerpo de la petición está compuesto por el `id` del dispositivo (`urn:ngsi-ld:Door:001`) tal y como está representado en el Context Broker y el nombre del comando (`lock`) a ejecutar en el dispositivo.

#### Respuesta:

La respuesta devuelve el comando y el resultado de la acción.

```
urn:ngsi-ld:Door:001@close| close OK
```

Una vez la puerta está bloqueada, no pueden entrar más clientes. El sensor de movimiento - **Motion Sensor** - no enviará ningún otra medida, la puerta inteligente - **Smart Door** - no podrá ser abierta manualemnte y la bombilla inteligente - **Smart Lamp** - volverá lentamente al nivel de luminosidad ambiente.

Las peticiones enviadas hacia el norte, generadas por la bombilla inteligente - **Smart Lamp** -, también se pueden ver en la página de monitor de dispositivos.

![](https://fiware.github.io/tutorials.IoT-Sensors/img/door-lock.gif)

# Próximos pasos

Quieres aprender cómo añadir más complejidad a tu aplicación añadiendo funcionalidades avanzadas? Puedes descubrirlas leyendo otros  [tutoriales de esta serie](https://fiware-tutorials.rtfd.io)

---

## License

[MIT](LICENSE) © 2018-2020 FIWARE Foundation e.V.
