iCub
=================

En esta sección se procederá a explicar los tutoriales disponibles como forma de abordarlos de manera sencilla.

## Trabajando con puertos en YARP

La comunicación en YARP se resuelve siguiente un patrón de diseño del observador o puerto. Estos **puertos especiales** llevan mensajes a cualquier número de observadores (otros puertos), en cualquier número de procesos, distribuido a través de cualquier número de máquinas, mediante cualquiera de los protocolos de comunicación soportados.


Aquí un ejemplo de una red de puertos YARP:

<p align="center">
  <img src="/img/yarp_ports.png"/>
</p>


Todos los puertos pertenecen a un proceso, pero no necesariamente al mismo. Además, pueden coexistir diversos protocolos de comunicación y/o redes físicas.

Características por protocolo:

* TCP → Confiable. Garantía de recepción de un mensaje.
* UDP → Más rápido que TCP pero sin garantías.
* multicast → Eficiente para transmisión de un mismo mensaje a múltiples objetivos.
* Shared memory → Usado en conexiones locales (No es necesario intervención de programación para hacer las conexiones). Usado para monitoreo, es fácil de traducir a texto simple.

### Ejemplo de trabajo

Para poder inicializarse en el trabajo con los puertos de comunicación mediante YARP se propone el siguiente ejemplo.

El primer paso es :warning:**IMPORTANTE** debido a que cada trabajo a desarrollarse, tanto en simulación como en el robot físico, es importante tener inicializado el servidor de YARP.

Para esto, se corre el siguiente comando:

```
[en terminal 1] yarpserver
```

Siguiendo con el ejemplo, en un segundo terminal se abre un puerto de lectura:

```
[en terminal 2] yarp read /read
```

El nombre del puerto siempre será con la sintaxis ```/NombrePuerto``` y este puede ser cualquiera, sin embargo, se recomienda que se le asigne un nombre relacionado con su funcionalidad.

En un tercer terminal se abre un puerto de escritura:

```
[en terminal 3] yarp write /write
```

En un cuarto terminal se ejecuta el siguiente comando de conexión:

```
[en terminal 4] yarp connect /write /read
```

Aquí hacemos referencia a los puertos por sus nombres, incluyendo el caracter ```/```


La conexión se vería de la siguiente forma:

<p align="center">
  <img src="/img/puertosConectados.png"/>
</p>


Ahora, se implementará un código que interactúa con estos puertos, añadiendo números al puerto de lectura.


```c++
/*
 * SPDX-FileCopyrightText: 2006-2021 Istituto Italiano di Tecnologia (IIT)
 * SPDX-FileCopyrightText: 2006-2010 RobotCub Consortium
 * SPDX-License-Identifier: BSD-3-Clause
 */
 
#include <yarp/os/Bottle.h>
#include <yarp/os/BufferedPort.h>
#include <yarp/os/LogStream.h>
#include <yarp/os/Network.h>
 
#include <iostream>
 
using yarp::os::Bottle;
using yarp::os::BufferedPort;
using yarp::os::Network;
 
int main(int argc, char* argv[])
{
    YARP_UNUSED(argc);
    YARP_UNUSED(argv);
 
    Network yarp;
    BufferedPort<Bottle> port;
    port.open("/summer");
    while (true) {
        yInfo() << "waiting for input";
        Bottle* input = port.read();
        if (input != nullptr) {
            yInfo() << "got " << input->toString().c_str();
            double total = 0;
            for (size_t i = 0; i < input->size(); i++) {
                total += input->get(i).asFloat64();
            }
            Bottle& output = port.prepare();
            output.clear();
            output.addString("total");
            output.addFloat64(total);
            yInfo() << "writing " << output.toString().c_str();
            port.write();
        }
    }
    return 0;
}
```

Para compilar y ejecutar este programa debemos crear un archivo ```makefile```. Este archivo variará en contenido dependiendo de las necesidades y requerimientos del código que se quiere implementar.

:warning: **NOTA:** El archivo **siempre** se llamará ```CMakeLists.txt```.


El contenido será el siguiente:

```
project(SUMMER)
 
find_package(YARP REQUIRED)
 
add_executable(summer main.cpp)
 
# we now add the YARP libraries to our project.
target_link_libraries(summer ${YARP_LIBRARIES})
```

El proceso de compilación y ejecución es el siguiente teniendo este archivo:


```
[en terminal 5] cmake ./ 
[en terminal 5] make
```

Este proceso creará un archivo ejecutable con el nombre asignado en el apartado ```add_executable()``` donde también se ingresa el nombre de todos los archivos ```.cpp``` presentes.


