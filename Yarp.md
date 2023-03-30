Instalar yarp
=================
YARP es un paquete de software de código abierto escrito en C++ que se utiliza para interconectar sensores, procesadores y actuadores en el iCub. Para su instalación se utilizará robotology superbuild.
## Robotology superbuild
Este es un repositorio que utiliza [CMake](https://cmake.org/) e [YCM](https://github.com/robotology/ycm) para descargar y compilar automáticamente software desarrollado en la organización de robotology GitHub, como el software intermedio YARP o el software utilizado para ejecutar el robot iCub.

CMake es una familia de herramientas multiplataforma de código abierto diseñada para crear, probar y empaquetar software. Un [YCM Superbuild](http://robotology.github.io/ycm/gh-pages/git-master/index.html#superbuild) es un proyecto de CMake cuyo único objetivo es descargar y compilar varios otros proyectos. Se puede leer más sobre el concepto de superbuild en la [documentación YCM ](http://robotology.github.io/ycm/gh-pages/latest/index.html) o en el [documento de IRC relacionado](http://lornat75.github.io/papers/2018/domenichelli-irc.pdf).

| Sistema  | Continuous Integration Status |
|:------:|:------:|
|  Linux/macOS/Windows  |  ![GitHub Actions Status](https://github.com/robotology/robotology-superbuild/workflows/C++%20CI%20Workflow/badge.svg)     |

1. Instalación git  
Para esto se espera primero la creación y verificación de una cuenta en GitHub ya que es una herramienta útil, tanto para trabajar con los repositorios de robotology como para otras fuentes útiles.
Luego de esto, se instala git a través de la terminal con el siguiente comando:

```
sudo apt install git
```

Junto con esto, se debe ejecutar en terminal lo siguiente:


```
sudo apt-get install build-essential cmake cmake-curses-gui coinor-libipopt-dev
freeglut3-dev git libace-dev libboost-filesystem-dev libboost-system-dev libboost-
thread-dev libdc1394-22-dev libedit-dev libeigen3-dev libgsl0-dev libjpeg-dev
liblua5.1-dev libode-dev libopencv-dev libsdl1.2-dev libtinyxml-dev libv4l-dev
libxml2-dev lua5.1 portaudio19-dev qml-module-qt-labs-folderlistmodel qml-module-qt-
labs-settings qml-module-qtmultimedia qml-module-qtquick-controls qml-module-qtquick-
dialogs qml-module-qtquick-window2 qml-module-qtquick2 qtbase5-dev qtdeclarative5-dev
qtmultimedia5-dev swig libmatio-dev
```

:warning: **NOTA**: Si esto no se ejecuta correctamente, se propone una [solución alternativa.](#item1)

2. Verificación cuenta git

En este paso se debe ejecutar los siguientes comandos en la terminal:

```
git config --global user.name NombreUsuario (Reemplazar NombreUsuario por usuario creado en GitHub)
git condig --global user.email usuario@email.com (Reemplazar usuario@email.com por correo GitHub)
```

Para verificar el usuario y correo se ejecuta el siguiente comando:
```
sudo gedit ~/.gitconfig
```

3. Instalación Robotology

Para instalar las herramientas de YARP  y ICUB, lo primero que se debe hacer es clonar el repositorio de robotology

```
git clone https://github.com/robotology/robotology-superbuild
```
<a name="item1"></a> 
- **Solución alternativa** al punto 1: Luego de clonar el repositorio, se debe ejecutar lo siguiente
	
```
	cd robotology-superbuild
	sudo bash ./scripts/install_apt_dependencies.sh
```
Luego, se debe ir a la carpeta robotology-superbuild y ejecutar el comando:

```
mkdir build
cd build
ccmake ../
```

El comando "ccmake ../" abrirá un apartado en el terminal que se debe setear, por lo que si se abre una pestaña vacía, primero se debe configurar con la tecla c. La configuración seteará por defecto algunos apartados en :heavy_check_mark:**ON** y otros en :x:**OFF**. 


Se recomienda dejar esta configuración y dejar en :heavy_check_mark:**ON** lo relacionado a YARP, ICUB o GAZEBO.


4. Configuración de entorno

Para setear la configuración hecha en el paso anterior se debe usar el siguiente comando:

```
source <directory-where-you-downloaded-robotology-superbuild>/build/install/share/robotology-superbuild/setup.sh
```

Y luego se debe actualizar `/setup.sh` con el archivo `.bashrc` mediante el comando:

```
source ~/.bashrc
```

5. Confirmación de instalación

Para asegurarnos de que la instalación se ha efectuado correctamente se propone probar 3 comandos importantes:

- Se revisa el servidor:
	
```
yarpserver
```
	
- Se revisa el entorno de simulación del iCub:

```
iCub_SIM
```

- Se revisa la conexión con los puertos a las articulaciones del robot

```
yarpmotorgui --robot icubSim	
```

:mag: **Para más referencias sobre instalación:**

* [Tutorial instalación 1](https://www.youtube.com/watch?v=4g5i_q1etck&t=20s)
* [Tutorial instalación 2](https://www.youtube.com/watch?v=Q0ppqigyCPI&t=2s)
* [Repositorio Robotology](https://github.com/robotology/robotology-superbuild)
