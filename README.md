<a href="https://www.gotoiot.com/">
    <img src="doc/gotoiot-logo.png" alt="logo" title="Goto IoT" align="right" width="60" height="60" />
</a>

Service AMQP Broker
===================

*Ayudaría mucho si apoyaras este proyecto con una ⭐ en Github!*

AMQP es un protocolo de colas que define el comportamiento de un servidor basado en exchanges y queues, que permite vincular a diferentes aplicaciones en múltiples lenguajes de programación, tanto internas como de terceros, mediante un `Mensaje AMQP` que representa la unidad de información a intercambiar.

RabbitMQ es un broker que implementa la especificación `AMQP 0-9-1`, y además de soportar el comportamiento estándar, posee extensiones a modo plugins donde se pueden interconectar diferentes protocolos como MQTT, MQTT sobre WebSockets, STOMP, HTTP, y otros. Así mismo, cuenta con plugins que permiten intercomunicar brokers entre sí, pudiendo armar clusters, federaciones y reenvío de entidades particulares. Además cuenta con un administrador web que lo hace muy conveniente para configurarlo.

Este servicio, además de soportar el protocolo `AMQP 0-9-1` hace uso del plugin `MQTT` y `Web MQTT`, que va a correr un broker MQTT asociado al broker RabbitMQ y te va a permitir conectarte mediante clientes MQTT como microcontroladores, asi como también con clientes web, desde el navegador. Realizando esta integración se tiene un puente entre clientes MQTT con un protocolo altamente escalable, donde podrá intercambiar información con otros servicios y dispositivos.

En la configuración por defecto de este servicio se tienen habilitados una serie de plugins que lo hacen muy conveniente para el desarrollo de aplicaciones. Por un lado, con el plugin `rabbitmq_management` se habilita el administrador web del broker y también una interfaz HTTP para realizar las configuraciones mediante su REST API. Con el servicio `rabbitmq_mqtt` y `rabbitmq_web_mqtt` se agrega al broker RabbitMQ un broker MQTT que permite conectar clientes en texto plano, por WebSockets y por SSL. Con los plugin `rabbitmq_federation` y `rabbitmq_federation_management` se habilita dentro del broker la posibilidad de replicar mensajes que se publican en exchanges remotos. Con los plugin `rabbitmq_shovel` y `rabbitmq_shovel_management` es posible tomar datos de un exchange o queue local y replicarlos en un exchange o queue remoto. 

Las configuraciones de los plugins y sus funcionamiento están detallados en cada una de las secciones de documentación correspondientes, pero para que tengas una idea, en esta imagen podés ver el diagrama de las funcionalidades habilitadas en el broker.

![rabbitmq_layout](doc/rabbitmq_plugins.png)