Ahora que se tiene todas las herramientas, la forma de hacer la conexión se puede desarrollar de la siguiente forma:


```
[en terminal 5] ./summer
[en terminal 4] yarp connect /write /summer
[en terminal 4] yarp connect /summer /read

```
Esto modifica nuestra red de puertos de la siguiente forma:

<p align="center">
  <img src="/img/puertosConSummer.png"/>
</p>


El código ```/summer``` calculará la suma de los números ingresados en write, por lo que en el terminal ```/read``` se verá algo así

```
10 -5 17.15 6
total 28.15
```

Si queremos desconectar el acceso directo que tiene ```/write``` con ```/read```, simplemente ejecutamos lo siguiente

```
yarp disconnect /write /read
```

Por otro lado, el protocolo de comunicación por defecto es ```tcp```pero si quisieramos modificarlo simplemente hacemos lo siguiente:

```
yarp disconnect /summer /read
yarp connect /summer /read udp
```

Lo que setea la conexión asignada con protocolo ```udp```, y funciona similar para el resto de protocolos.

### Closer look

Si queremos información sobre un puerto podemos consultarlo en la terminal

```
[en terminal] yarp name query /summer
[respuesta] registration name /summer ip 5.255.222.252 port 10012 type tcp
	*** end of message
```

En este caso obtendremos la dirección ip, el número de puerto y el protocolo utilizado.

Ahora, para saber qué protocolos acepta un puerto, se puede consultar en la terminal

```
[en terminal] yarp name get /summer accepts
[respuesta] port /summer property accepts = tcp text text_ack udp mcast shmem
	*** end of message
```


## Puertos especializados RPC


 ```RPC``` se refiere a Remote Procedure Call, y es referido a un tipo de comunicación que consiste en envío de mensaje y una respuesta a este.

### RPC utilizando puertos especializados de YARP

Para la utilización de estos puertos, se utilizan 2 clases, ```yarp::os::RpcClient``` y ```yarp::os::RpcServer```, creando una conexión cliente-servidor.


Estos puertos permiten una interacción del código que entrega más información sobre lo que ocurre con este mientras se está ejecutando.

Para trabajar con puertos del tipo cliente y del tipo servidor, se tiene este primer ejemplo:

* Servidor

```cpp
/*
 * SPDX-FileCopyrightText: 2006-2021 Istituto Italiano di Tecnologia (IIT)
 * SPDX-FileCopyrightText: 2006-2010 RobotCub Consortium
 * SPDX-License-Identifier: BSD-3-Clause
 */
 
/*
 * This is an example of using a specialized RpcServer port to receive
 * and reply to messages.  Regular YARP ports can do this as well (see
 * summer.cpp), but use of RpcServer/RpcClient allows for better
 * run-time checking of port usage to catch mistakes.
 */
 
#include <yarp/os/Bottle.h>
#include <yarp/os/Network.h>
#include <yarp/os/RpcServer.h>
#include <yarp/os/Time.h>
 
#include <cstdio>
 
using yarp::os::Bottle;
using yarp::os::Network;
using yarp::os::RpcServer;
 
using namespace yarp::os;
 
int main(int argc, char* argv[])
{
    if (argc < 2) {
        fprintf(stderr, "Please supply a port name for the server\n");
        return 1;
    }
 
    const char* name = argv[1];
 
    Network yarp;
    RpcServer port;
    port.open(name);
 
    while (true) {
        printf("Waiting for a message...\n");
        Bottle cmd;
        Bottle response;
        port.read(cmd, true);
        printf("Got message: %s\n", cmd.toString().c_str());
        response.addString("you");
        response.addString("said");
        response.append(cmd);
        printf("Sending reply: %s\n", response.toString().c_str());
        port.reply(response);
    }
}
```

* Cliente

```cpp
/*
 * SPDX-FileCopyrightText: 2006-2021 Istituto Italiano di Tecnologia (IIT)
 * SPDX-FileCopyrightText: 2006-2010 RobotCub Consortium
 * SPDX-License-Identifier: BSD-3-Clause
 */
 
/*
 * This is an example of using a specialized RpcClient port to send
 * messages and receive replies.  Regular YARP ports can do this as well,
 * but use of RpcServer/RpcClient allows for better
 * run-time checking of port usage to catch mistakes.
 */
 
#include <yarp/os/Bottle.h>
#include <yarp/os/Network.h>
#include <yarp/os/RpcClient.h>
#include <yarp/os/Time.h>
 
#include <cstdio>
 
using yarp::os::Bottle;
using yarp::os::Network;
using yarp::os::RpcClient;
 
int main(int argc, char* argv[])
{
    if (argc < 3) {
        fprintf(stderr, "Please supply (1) a port name for the client\n");
        fprintf(stderr, "              (2) a port name for the server\n");
        return 1;
    }
 
    Network yarp;
    const char* client_name = argv[1];
    const char* server_name = argv[2];
 
    RpcClient port;
    port.open(client_name);
 
    int ct = 0;
    while (true) {
        if (port.getOutputCount() == 0) {
            printf("Trying to connect to %s\n", server_name);
            yarp::os::Network::connect(client_name, server_name);
        } else {
            Bottle cmd;
            cmd.addString("COUNT");
            cmd.addInt32(ct);
            ct++;
            printf("Sending message... %s\n", cmd.toString().c_str());
            Bottle response;
            port.write(cmd, response);
            printf("Got response: %s\n", response.toString().c_str());
        }
        yarp::os::Time::delay(1);
    }
}
```

La comunicación cliente-servidor se efectúa mediante la instancia de objetos del tipo ```yarp::os::Bottle```.

### Monitoreo comunicación RPC

El monitoreo general de puertos consta de conectar simplemente un puerto al emisor. Para la comunicación RPC es distinto.

Funcionamiento de server y client:

```
[en terminal 1] ./rpc_server /server
[en terminal 2] ./rpc_client /client /server
```

* El cliente envía un mensaje y el servidor responde.
* log.in: Reporta todos los inputs que un puerto está recibiendo, junto con sus respuestas.
* Se hace una conexión desde servidor a un logging port y setear log.in

La sintaxis sería:

```
yarp read /read
yarp connect /server /read tcp+log.in
```

* log.in permite avisar a YARP que se tiene una conexión con propósito especial, por ejemplo, el registro de tráfico.
* ¿Qué sucede si no está este comando? Si RPC está conectado con el cliente, la comunicación quedaría prohibida o, si se permite la comunicación, no se comportaría correctamente, por lo que es recomendable setearlo cuando sea debido.


## Clase RFModule

Esta clase ayuda a simplificar escribir un módulo genérico. Esto entrega soporte para:
* Realizar actividades periódicas.
* Manejar señales, atrapar "termination" y apagar sin problemas.
* Analizar mensajes desde un puerto para monitorear la actividad del módulo y parámetros set/get.
* Usar la clase ResourceFinder para manejar parámetros.


### Clase ResourceFinder (basic)

El tutorial muestra cómo usar ```yarp::os::ResourceFinder``` para organizar el paso de parámetros.


Suponiendo que se desea escribir un módulo llamado ```rf_basic``` que realice movimientos aleatorios de una misma articulación (joint). El módulo se construye de forma genérica para permitir el control de cualquier articulación de una extremidad del robot, simulada o real.


El módulo recibe los siguientes parámetros:
* --robot  ```<name>``` (nombre del robot)
* --part ```<robotpart>``` (extremidad del robot)
* --joint ```<jointnumber>``` (articulación a controlar)


En el ```main``` añadimos:

```cpp
ResourceFInder rf;
rf.configure(argc, argv);
```


Creando una instancia de ResourceFInder y configurándola usando los datos desde la terminal. Consultamos el valor de los parámetros de la siguiente forma:

```cpp
std::string robotName=rf.find("robot").asString();
std::string partName=rf.find("part").asString();
int joint=rf.find("joint").asInt32();
 
std::cout << "Running with:\n";
std::cout << " robot: " << robotName.c_str() << '\n';
std::cout << " part: " << partName.c_str() << '\n';
std::cout << " joint: " << joint << '\n';
```


Esto permite setear lo necesario. El programa ```rf_basic``` analiza los parámetros desde la terminal. 

Se asume que se está en el directorio fuente de ```rf_basic``` y que se tiene creado y seteado el archivo ```CMakeLists.txt```.


Entonces, se compila de la siguiente manera:

```
mkdir build
cd build
cmake ../
make
```

Finalmente, se corre el ejecutable creado:

```
./rf_basic --robot robby --part head --joint 3
```


Ahora, suponga que queremos tener los parámetros de configuración en un archivo llamado ```config.ini```. Podemos poner este archivo junto con la fuente en una carpeta específica llamada ```randomMotion```.

Se crea el archivo ```./randomMotion/config.ini``` y se abre para ingresar lo siguiente:

```
robot icub
part head
joint 0
```