> Para que entiendas el alcance de este proyecto, es recomendable que leas los artículos de [Introducción a AMQP](https://www.gotoiot.com/pages/articles/amqp_intro/index.html), [Introducción a RabbitMQ](https://www.gotoiot.com/pages/articles/rabbitmq_intro/index.html) y [RabbitMQ Distribuido](https://www.gotoiot.com/pages/articles/rabbitmq_distribuited/index.html) que se encuentran publicados en nuestra web.

## Instalar las dependencias 🔩

Para correr este proyecto es necesario que instales `Docker` y `Docker Compose`. 

<details><summary><b>Mira cómo instalar las dependencias</b></summary><br>

En [este documento](https://www.gotoiot.com/pages/articles/docker_installation/index.html) publicado en nuestra web están los detalles para instalar Docker y Docker Compose. Si querés instalar ambas herramientas en una Raspberry Pi podés seguir [esta guía](https://devdojo.com/bobbyiliev/how-to-install-docker-and-docker-compose-on-raspberry-pi) que muestra todos los detalles de instalación.

En caso que tengas algún incoveniente o quieras profundizar al respecto, podes leer la documentación oficial de [Docker](https://docs.docker.com/get-docker/) y también la de [Docker Compose](https://docs.docker.com/compose/install/).

</details>

## Descargar el código 💾

Para descargar el código, lo más conveniente es que realices un `fork` de este proyecto a tu cuenta personal haciendo click en [este link](https://github.com/gotoiot/service-amqp-broker/fork). Una vez que ya tengas el fork a tu cuenta, descargalo con este comando (acordate de poner tu usuario en el link):

```
git clone https://github.com/USER/service-amqp-broker.git
```

> En caso que no tengas una cuenta en Github podes clonar directamente este repo.

## Ejecutar la aplicación 🚀

Cuando tengas el código descargado, desde una terminal en la raíz del proyecto ejecuta el comando `docker-compose up -d` que se va descargar la imagen de Docker del broker y ponerlo en marcha. 

Una vez que el broker inicie, espera unos momentos a que se realicen las configuraciones iniciales, y luego desde un navegador ingresa en la URL [http://localhost:15672](http://localhost:15672) que te va a llevar al portal de administración de RabbitMQ. Accede con el usuario `gotoiot` y contraseña `gotoiot`. Si pudiste accceder significa que el broker se encuentra funcionando correctamente.

Continua explorando la Información Util del proyecto para concer más detalles.

## Información útil 🔍

En esta sección vas a encontrar información que te va a servir para tener un mayor contexto.

<details><summary><b>Accedé a todos los detalles importantes</b></summary>

### Índice de la sección

* [Configuración del servicio](#configuración-del-servicio)
* [Definiciones en el broker](#definiciones-en-el-broker)
* [Producir y consumir mensajes](#producir-y-consumir-mensajes)
* [Conexión por MQTT plano](#conexión-por-mqtt-plano)
* [Conexión por MQTT por WebSockets](#conexión-mqtt-por-websockets)
* [Conexión por HTTP](#conexión-por-http)
* [Ejecutar comandos dentro del broker](#ejecutar-comandos-dentro-del-broker)
* [Configuración de federación](#configuración-de-federación)
* [Configuración de Shovel](#configuración-de-shovel)

### Configuración del servicio

El archivo `docker-compose.yml` administra los parámetros generales de ejecución del broker. Está basado en la imagen oficial de `RabbitMQ` y soporta la conexión con el protocolo AMQP en el binding de puertos 5672:5672, la comunicación por MQTT en 1883:1883, MQTT por WebSockets en 9001:9001 y la comunicación para el administrador del broker por HTTP en el puerto 15672:15672. Así mismo, el broker viene con unos ejemplos para WebSockets configurados en el binding de puertos 9002:9002.

También, dentro del archivo `docker-compose.yml` se definen los bind volumes que se comparten con el broker. Todos se encuentran mapeados dentro del directorio `rabbitmq` y se definen de la siguiente manera:

* **enable_plugins:** En este archivo se pueden especificar los plugins habilitados por el broker. Si querés saber más al respecto podés ingresar a [este link](https://www.rabbitmq.com/plugins.html).
* **rabbitmq-env.conf**: Este es el archivo donde se comparten las variables de entorno con las que inicia el broker. Si querés saber más al respecto podés ingresar a [este link](https://www.rabbitmq.com/configure.html#customise-environment).
* **rabbitmq.conf**: Este es el archivo donde se realiza la configuración específica del broker. Para este proyecto mayormente se realiza la configuración para MQTT y también desde qué path tomar las definiciones. Si querés saber más al respecto podés ingresar a [este link](https://www.rabbitmq.com/configure.html).
* **definitions.json**: Este archivo permite crear las definiciones de todo el broker antes de comenzar su ejecución y sin tener que hacerlo manualmente. Si querés saber más al respecto podés ingresar a [este link](https://github.com/tyranron/lapin-issue-133-example/blob/master/rabbitmq-definitions.json).

### Definiciones en el broker

Tal como vimos, el archivo `definitions.json` tiene toda la declaración de entidades, usuarios, permisos, exchanges, queues y bindings, que se realizan de manera automática al iniciar el broker. Esta característica resulta realmente útil para compartir la información, por lo que es recomendable que siempre que quieras realizar un proyecto lo tengas en cuenta y trates de realizarla mediante este archivo.

En [este link](https://github.com/tyranron/lapin-issue-133-example/blob/master/rabbitmq-definitions.json) podés ver un ejemplo completo de definiciones que lo podés tomar como punto de partida para realizar tus configuraciones. Este proyecto trae algunas definiciones preestablecidas, y podés modificarla a tus necesidades editando el archivo `definitions.json`.

### Producir y consumir mensajes

Para poder realizar una comunicación entre un productor y un consumidor es necesario que el productor se conecte a un exchange, un consumidor a una queue, y que haya un binding que vincule estas dos entidades.

Para este ejemplo vamos a utilizar el exchange que se crea por defecto `amq.topic` (un exchange basado en topic), una queue que se llame `events`, y un binding que vincule el exchange `amq.topic` con la queue `events` utilizando la routing key `event.#` que permitira recibir cualquier tipo de eventos que comiencen con `event.`, como por ejemplo `event.alarm`, `event.user`, pero no algo como `user.logout`.

Como primera medida debés logearte en el [admin de RabbitMQ](http://localhost:15672) con el usuario y contraseña que figuran en el archivo `definitions.json` (el usuario por defecto es `gotoiot` y contraseña `gotoiot`). Luego accedé a la pestaña `Queues` en la parte superior.

Dentro de la pestaña `Queues`, en la opción `Add a new queue` ingresa los datos como se muestran en esta imagen.

![queue-create](doc/queue-create.png)

Luego, anda a la pestaña de `Exchanges`, y en la lista de exchanges disponibles hacé click sobre el exchange `amq.topic`. Dentro de la descripción del exchange, anda a la opción `Add binding from this exchange` y realiza la siguiente configuración.

![bind-create](doc/bind-create.png)

Al realizar el paso anterior, dentro de los bindings deberías ver el que acabas de realizar, como en esta imagen. Tene en cuenta que podés ver mas de una queue asociada a un exchange.

![bind-show](doc/bind-show.png)

Ahora que realizaste la configuración podés enviar mensajes al exchange. Dentro de la sección `Exchanges->Publish Message` realiza el envío de un mensaje como este.

![message-create](doc/message-create.png)

Luego, anda a la pestaña Queues, y en la sección Get Messages presioná el botón para obtener los mensajes. Deberías ver una imagen como la siguiente.

![message-show](doc/message-show.png)

### Conexión por MQTT plano

La conexión por MQTT se realiza mediante el `plugin oficial de RabbitMQ`. Es recomendable que leas [la documentación](https://www.rabbitmq.com/mqtt.html) para entender cómo trabaja. 

**Funcionamiento del plugin**

Para que tengas un poco de contexto respecto al funcionamiento del plugin, podemos decir que soporta MQTT 3.1.1, los niveles de calidad de servicio QoS0 y QoS1 (los mensajes con QoS2 son pasados a QoS1 automáticamente), los mensajes LWT (Last Will and Testament), TLS y retención de mensajes. Utilizando el plugin de MQTT es posible interactuar con otros clientes con AMQP 0-9-1, AMQP 1.0 y STOMP.

Para habilitarlo podés hacerlo de al menos dos maneras. Ingresando en ejecutar comandos dentro del broker (como está mostrado en la sección `Ejecutar comandos`) y corriendo el comando `rabbitmq-plugins enable rabbitmq_mqtt` o bien asegurando que se encuentre dentro del archivo `enable_plugins`.

Para darle una capa de seguridad al broker, será necesario que crees un usuario y una contraseña correspondiente para que puedan conectarse los clientes por MQTT. En la [sección de autenticación](https://www.rabbitmq.com/mqtt.html#authentication) de la documentación oficial, se muestran los comandos para crear un usuario afin.

El plugin está creado sobre las entidades core (exchanges y queues) de RabbitMQ. Los mensajes publicados usando MQTT son mapeados internamente al exchange `amq.topic` creado por defecto. Los suscriptores - tanto MQTT como otros - consumen de las colas de RabbitMQ vinculadas al exchange `amq.topic`. Esto permite la interoperabilidad con otros protocolos y hace posible usar el panel de administración para inspeccionar las colas correspondientes.

Tené en cuenta que MQTT utiliza barras inclinadas ("/") para separadores de topics y AMQP 0-9-1 utiliza puntos. El plugin MQTT traduce estos patrones hacia ambos lados automáticamente. Por ejemplo, `sensor/humidity` se convierte en `sensor.humidity` y viceversa. Hay que tener en cuenta una advertencia: evita usa el caracter `"/"` y `"."` en ambos protocolos, ya que se pueden generar comportamientos inesperados.

**Configuración del plugin en este servicio**

Utilizando parte de las características por defecto, este servicio está pre configurado para reenviar los topics que llegan por MQTT hacia el exchange por defecto `amq.topic`; del mismo todo, todo lo que se publica en el exchange `amq.topic` que concide con la suscripción MQTT es enviado hacia los clientes respectivos. 

Para conectarse al broker es necesario utilizar el usuario y contraseña definidos en las variables `mqtt.default_user` y `mqtt.default_pass` en el archivo `rabbitmq.config`. Así mismo existen otras configuraciones útiles que resultan interesantes destacar. Por ejemplo el plug `mqtt.allow_anonymous` permite que se conecten usuarios sin autenticación. Para darle un punto de seguridad al broker, en este servicio esa funcionalidad está deshabilitada.

Para realizar alguna modificación en particular, como por ejemplo el exchange al cual se conectan los topics MQTT, los puertos por defecto, usuarios, y otros, directamente edita el archivo `rabbitmq/rabbitmq.config`.

**Ejemplo de comunicación**

En este ejemplo te vamos a mostrar como realizar una suscripción y publicación por MQTT usando los `Mosquitto Clients` del broker [Mosquitto](https://www.mosquitto.org) mediante un contenedor de docker. Las credenciales de acceso son las por defecto del archivo de configuración.

Abrí una terminal y ejecutá este comando para suscribirte a todos eventos (`event/#`).

```
docker run --rm --net host eclipse-mosquitto \
mosquitto_sub -h localhost -p 1883 -u gotoiot -P gotoiot -t event/#
```

Luego, desde otra terminal corré el siguiente comando para publicar un topic `event/failure` con el payload `'{"sensor_connected": false}'`.

```
docker run --rm --net host eclipse-mosquitto \
mosquitto_pub -h localhost -p 1883 -u gotoiot -P gotoiot -t event/failure -m '{"sensor_connected": false}'
```

### Conexion MQTT por WebSockets

Otra funcionalidad importante del servicio, es que está configurado para poder conectarse al broker MQTT mediante WebSockets. Esto es una gran ventaja, ya que habilita a aplicaciones web a tener comunicación con MQTT y con el ecosistema RabbitMQ.

Para esta funcionalidad se utiliza el [plugin Web MQTT](https://www.rabbitmq.com/web-mqtt.html) provisto por el core de RabbitMQ. El puerto de conexión MQTT por WebSockets es el 9001, al cual es necesario acceder con el usuario y contraseña. Tanto la configuración del puerto para WebSockets como el usuario y contraseña se encuentran en el archivo `rabbit/rabbitmq.config`. 

Para que puedas realizar una prueba de comunicación MQTT por WebSockets podés utilizar el proyecto [Web MQTT Client](https://github.com/gotoiot/web-mqtt-client) de nuestra organización, que es una terminal web donde podés configurar el broker, el puerto, usuario, y contraseña. En la siguiente imagen podés ver una configuración de la herramienta donde se suscribe a un topic y luego se envía, mostrando la información en pantalla.

![mqtt-websocket-demo](doc/mqtt-websocket-demo.png)

### Conexión por HTTP

En muchos casos puede resultar particularmente útil conectarse al broker RabbitMQ por HTTP. Esto va a permitir conectar clientes que no soporten nativamente las bibliotecas AMQP, o bien establecer una comunicación desde un navegador web, ya que actualmente no se cuenta con soporte web nativo para AMQP.

Para poder conectarte por HTTP al broker, es necesario que esté habilitado el `plugin rabbitmq_management` que es el mismo que te permite acceder al panel de administración. Es decir, que todos los comandos y acciones que se pueden realizar desde el panel de administración las podrías realizar desde cualquier cliente HTTP.

Para demostrarte el funcionamiento por HTTP vamos a hacer uso de la herramienta `cURL` ampliamente utilizada, la cual la vamos a ejecutar mediante un contenedor de Docker. 

Como primera medida vamos a crear un exchange llamado `gotoiot.http` del tipo `direct`, con el usuario y contraseña `gotoiot:gotoiot`, y además algunos settings necesarios para la creación.

```
docker run --rm --net host curlimages/curl curl \
-i -u gotoiot:gotoiot \
-H "content-type:application/json" \
-X POST http://localhost:15672/api/exchanges/%2f/gotoiot.http \
-d '{"type": "direct", "auto_delete": false, "durable": true, "internal": false, "arguments": {}}'
```

El siguiente paso será crear una queue llamada `http.queue` con los settings necesarios para la creación.

```
docker run --rm --net host curlimages/curl curl \
-i -u gotoiot:gotoiot \
-H "content-type:application/json" \
-X PUT http://localhost:15672/api/queues/%2f/http.queue \
-d '{"auto_delete": false, "durable": true, "arguments": {}}'
```

Ahora que ya contamos con el exchange y la queue, vamos a crear un binding que una a ambas entidades utilizando la routing_key `event`.

```
docker run --rm --net host curlimages/curl curl \
-i -u gotoiot:gotoiot \
-H "content-type:application/json" \
-X POST http://localhost:15672/api/bindings/%2f/e/gotoiot.http/q/http.queue \
-d '{"routing_key": "events", "arguments": {}}'
```

Ahora que ya contamos con el exchange, la queue y el binding, enviemos un mensaje al exchange `gotoiot.http`, con la routing key `event` y el mensaje `USER_REGISTRATION`.

```
docker run --rm --net host curlimages/curl curl \
-i -u gotoiot:gotoiot \
-H "content-type:application/json" \
-X POST http://localhost:15672/api/exchanges/%2f/gotoiot.http/publish \
-d '{"routing_key": "events", "payload": "USER_REGISTRATION", "payload_encoding": "string", "properties": {}}'
```

Finalmente, una vez que se haya enviado el mensaje es momento de consumirlo. Para ello vamos a obtener de la HTTP API los datos de la queue `http.queue`.

```
docker run --rm --net host curlimages/curl curl \
-i -u gotoiot:gotoiot \
-H "content-type:application/json" \
-X POST http://localhost:15672/api/queues/%2f/http.queue/get \
-d '{"count": 10, "requeue": true, "encoding": "auto", "truncate": 50000, "ackmode": "ack_requeue_false"}'
```

Con los pasos anteriores se demuestra la comunicación entre un cliente HTTP y la REST API del administrador de RabbitMQ. Estas mismas interacciones podés realizarla desde cualquier cliente HTTP en cualquier lenguaje de programación. De todas maneras, tené en cuenta que esta API no está pensada para un alto intercambio de mensajes, sino más bien para integrar clientes que no soportan el protocolo AMQP o MQTT de manera nativa.

Si querés ver los detalles completos de la REST API del administrador de RabbitMQ podés ingresar en [este link](https://pulse.mozilla.org/api/).

### Ejecutar comandos dentro del broker

Si vas a realizar configuraciones en particular, es común ejecutar comandos dentro del broker, que realizan la misma acción que desde el panel de administración.

Para correr cualquier comando, primero necesitas saber el nombre del container del servicio, para ello, podes ejecutar el comando `docker ps` y ver su nombre. Luego, una vez que sepas el nombre corre el comando `docker exec -it CONTAINER_NAME /bin/sh` para ingresar dentro del contenedor.

En este ejemplo, podés ver los pasos necesarios para crear un usuario llamado `myuser` con contraseña `mypass`, con permisos de administrador del sistema.

```sh
rabbitmqctl add_user myuser mypass
rabbitmqctl set_permissions -p / myuser ".*" ".*" ".*"
rabbitmqctl set_user_tags myuser management administrator
```

### Configuración de federación

La federación se encarga de tomar los datos remotos provenientes de queues o exchanges remotos y replicarlos de manera local. 

La federación posibilita la comunicación - llamada upstream - desde broker remotos al local sin necesidad de configuración en el broker remoto. Esto permite que las federaciones sean creadas a demanda, a medida que nuevos dispositivos edge se unen a la red, y permitiendo que las conexiones puedan ser automatizadas.

Para la configuración de la federación es necesario que estén habilitados los plugins `rabbitmq_federation` y `rabbitmq_federation_management` en cada broker local. Esto puede ser realizado desde la línea de comandos del broker, o bien agregando ambos plugins a la lista en el archivo `enable_plugins` del broker. 

Una vez habilitados los plugins, en la sección de `Admin -> Federation Upstream` deberías cargar los datos del broker remoto. Para ello, sólo necesitas cargar la URL del broker remoto y el nombre que le vas a poner a la federación, los demás datos podés dejarlos en blanco.

Luego de crear la federación, es necesario crear una policy dentro del broker para indicar que todos los exchanges/queues que estén declarados remotamente que coincidan con el patron X, sean replicados en cada broker edge. Recordá agregar la definición `federation-upstream-set` igual a `all` en las properties.

Una vez declarada la policy, en la sección `Admin -> Federation Status` deberías ver el estado de la conexión de la federación.

### Configuración de Shovel

La funcionalidad del plugin shovel es recibir mensajes de una fuente y publicarlos a un destino. Esta es una necesidad común a la hora de trabajar con brokers distribuidos, y el plugin de shovel cumple a la perfección con esta tarea, ya que es un cliente realizado por el core de RabbitMQ diseñado para tal fin.

Para la configuración del shovel es necesario que estén habilitados los plugins `rabbitmq_shovel` y `rabbitmq_shovel_management`. Esto puede ser realizado desde la línea de comandos del broker, o bien agregando ambos plugins a la lista en el archivo `enable_plugins` del broker. 

Una vez habilitados los plugins, en la sección de `Admin -> Shovel Management` deberías cargar los datos del source y destination. En el source deberías poner la URI del broker local y el exchange o queue correspondientes. Para la parte de destino deberías cargar la URI del broker remoto y el exchange o queue correspondientes.

Una vez declarados, en la sección `Admin -> Shovel Status` deberías ver el estado de la conexión del plugin.


</details>

## Tecnologías utilizadas 🛠️

<details><summary><b>Mira la lista de tecnologías usadas en el proyecto</b></summary><br>

* [Docker](https://www.docker.com/) - Ecosistema que permite la ejecución de contenedores de software.
* [Docker Compose](https://docs.docker.com/compose/) - Herramienta que permite administrar múltiples contenedores de Docker.
* [RabbitMQ](https://rabbitmq.com/) - Broker AMQP libre y abierto ampliamente utilizado.

</details>

## Contribuir 🖇️

Si estás interesado en el proyecto y te gustaría sumar fuerzas para que siga creciendo y mejorando, podés abrir un hilo de discusión para charlar tus propuestas en [este link](https://github.com/gotoiot/service-amqp-broker/issues/new). Así mismo podés leer el archivo [Contribuir.md](https://github.com/gotoiot/gotoiot-doc/wiki/Contribuir) de nuestra Wiki donde están bien explicados los pasos para que puedas enviarnos pull requests.

## Sobre Goto IoT 📖

Goto IoT es una plataforma que publica material y proyectos de código abierto bien documentados junto a una comunidad libre que colabora y promueve el conocimiento sobre IoT entre sus miembros. Acá podés ver los links más importantes:

* **[Sitio web](https://www.gotoiot.com/):** Donde se publican los artículos y proyectos sobre IoT. 
* **[Github de Goto IoT:](https://github.com/gotoiot)** Donde están alojados los proyectos para descargar y utilizar. 
* **[Comunidad de Goto IoT:](https://groups.google.com/g/gotoiot)** Donde los miembros de la comunidad intercambian información e ideas, realizan consultas, solucionan problemas y comparten novedades.
* **[Twitter de Goto IoT:](https://twitter.com/gotoiot)** Donde se publican las novedades del sitio y temas relacionados con IoT.
* **[Wiki de Goto IoT:](https://github.com/gotoiot/doc/wiki)** Donde hay información de desarrollo complementaria para ampliar el contexto.

## Muestas de agradecimiento 🎁

Si te gustó este proyecto y quisieras apoyarlo, cualquiera de estas acciones estaría más que bien para nosotros:

* Apoyar este proyecto con una ⭐ en Github para llegar a más personas.
* Sumarte a [nuestra comunidad](https://groups.google.com/g/gotoiot) abierta y dejar un feedback sobre qué te pareció el proyecto.
* [Seguirnos en twitter](https://github.com/gotoiot/doc/wiki) y dejar algún comentario o like.
* Compartir este proyecto con otras personas.

## Autores 👥

Las colaboraciones principales fueron realizadas por:

* **[Agustin Bassi](https://github.com/agustinBassi)**: Ideación, puesta en marcha y mantenimiento del proyecto.

También podés mirar todas las personas que han participado en la [lista completa de contribuyentes](https://github.com/gotoiot/service-amqp-broker/contributors).

## Licencia 📄

Este proyecto está bajo Licencia ([MIT](https://choosealicense.com/licenses/mit/)). Podés ver el archivo [LICENSE.md](LICENSE.md) para más detalles sobre el uso de este material.

---

**Copyright © Goto IoT 2021** - [**Website**](https://www.gotoiot.com) - [**Group**](https://groups.google.com/g/gotoiot) - [**Github**](https://www.github.com/gotoiot) - [**Twitter**](https://www.twitter.com/gotoiot) - [**Wiki**](https://github.com/gotoiot/doc/wiki)