La clase ```ResourceFInder``` automáticamente busca el parámetro ```--from``` que especifica desde qué archivo se leerán los parámetros de configuración.

```
./rf_basic --from ../randomMotion/config.ini
```

:warning: **NOTA:** Esto se desarrolla asumiendo que se está dentro de la carpeta build.


Existen casos en que no se sabe exactamente dónde está el archivo ini. Esto ocurre, por ejemplo, cuando se escribe un archivo que ejecuta el módulo y no se sabe la configuración exacta de la máquina. Este caso ocurre, por ejemplo, cuando se escribe scripts para yarpmanager.

:mag: **yarpmanager:** Interfaz gráfica para correr y manejar múltiples programas en un set de máquinas. Para los tutoriales y trabajos con código incompleto se utilizará más detalladamente.

Default configuration: Para que el usuario no tenga que especificar en la terminal.

```cpp
ResourceFinder rf;
rf.setDefaultConfigFile("config.ini"); //specifies a default configuration file
rf.configure(argc, argv);
```

Con esto, podemos poner ```config.ini``` en uno de los directorios definidos por YARP. Una opción posible es el directorio ```.local/share/yarp/contexts``` dentro del home del usuario. Por defecto, este es el primer lugar donde ```ResourceFInder``` busca el archivo si no se especifica con un path específico.


Para evitar confusiones y choques con otros archivos similares, mantenemos este archivo de seteo dentro del directorio ```randomMotion```. Llamaremos a este directorio un contexto para el módulo ```rf_basic```.

```
cp -r ../randomMotion/ $HOME/.local/share/yarp/contexts
```

Desde cualquier directorio de trabajo se puede correr el módulo y podrá leer los parámetros del archivo ```config.ini``` en el lugar donde fue guardado.

Para esto, se corre el módulo con el parámetro ```--context```:

```
./rf_basic --context randomMotion
```

Utilidades:
* De esta manera, el módulo puede contener más de un archivo de seteo, en donde solo se debe cambiar el directorio de donde obtiene el archivo con un solo comando.
* YARP define un grupo de lugares donde puede contener **context directories**, en este ejemplo, la carpeta ```randomMotion``` se puede instalar con el ejecutable y ubicada en tiempo de ejecución, sin que el usuario entregue la dirección exacta del archivo.



Teniendo el ejemplo anterior, podríamos querer tener un nuevo directorio con una configuración distinta. En este caso se tendrá un directorio llamado ```randomMotionSim```, que creamos de la siguiente forma:j

```
mkdir $HOME/.local/share/yarp/contexts/randomMotionSim
```



Creamos un archivo ```config.ini``` en el directorio creado con la siguiente configuración:

```
robot robbySim
part right_arm
joint 2
```

Luego, para usar esta configuración corremos lo siguiente:

```
./rf_basic --context randomMotionSim
```


Además, dentro del código es posible setear un contexto por defecto con la siguiente línea de comando:

```cpp
rf.setDefaultContext("randomMotion");
```


Entonces, cuando se corra el código y no exista un contexto especificado, el código elegirá el archivo de configuración por defecto.



:mag: Más información en [ResourceFinder](https://www.yarp.it/latest//yarp_resource_finder_tutorials.html).


## El cásico "hello world"

Este tutorial busca como objetivo familiarizarse con el uso de CMake y linkeo de librerías de YARP. 


Primero, se prepara el archivo ```main.cpp```.

```cpp
#include <cstdio>
#include <yarp/os/SystemClock.h>
 
using namespace std;
using namespace yarp::os;
 
int main()
{
    printf("Starting the application\n");
    int times=10;
 
    while(times--)
    {
        printf("Hello iCub\n");
        SystemClock::delaySystem(0.5); //wait 0.5 seconds
    }
    printf("Goodbye!\n");
}
```


Este programa hará un print con "Hello iCub" con un periodo, esperando 0.5 segundos en cada iteración del loop.

Para compilar el ejercicio, se crea el archivo```CMakeLists.txt``` de la siguiente forma:

```
project(HELLO_ICUB)
 
find_package(YARP REQUIRED)
 
add_executable(hello main.cpp)
 
# we now add the YARP libraries to our project.
target_link_libraries(hello ${YARP_LIBRARIES})
```


En la terminal se debe ejecutar primero

```
cmake ./
```

Esto permite chequear si se ha seteado correctamente el entorno de YARP y se han linkeado las librerías correspondientes.

Luego, se compila y se ejecuta lo siguiente en la terminal.

```
make
./hello
```
