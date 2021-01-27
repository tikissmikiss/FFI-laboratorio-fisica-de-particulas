# <center><span style="color:blue">Laboratorio de física de partículas</span></center>

## <center>Geant4 powered with Docker</center>

![](https://pbs.twimg.com/media/CWhF64OXIAAtCa4.png#center)

## Objetivo

Simular la interacción básica de partículas con ciertos medios gracias al software Geant4, desarrollado por el CERN.

## Play with Docker

Play with Docker es un servicio cloud que nos permite jugar con Docker sin necesidad de instalar Docker localmente. Cada vez que accedemos a una sesión en [Play with Docker](https://labs.play-with-docker.com/), tenemos 4 maravillosas horas para experimentar con Docker, hasta que esta se cierra automáticamente. Parece poco tiempo pero es el suficiente para llevar a cabo nuestro humilde y divertido laboratorio. Dicho de otra manera: tenemos acceso casi ilimitado (en recursos) a un super computador orientado a familiarizarnos con contenedores virtuales durante todo el periodo de duración de la clase/laboratorio. Para acceder a Play with Docker, sólo necesitamos una [cuenta en Docker](https://hub.docker.com/register/). Como en la gran mayoría de servicios, este registro es gratuito, rápido y comporta los típicos pasos de selección de contraseña, envío de correo, confirmación de recepción de correo, etc.

Una vez que ya tenemos acceso a la consola de Play with Docker, podemos solicitar máquinas reales (en realidad siguen siendo virtuales, pero a efectos prácticos, las trataremos como si fueran reales) sobre las que ejecutar a su vez contenedores Docker gracias a su [toolkit](https://docs.docker.com/engine/reference/commandline/docker/#child-commands). Al inicio no habrá ninguna, pero podemos crear la primera (y única que necesitamos) con el comando ADD NEW INSTANCE que se puede ver en la imagen.

![](https://i.stack.imgur.com/y9qCI.png)

Cuando creamos una instancia, es como si accediéramos a una máquina pseudo real (o para lo que a todos los efectos, será una máquina real para nosotros). Se nos abrirá una sesión de shell de GNU Linux (Bash en este caso) justo al lado y sobre la que podemos introducir comandos Unix de toda la vida, entre ellos los relacionados con Docker. En estas máquinas cuasi-reales ya está instalado todo el stack de Docker y podemos, a su vez, trabajar con contenedores virtuales basados en esta tecnología de vanguardia. En realidad Play with Docker hace uso de [DIND](https://github.com/jpetazzo/dind). Esta tecnología permite albergar contenedores Docker dentro de un contenedor Docker (sí, algo así como un sueño dentro de un sueño). Es decir: las instancias en Play with Docker son en realidad… ¡contenedores Docker!

### Pregunta: ¿cuál es la diferencia entre una imagen y un contenedor?

> Una máquina virtual es una visualización de un equipo físico completo. Se emula por software todos sus componentes, tanto físicos como lógico. Esto incluye los discos duros, tarjeta de red, gráfica, sonido, etc. Además del SO con su kernel y todos sus servicios.

Fíjate que la instancia que has creado en Play with Docker es como si se tratara de un ordenador tuyo que estuviera en red y en ese ordenador tuvieras instalado Docker. Así de sencillo. A partir de ese momento ya no necesitamos acceder a Play with Docker mediante en el navegador (aunque está bien que lo dejes aparcado a un lado en caso de necesidad). Usaremos VS Code y acceso SSH tradicional, concretamente mediante dos clientes SSH milenarios: [OpenSSH](https://www.openssh.com/) y [Putty](https://www.putty.org/). El primero viene preinstalado típicamente en máquinas GNU/Linux y macOS y el segundo se tiene que descargar y hay versiones para todos los sistemas operativos (es el que se recomienda, por simplicidad, si estás en Windows). Este enunciado viene preparado para uses cualquiera de los dos, en cualquier sistema.

Importante: tómate nota del usuario que te da Play with Docker y que empieza por `ip` y acaba en `@direct…` (Ejemplo: `ip172-18-0-27-bg3pvjs3uhdg008ir0fg`). Copia esa cadena de texto y guárdatela para futuros usos. A lo largo de este trabajo nos referiremos a ella con la variable de entorno `$usuariopwd`. También puedes regenerar el contenido de esta variable mediante este comando (a ejecutar en la consola web de Play with Docker):

```bash
echo ip$(ifconfig eth1 | grep "inet " | awk -F'[: ]+' '{ print $4 }' | sed 's/\./-/g')-$SESSION_ID
```

### Pregunta: ¿qué hace el comando anterior? ¿De qué subcomandos se compone y qué papel juegan (ifconfig, grep, awk y sed)?

> Ejecuta el comando `echo` que imprime por la salida estándar el texto que le siga, expandiendo el contenido de las variables. En nuestro caso tenemos dos variables, una de ellas implícita. El formato que tenemos es este `echo ip$var1-$var2`. Donde `$var1` representa el resultado de la expresión contenida entre paréntesis y `$var2` representa el contenido de la variable `$SESSION_ID`, que no es otro que el identificador de sesión que nos ha asignado `Play whit Docker`.
>
> Para lo que sería `$var1`, dentro del paréntesis se ejecuta el comando `ifconfig eth1` y su salida la pasa al comando `grep "inet "` usando una tubería (pipe), este pasa su salida a `awk -F'[: ]+' '{ print $4 }'` y este a `sed 's/\./-/g'`.
>
> - **ifconfig:** El comando `ifconfig` muestra las propiedades de las interfaces de red instaladas en el equipo. El argumento `eth1` filtra la salida para que solo devuelva las propiedades de la interfaz de red con ese nombre.
>
>   Ejemplo de salida del comando `ifconfig eth0`
>
>   ```bash
>   jose@Aurora-R6:/mnt/c/Users/joseh$ ifconfig eth0
>   eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
>           inet 172.23.122.172  netmask 255.255.240.0  broadcast 172.23.127.255
>           inet6 fe80::215:5dff:fe1b:172d  prefixlen 64  scopeid 0x20<link>
>           ether 00:15:5d:1b:17:2d  txqueuelen 1000  (Ethernet)
>           RX packets 23126  bytes 6106428 (5.8 MiB)
>           RX errors 0  dropped 0  overruns 0  frame 0
>           TX packets 19  bytes 1426 (1.3 KiB)
>           TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
>   ```
>
>   Nótese que he cambiado el argumento `eth1` por `eth0`, puesto que mi maquina no dispone de una interfaz de red con el nombre `eth1`.
>
> * **grep:** El comando `grep` busca un fragmento de texto y filtra cualquier contenido entre retornos de carro, y/o cambios de línea, que no contengan el fragmento de texto pasado como argumento. En nuestro caso todas las líneas que devuelva el comando `ifconfig eth0` que no contengan la cadena de caracteres `"inet "` incluyendo el espacio. Para nuestro ejemplo el comando `ifconfig eth0 | grep "inet "` devolvería lo siguiente:
>
>   ```bash
>   jose@Aurora-R6:/mnt/c/Users/joseh$ ifconfig eth0 | grep "inet "
>           inet 172.23.122.172  netmask 255.255.240.0  broadcast 172.23.127.255
>   ```
>
>   Como se ve, se han eliminado todas las líneas que no contenían el texto `"inet "`
>
> * **awk:** El comando `awk` al igual que `grep` busca un fragmento de texto, pero este puede filtrar el contenido de cada línea. En nuestro caso se establece el modo de funcionar usando varios argumentos. En `awk -F'[: ]+' '{ print $4 }'`, `-F'[: ]+'` establece la secuencia de caracteres que se usará como separador de campos, `-F` indica que se va a establecer un separador de campos especifico. Lo que le sigue es una expresión regular, o _regex_, con la que desea que coincida el texto que hará de separador de campos. Concretamente `-F'[: ]+'` establece como separador de campos cualquier secuencia que este compuesta de uno o más caracteres de los contenidos entre los corchetes, estos son los dos puntos ':' y el espacio. Todos estos ejemplos coincidirían con ese regex `' :: '`, `' '`, `'::::::::'`. La parte `'{ print $4 }'` expresa como formatear la salida, en este ejemplo se establece que se imprima el cuarto campo, es decir el contenido entre la tercera y la cuarta coincidencia del regex.
>
>   En mi ejemplo nos encontramos un problema, este es que el comando anterior devuelve la línea con espacios antes de la cadena `"inet "`, por lo que el primer campo corresponde al contenido antes de la primera coincidencia del regex, que evidentemente es una cadena vacía `''`. Esto provoca que no se devuelva lo que se espera, si no que devuelve `netmask`.
>
>   ```bash
>   jose@Aurora-R6:/mnt/c/Users/joseh$ ifconfig eth0 | grep "inet " | awk -F'[: ]+' '{ print $4 }'
>   netmask
>   ```
>
>   Para conseguir que se devuelva mi mascara de red se debería de usar `awk -F'[: ]+' '{ print $5 }'`:
>
>   ```bash
>   jose@Aurora-R6:/mnt/c/Users/joseh$ ifconfig eth0 | grep "inet " | awk -F'[: ]+' '{ print $3 }'
>   255.255.240.0
>   ```
>
>   Pero intuyo que lo que se desea es mi dirección IP, para lo cual necesitaremos usar `awk -F'[: ]+' '{ print $3 }'`
>
>   ```bash
>   jose@Aurora-R6:/mnt/c/Users/joseh$ ifconfig eth0 | grep "inet " | awk -F'[: ]+' '{ print $5 }'
>   172.23.122.172
>   ```
>
> * **sed:** El comando `sed` permite modificar el texto que recibe. En tal como está configurado con `'s/\./-/g'`, se le está pidiendo que sustituya cada punto `.` que encuentre por un guion `-`.
>
>   En el ejemplo resultaría:
>
>   ```bash
>   jose@Aurora-R6:/mnt/c/Users/joseh$ ifconfig eth0 | grep "inet " | awk -F'[: ]+' '{ print $3 }' | sed 's/\./-/g'
>   172-23-122-172
>   ```
>
>   Con esto tenemos el contenido del paréntesis, a lo que le precede los caracteres 'ip' y le sigue un guion seguido del contenido de la variable `$SESSION_ID` que para el enunciado contiene `bg3pvjs3uhdg008ir0fg`.
>   Así que el comando `echo` mostrará primero la parte explicita `ip`, seguido de lo del paréntesis `172-23-122-172`, seguido de un guion `-` explícito y finalmente la variable $SESSION_ID `bg3pvjs3uhdg008ir0fg`. Si lo juntamos el resultado es:
>
>   ```bash
>   ip172-23-122-172-bg3pvjs3uhdg008ir0fg
>   ```
>
>   Como vemos coincide con lo mencionado `ip172-18-0-27-bg3pvjs3uhdg008ir0fg`, con la salvedad de que la ip que aparece es la de mi máquina.

## Software a instalar localmente

Estos son los distintos programas/paquetes que necesitas en tu sistema local:

- [Visual Studio Code](https://code.visualstudio.com/) (VS Code, para los amigos).
- [Python](https://marketplace.visualstudio.com/items?itemName=ms-python.python). ¡Ojo!, no es necesario instalar Python ni vamos a trabajar con un intérprete de este lenguaje en nuestra máquina local. Sólo queremos esta extensión para que VS Code haga sintaxis de color de código Python (que será ejecutado por un contenedor Docker que crearemos en nuestra instancia pseudoreal en el servicio Play with Docker).
- [Preview on Web Server](https://github.com/YuichiNukiyama/vscode-preview-server), de Yuichi Nuikiyama. Esta extensión nos permite visualizar las simulaciones 3D en un navegador. Con la combinación de teclas Cntrl + Shift + L se abrirá un navegador que estará atento a cambios en la página web donde se dibujen las escenas 3D representando la física que estamos simulando.
- [Putty](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html), un cliente SSH muy conocido en Windows. También puedes usar Putty en macOS. Te recomiendo que los instales mediante [Scoop](https://scoop.sh/) o [Homebrew](https://brew.sh/), respectivamente.
- [Instant Player](http://www.instantreality.org/downloads) para visualizar las trazas de partículas. Se trata un visor de instrucciones [VWRL](https://en.wikipedia.org/wiki/VRML) disponible tanto para [macOS](http://doc.instantreality.org/media/uploads/downloads/2.8.0/InstantPlayer-MacOS-10.10-x64-2.8.0.38619.dmg), [Windows](http://doc.instantreality.org/media/uploads/downloads/2.8.0/InstantReality-Windows-vs2013-x64-2.8.0.38619.msi), [Ubuntu](http://doc.instantreality.org/media/uploads/downloads/2.8.0/InstantReality-Ubuntu-14.04-x64-2.8.0.38619.deb), [Red Hat](http://doc.instantreality.org/media/uploads/downloads/2.8.0/InstantReality-RedHat-7-x64-2.8.0.38619.rpm) y [SUSE](http://doc.instantreality.org/media/uploads/downloads/2.8.0/InstantReality-SLES-12.1-x64-2.8.0.38937.rpm). Para solucionar el problema de que falta un archivo `MSVCP110.dll` en el caso de algunas versiones de Windows, instalad el runtime mínimo de Microsoft Visual C++ 2012 Redistributable para 64 bit desde [aquí](http://www.microsoft.com/en-au/download/confirmation.aspx?id=30679).

  ### Preguntas:

  **¿Qué otros formatos estándar de representación 3D conoces (Collada, Stanford, Wavefront, X3D Extensible, Standard Tessellation Language, x3dom, etc.)?**

  > Sinceramente, hasta antes de esto ninguno, salvo CAD, que en realidad no es un formato estándar, ya que CAD admite diferentes formatos, tanto para 2D como para 3D. Después de esto conozco los aquí mencionados. Collada, Stanford, Wavefront, X3D Extensible, Standard Tessellation Language, x3dom

  **¿Quién desarrolla Instant Player qué contribuciones ha hecho al mundo de la tecnología?**

  > Instant Player pertenece a 'The instantReality framework'.
  > El proyecto está actualmente coordinado y respaldado por el grupo Fraunhofer IGD / [Visual Computing System Technologies](http://www.igd.fraunhofer.de/vcst/).
  >
  > [El Instituto Fraunhofer de Gráficos por Computadora (IGD-VCST)](http://www.igd.fraunhofer.de/vcst/) cubre un amplio espectro de experiencias y competencias en los campos de las tecnologías de Realidad Virtual / Aumentada y Computación Visual.

- [ParaView](http://www.paraview.org/), de [Kitware](https://www.kitware.com/), pero sólo se recomienda su uso en el caso de que, por alguna razón Instant Player no funcione de ninguna manera (quizás en el caso de que tu arquitectura sea todavía 32 bit). Para ver las trazas de las partículas correctamente, elige el modo Wireframe (barra de herramientas).

  ### Pregunta:

  **¿en qué proyectos está involucrada la fundación Kitware y cómo han cambiado el mundo de la informática, la tecnología, la ciencia y la medicina? Pista: CMake, VTK, ITK, etc.**

  > - **The Visualization Toolkit (VTK)** es un sistema de software para gráficos 3D por ordenador, procesamiento de imágenes y visualización. Admite muchos algoritmos de visualización y técnicas de modelado. VTK puede realizar un procesamiento paralelo y puede representar datos científicos en un navegador web . VTK se utiliza en todo el mundo en aplicaciones comerciales, así como en investigación y desarrollo. Las aplicaciones que usan VTK incluyen Molekel, ParaView, VisIt, VisTrails, MOOSE, 3D Slicer, MayaVi y OsiriX. VTK está escrito en C ++ y está empaquetado para acceder a través de Python, Java y Tcl. ActiViz proporciona soporte para VTK en proyectos .Net / C #.
  >
  > - **ParaView** crea visualizaciones científicas interactivas para analizar datos utilizando técnicas cualitativas y cuantitativas. Tiene una arquitectura cliente-servidor para facilitar la visualización remota de conjuntos de datos y genera modelos de nivel de detalle (LOD) para mantener velocidades de cuadro interactivas para grandes conjuntos de datos. Está diseñado para el paralelismo de datos en multicomputadoras y clústeres de memoria compartida o memoria distribuida. También se puede ejecutar como una aplicación para una sola computadora.
  >
  > - **CMake** es una herramienta de compilación multiplataforma para controlar el proceso de compilación de software utilizando archivos de configuración simples independientes de la plataforma y del compilador. CMake genera archivos MAKE y espacios de trabajo nativos que se pueden usar en el compilador de su elección.
  >
  > - **The Insight Toolkit (ITK)** Es una biblioteca de algoritmos de segmentación y registro de imágenes adaptados para investigaciones médicas.
  >
  >   El kit de herramientas es compatible con una variedad de formatos de datos de imágenes, que incluyen imágenes digitales y comunicaciones en medicina (DICOM), MRI, CT y ultrasonido. Además, presenta un proceso de empaquetado automatizado para generar interfaces entre C++ y lenguajes de programación interpretados como Python o JavaScript, para permitir a los desarrolladores crear software usando una gran variedad de lenguajes de programación. Tiene una estructura modular flexible que es fácil de ampliar e integrar en varios proyectos.
  >
  > - **3D Slicer** es una aplicación de código abierto extensible para visualización y análisis de imágenes médicas. 3D Slicer funciona con imágenes ópticas, resonancia magnética, tomografía computarizada y datos de ultrasonido. Se ha utilizado para aplicaciones comerciales y de investigación que van desde estudios preclínicos en animales, hasta planificación y orientación quirúrgica, análisis de imágenes de ultrasonido, control de robots médicos y estudios de población.
  >
  >   3D Slicer utiliza VTK e ITK: VTK para sus procesos de renderizado 2D y 3D, transformación lineal y no lineal, infraestructura de segmentación, procesamiento de malla e integración de realidad virtual e ITK para el procesamiento de imágenes más lectura y escritura de imágenes.
  >
  > - **VIAME** es una plataforma de software de visión por computadora de código abierto diseñada para la inteligencia artificial (IA) _do-it-yourself_ (hazlo tu mismo). Es un conjunto de herramientas en evolución que contiene muchos flujos de trabajo que se utilizan para generar diferentes detectores de objetos, clasificadores de fotograma completo, mosaicos de imágenes, generación rápida de modelos, búsqueda de imágenes y videos y métodos de medición estéreo.
  >
  > Originalmente dirigido al análisis de especies marinas, ahora contiene muchos algoritmos y bibliotecas comunes, y también es útil como una biblioteca genérica de visión por computadora fuera de las imágenes y videos submarinos. VIAME está disponible como aplicación web o de escritorio.
  >
  > - **TeleSculptor** es una aplicación de escritorio de usuario final de código abierto para fotogrametría aérea. Aprovecha los algoritmos de KWIVER para construir modelos de escenas en 3D a partir de videos aéreos, como videos recopilados de UAV. Maneja coordenadas geoespaciales y puede hacer uso de metadatos, si están disponibles, de sensores GPS e IMU. Sin embargo, también puede funcionar con datos no geoespaciales y con colecciones de imágenes sin metadatos asociados. Aunque es principalmente una aplicación de usuario final, es altamente reconfigurable para soportar también la investigación y el desarrollo de fotogrametría.
  >
  > - **Girder** es una plataforma de gestión de datos basada en web gratuita y de código abierto que forma parte del ecosistema de análisis y datos de Resonant. Puede usarse como una aplicación independiente o una plataforma para crear nuevos servicios web. Permite la construcción rápida y sencilla de aplicaciones web que tienen algunos o todos los siguientes requisitos: organización y difusión de datos, gestión y autenticación de usuarios, gestión de autorizaciones.

- [view3dscene](https://castle-engine.io/view3dscene.php), de [Castle Game Engine](https://castle-engine.io/index.php). Es otro visualizador 3D multiplataforma, parte de este conocidísimo motor de videojuegos (competencia directa de [Unity](https://unity.com/)).
- [Docker](https://www.docker.com/) ¡No! No es necesario instalar Docker, pero ya que estamos…

  ### Preguntas:

  **¿qué es Docker?**

  **¿Qué son los contenedores virtuales?**

  > La idea detrás de Docker es crear contenedores ligeros y portables para las aplicaciones software que puedan ejecutarse en cualquier máquina con Docker instalado, independientemente del sistema operativo que la máquina tenga por debajo, facilitando así también los despliegues. Es un entorno de virtualización ligero que te permite construir y ejecutar aplicaciones dentro de un contenedor de software aislado. No se apoya en un hypervisor como la virtualización tradicional, se puede considerar una aplicación dentro del servidor.

  **¿Qué alternativas a Docker existen (OpenVZ, LXC, Vagrant, rkt, Singularity, etc.)?**

  > Algunas de las alternativas que podemos encontrar son:
  >
  > - RKT.
  > - PodMan.
  > - Singularity.
  > - Linux Containers – LXC.
  > - OpenVz.

  **Resume brevemente la historia de Docker.**

  > [Docker](https://www.docker.com/) nace como un sistema enfocado a **Plataforma como Servicio**, o, **PaaS** de sus siglas en ingles _Platform as a Service_). **Solomon Hykes**, junto con otros ingenieros como **Andrea Luzzardi**, **Francois-Xavier Bourlet** y **Jeff Lindsay**, comenzó Docker como un proyecto interno dentro de la empresa **dotCloud**.
  >
  > Docker fue liberado en marzo de 2013 convirtiéndose en un software de código abierto.
  >
  > En el 2014 remplazó su entorno de ejecución por defecto por su propia biblioteca, libcontainer, escrita en **Go**.
  >
  > En abril de 2015, el proyecto tenía más de **20.700 estrellas de GitHub** (Posicionándolo en el **puesto 20ª** del rancking de proyectos con más estrellas de GitHub), más de 4.700 bifurcaciones (forks), y casi 900 colaboradores.
  >
  > En 2018 un estudio mostró que organizaciones como **Red Hat**, **Microsoft**, **IBM**, **Google**, **Cisco Systems** y **Amadeus IT Group**, aportaban mayores contribuciones al proyecto que el propio equipo de Docker, siendo **Red Hat** la organización más volcada con el proyecto.

- [PowerShell](https://docs.microsoft.com/en-us/powershell/). En el caso de Windows (en incluso [Linux](https://docs.microsoft.com/en-us/powershell/scripting/install/installing-powershell-core-on-linux?view=powershell-6) y [macOS](https://docs.microsoft.com/en-us/powershell/scripting/install/installing-powershell-core-on-macos?view=powershell-6)), te recomiendo que uses PowerShell. Se trata de un shell de código abierto y multiplataforma creado por Microsoft. Todos los comandos de este manual vienen preparados para Bash y PowerShell. VS Code debería detectar que tienes PowerShell instalado y ofrecerte su uso como shell por defecto, pero si acaso no fuera así, consulta esta [página de ayuda](https://techcommunity.microsoft.com/t5/itops-talk-blog/configure-visual-studio-code-to-run-powershell-for-windows-and/ba-p/283258) de Microsoft.

## Ejecución del laboratorio

La realización de este actividad se ha realizado ejecutando la simulación en un contenedor Docker en local, cortesía del Dr. Alberto Corbi.

- Lo primero es tener Docker instalado.
- En mi caso no necesitaremos necesitariamos los archivos `.key` y `.ppk` que contienen las claves que se utilizan en una comunicacion encriptada con cifrado de clave publica. Por lo que no necesitamos crear una instancia en Play with Docker.
- Abrimos una carpeta de trabajo (workspace) desde Visual Studio Code. En mi caso en un Subsistema de Windows para Linux ([WLS2](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=&cad=rja&uact=8&ved=2ahUKEwiUk_6L1LruAhXE0eAKHZ5dD1wQFjAAegQIBRAC&url=https%3A%2F%2Fdocs.microsoft.com%2Fes-es%2Fwindows%2Fwsl%2Finstall-win10&usg=AOvVaw1ZXumGCZrY2vdbQ6QgReYX)).
- Tampoco necesitamos sdscárgar los archivos `geant4lab.ppk` y `geant4lab.key` desde [aquí](https://github.com/pammacdotnet/FFRepo/raw/master/ppkkey.zip).

  ### Preguntas:

  **¿Qué es un fichero ppk?**

  > Como he comentado mas arriba estos archivos contien las claves necesarias para encriptar y desemcriptar la informacion que se envia y se recive. Concretamente ppk contieen la clave privada y su nombre viene de **PuTTY Private Key**.

  **¿Qué es una clave RSA público/privada (fichero key)?**

  > Es el equivalente al ppk pero para usarlo en SSH en vez de PuTTY

## Descarga de la imagen Docker del laboratorio

Lo primero que tenemos que hacer es descargarnos la imagen de la pila de contenedores que conforman una imagen con todo lo necesario para ejecutar Geant4. Esta imagen [se encuentra ya en el Docker Hub](https://cloud.docker.com/u/pammacdotnet/repository/docker/pammacdotnet/geant4lab).

### Preguntas:

**¿qué es el [Hub](https://hub.docker.com/) de Docker?**

> Docker Hub es un servicio proporcionado por Docker para encontrar y compartir imágenes de contenedores en nuestros ordenadores.

**¿Cuál es la diferencia entre una imagen y un contenedor?**

> Una imagen es una plantilla para construir un contenedor. Una imagen nunca cambia.
> Un contenedor es una instancia de una imagen. Se pueden costruir los contenedores que se quiera partiendo de la misma imagen.

Para descargar la imagen Docker con la que ejecutaremos la simulación, enviaremos el comando (`docker pull`) desde nuestra máquina local y esperamos a que se descargue.

Este comando Docker (`docker pull`) lo que hace es descargarse la imagen `geant4lab` del usuario `pammacdotnet` alojada en el [Hub de Docker](http://hub.docker.com/). La descarga se produce desde este _Hub_ al disco de nuestra máquina local.

```bash
jose@Aurora-R6:~/FFI$ docker pull pammacdotnet/geant4lab
Using default tag: latest
latest: Pulling from pammacdotnet/geant4lab
32802c0cfa4d: Pulling fs layer
da1315cffa03: Pulling fs layer
fa83472a3562: Pulling fs layer
f85999a86bef: Pulling fs layer
4adeade91205: Pulling fs layer
a8e4b753ce11: Pulling fs layer
e9f91fadad2e: Pulling fs layer
05bd8563d519: Pulling fs layer
17dea6add1aa: Pulling fs layer
ffc09338d88e: Pulling fs layer
c7061889ec78: Pulling fs layer
82cf27079d06: Pull complete
3fd0b7114b4d: Pull complete
762fc1ce02fe: Pull complete
e29f76d111d5: Pull complete
10d5c3407cc3: Pull complete
cacafc401cf0: Pull complete
2e5ea4f36bbd: Pull complete
9a46be81e6b4: Pull complete
dd1b29d9ca12: Pull complete
b3d7bb7e2722: Pull complete
485e80a4790d: Pull complete
Digest: sha256:7b342489eda508f6bf3d9723437c1107219c2f46ab24d0072f6341d00d486446
Status: Downloaded newer image for pammacdotnet/geant4lab:latest
docker.io/pammacdotnet/geant4lab:latest
```

Desde la consola de `Bash` integrada en en **VS Code** descargamos el fichero `simulation.py`.

```bash
curl https://raw.githubusercontent.com/pammacdotnet/FFRepo/master/simulation.py -o simulation.py -s
```

De igual manera, descargamos el fichero `wrl2html.py`, que servirá más tarde para generar versiones WebGL de las simulaciones:

```bash
curl https://raw.githubusercontent.com/pammacdotnet/FFRepo/master/wrl2html.py -o wrl2html.py -s
chmod +x wrl2html.py
```

Este código de la simulación describe un experimento de física de partículas cuya representación es más o menos esta:

![](https://pbs.twimg.com/media/CYbab8dWAAA4CCi.png)

Se trata de un _universo_ en forma de _cuboide_ donde las partículas subatómicas son aceleradas (por los medios que sean) y son proyectadas con forma _cónica_ contra un objetivo (un _target_ o _fantoma_) de un material que podemos variar a nuestro antojo en composición, tamaño y posición. Nosotros no nos preocupamos sobre cómo esas partículas han cobrado esa energía, sino solamente por cómo interaccionan con la materia.

### Pregunta:

**¿cómo se llama a este [tipo de haz](https://htmlpreview.github.io/?https://github.com/pammacdotnet/FFRepo/blob/master/geant4lab2021.html#search=%22pencil%20beam%22) de partículas?**

> Pencil beam

## Ejecución de las simulaciones

Ahora sólo tenemos que lanzar el comando de ejecución de la simulación (con opciones por defecto) desde el terminal integrado de VS Code:

```bash
jose@Aurora-R6:~/FFI$ docker run -v ${PWD}:/root pammacdotnet/g4lpwd
**************************************************************
 Geant4 version Name: geant4-10-04-patch-02    (25-May-2018)
                       Copyright : Geant4 Collaboration
                      References : NIM A 506 (2003), 250-303
                                 : IEEE-TNS 53 (2006), 270-278
                                 : NIM A 835 (2016), 186-225
                             WWW : http://geant4.org/
**************************************************************

Visualization Manager instantiating with verbosity "warnings (3)"...
/tracking/storeTrajectory 1
/tracking/storeTrajectory 2

phot:   for  gamma    SubType= 12  BuildTable= 0
      LambdaPrime table from 200 keV to 100 TeV in 61 bins
      ===== EM models for the G4Region  DefaultRegionForTheWorld ======
       PhotoElectric :  Emin=        0 eV    Emax=      100 TeV   AngularGenSauterGavrila

compt:   for  gamma    SubType= 13  BuildTable= 1
      Lambda table from 100 eV  to 1 MeV, 7 bins per decade, spline: 1
      LambdaPrime table from 1 MeV to 100 TeV in 56 bins
      ===== EM models for the G4Region  DefaultRegionForTheWorld ======
       Klein-Nishina :  Emin=        0 eV    Emax=      100 TeV

conv:   for  gamma    SubType= 14  BuildTable= 1
      Lambda table from 1.022 MeV to 100 TeV, 18 bins per decade, spline: 1
      ===== EM models for the G4Region  DefaultRegionForTheWorld ======
        BetheHeitler :  Emin=        0 eV    Emax=       80 GeV
     BetheHeitlerLPM :  Emin=       80 GeV   Emax=      100 TeV

msc:   for e-    SubType= 10
      RangeFactor= 0.04, stepLimitType: 1, latDisplacement: 1
      ===== EM models for the G4Region  DefaultRegionForTheWorld ======
            UrbanMsc :  Emin=        0 eV    Emax=      100 TeV  Table with 84 bins Emin=    100 eV    Emax=    100 TeV

eIoni:   for  e-    SubType= 2
      dE/dx and range tables from 100 eV  to 100 TeV in 84 bins
      Lambda tables from threshold to 100 TeV, 7 bins per decade, spline: 1
      finalRange(mm)= 1, dRoverRange= 0.2, integral: 1, fluct: 1, linLossLimit= 0.01
      ===== EM models for the G4Region  DefaultRegionForTheWorld ======
        MollerBhabha :  Emin=        0 eV    Emax=      100 TeV

eBrem:   for  e-    SubType= 3
      dE/dx and range tables from 100 eV  to 100 TeV in 84 bins
      Lambda tables from threshold to 100 TeV, 7 bins per decade, spline: 1
      LPM flag: 1 for E > 1 GeV,  VertexHighEnergyTh(GeV)= 100000
      ===== EM models for the G4Region  DefaultRegionForTheWorld ======
             eBremSB :  Emin=        0 eV    Emax=        1 GeV   DipBustGen
            eBremLPM :  Emin=        1 GeV   Emax=      100 TeV   DipBustGen

msc:   for e+    SubType= 10
      RangeFactor= 0.04, stepLimitType: 1, latDisplacement: 1
      ===== EM models for the G4Region  DefaultRegionForTheWorld ======
            UrbanMsc :  Emin=        0 eV    Emax=      100 TeV  Table with 84 bins Emin=    100 eV    Emax=    100 TeV

eIoni:   for  e+    SubType= 2
      dE/dx and range tables from 100 eV  to 100 TeV in 84 bins
      Lambda tables from threshold to 100 TeV, 7 bins per decade, spline: 1
      finalRange(mm)= 1, dRoverRange= 0.2, integral: 1, fluct: 1, linLossLimit= 0.01
      ===== EM models for the G4Region  DefaultRegionForTheWorld ======
        MollerBhabha :  Emin=        0 eV    Emax=      100 TeV

eBrem:   for  e+    SubType= 3
      dE/dx and range tables from 100 eV  to 100 TeV in 84 bins
      Lambda tables from threshold to 100 TeV, 7 bins per decade, spline: 1
      LPM flag: 1 for E > 1 GeV,  VertexHighEnergyTh(GeV)= 100000
      ===== EM models for the G4Region  DefaultRegionForTheWorld ======
             eBremSB :  Emin=        0 eV    Emax=        1 GeV   DipBustGen
            eBremLPM :  Emin=        1 GeV   Emax=      100 TeV   DipBustGen

annihil:   for  e+, integral: 1     SubType= 5  BuildTable= 0
      ===== EM models for the G4Region  DefaultRegionForTheWorld ======
            eplus2gg :  Emin=        0 eV    Emax=      100 TeV
```

Aparecerá un nuevo fichero en el directorio de trabajo: `simulation.wrl`. Este fichero es un diagrama 3D que puedes ver con un visualizador externo.

### Pregunta:

**¿Qué son los archivos `wrl`?**

> La extensión de archivo `WRL` se refiere a un archivo creado en `VRML`, que viene de Lenguaje de Modelado de Realidad Virtual. Este archivo, representa un mundo 3D virtual, es guardado con un formato de texto `ASCII` y puede ser editado por un editor de texto.

**¿De qué estándar internacional se trata?**

> De `VRML` o Lenguaje de Modelado de Realidad Virtual (Siglas del inglés Virtual Reality Modeling Language)

Este comando que acabamos de comentar (`docker run`) crea un contenedor Docker que tiene mapeada la carpeta interna `/root` con el directorio personal (`home`) del usuario root. Esto se consigue con la opción `-v` (más info [aquí](https://htmlpreview.github.io/?https://github.com/pammacdotnet/FFRepo/blob/master/geant4lab2021.html#mount-into-a-non-empty-directory-on-the-container)). Sí… suena un poco mareante y efectivamente así es.

El contenedor anterior tiene como cometido ejecutar un único proceso (que no es otro que un intérprete **Python** que interpreta las instrucciones de `simulation.py`). Como las dos carpetas están enlazadas tanto para lectura como escritura, el fichero `simulation.wrl` estará disponible en `/root`.

Para entender el resto de opciones disponibles para la simulación, es recomendable que leas la guía que tienes disponible [aquí](https://github.com/pammacdotnet/play-with-docker.github.io/raw/master/_posts/2020-02-23-geant4lab.markdown). También puedes visitar la [versión live](https://training.play-with-docker.com/geant4lab/) en el [sitio oficial de pruebas y formación](http://training.play-with-docker.com/) de Docker.

## Visualización de la simulación

Una vez que ya tenemos la simulación, sólo tenemos que visualizarla con Instant Player (o ParaView o view3dscene). Para ello, tenemos que abrir el fichero que, encontraremos en el directorio de trabajo, con Instant Player o, si ya está el fichero abierto, puedes simplemente recargarlo (File ➡️ Reload). Lo que vemos en el visualizador son trazas de partículas, es decir, sólo vemos partículas que estén en movimiento. Las partículas o materia paradas, no se muestran.

## Visualización con estándares web

Pero quizás, la visualización más _developer-friendly_ es la basada en HTML y estándares Web (WebGL). Luego ejecuta este comando para convertir el archivo _.wrl a _.html.

```bash
/usr/bin/python3 ./wrl2html.py
```

Esto generará el archivo `simulation.html` en tu carpeta de trabajo local.

Abre esta página web con cualquier navegador o directamente en VS Code con la extensión vscode-preview-server (comando Launch on browser) que has instalado al principio. Cada vez que se reescriba este fichero, la visualización se actualizará. En principio, la extensión preview-server debería actualizarla sin que digamos nada, pero si no lo hace, puedes guardar cualquier fichero (touch al directorio) y así debería actualizarse. También puedes simplemente indicarle al navegador (si estas usando un navegador externo) que refresque la página web.

-----------------

Simulación

This is an HTML embed test.

<div>
    	<meta http-equiv='Content-Type' content='text/html;charset=utf-8'>
    	</meta>
    	<link rel='stylesheet' type='text/css' href='http://www.x3dom.org/x3dom/release/x3dom.css'>
    	</link>
    	<script type='text/javascript' src='http://www.x3dom.org/x3dom/release/x3dom.js'></script>
    <body bgcolor='000'>
    	<x3d id='someUniqueId' showStat='false' showLog='false' x='0px' y='0px' width='1' height='1'>
    		<scene DEF='scene'>
    			<viewpoint position='0 0 4350.66'></viewpoint>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 1 1' transparency='0.75'></material>
    				</appearance>
    				<indexedFaceSet solid='false'
    					coordIndex='0 3 2 1 -1 4 7 3 0 -1 7 6 2 3 -1 6 5 1 2 -1 5 4 0 1 -1 4 5 6 7 -1'>
    					<coordinate
    						point='-605 -605 -2000 605 -605 -2000 605 605 -2000 -605 605 -2000 -605 -605 2000 605 -605 2000 605 605 2000 -605 605 2000'>
    					</coordinate>
    				</indexedFaceSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='0 0.9 1' transparency='0.75'></material>
    				</appearance>
    				<indexedFaceSet solid='false'
    					coordIndex='0 3 2 1 -1 4 7 3 0 -1 7 6 2 3 -1 6 5 1 2 -1 5 4 0 1 -1 4 5 6 7 -1'>
    					<coordinate
    						point='-600 -600 900 600 -600 900 600 600 900 -600 600 900 -600 -600 1100 600 -600 1100 600 600 1100 -600 600 1100'>
    					</coordinate>
    				</indexedFaceSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 1 1' emissiveColor='1 1 1'></material>
    				</appearance>
    				<indexedLineSet coordIndex='0 1 2 3 4 5 6 -1'>
    					<coordinate
    						point='0 0 -900 -258.919 -224.156 900 -265.457 -229.816 945.451 -259.127 -184.583 997.138 -262.224 -189.716 1002.85 -225.787 -42.3382 900 -65.7429 605 448.256'>
    					</coordinate>
    				</indexedLineSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 0 0' emissiveColor='1 0 0'></material>
    				</appearance>
    				<indexedLineSet coordIndex='0 1 -1'>
    					<coordinate point='-262.224 -189.716 1002.85 -262.271 -189.817 1002.94'></coordinate>
    				</indexedLineSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 0 0' emissiveColor='1 0 0'></material>
    				</appearance>
    				<indexedLineSet coordIndex='0 1 -1'>
    					<coordinate point='-259.127 -184.583 997.138 -259.009 -184.172 997.317'></coordinate>
    				</indexedLineSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 0 0' emissiveColor='1 0 0'></material>
    				</appearance>
    				<indexedLineSet coordIndex='0 1 -1'>
    					<coordinate point='-265.457 -229.816 945.451 -265.643 -230.301 945.967'></coordinate>
    				</indexedLineSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 1 1' emissiveColor='1 1 1'></material>
    				</appearance>
    				<indexedLineSet coordIndex='0 1 2 3 4 5 6 7 8 9 10 11 -1'>
    					<coordinate
    						point='0 0 -900 -460.088 -273.035 900 -473.214 -280.824 951.352 -472.666 -271.482 951.188 -439.741 -217.331 938.905 -429.133 -186.362 925.278 -386.603 -190.34 963.048 -132.877 -179.486 992.525 -92.598 -189.899 1094.44 -96.4547 -191.255 1098.95 -94.1894 -189.407 1100 605 381.018 1425.52'>
    					</coordinate>
    				</indexedLineSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 0 0' emissiveColor='1 0 0'></material>
    				</appearance>
    				<indexedLineSet coordIndex='0 1 -1'>
    					<coordinate point='-96.4547 -191.255 1098.95 -96.462 -191.259 1098.95'></coordinate>
    				</indexedLineSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 0 0' emissiveColor='1 0 0'></material>
    				</appearance>
    				<indexedLineSet coordIndex='0 1 -1'>
    					<coordinate point='-92.598 -189.899 1094.44 -92.5947 -189.899 1094.44'></coordinate>
    				</indexedLineSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 0 0' emissiveColor='1 0 0'></material>
    				</appearance>
    				<indexedLineSet coordIndex='0 1 -1'>
    					<coordinate point='-132.877 -179.486 992.525 -132.873 -179.485 992.52'></coordinate>
    				</indexedLineSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 0 0' emissiveColor='1 0 0'></material>
    				</appearance>
    				<indexedLineSet coordIndex='0 1 -1'>
    					<coordinate point='-386.603 -190.34 963.048 -386.604 -190.341 963.049'></coordinate>
    				</indexedLineSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 0 0' emissiveColor='1 0 0'></material>
    				</appearance>
    				<indexedLineSet coordIndex='0 1 -1'>
    					<coordinate point='-429.133 -186.362 925.278 -429.144 -186.304 925.227'></coordinate>
    				</indexedLineSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 0 0' emissiveColor='1 0 0'></material>
    				</appearance>
    				<indexedLineSet coordIndex='0 1 -1'>
    					<coordinate point='-439.741 -217.331 938.905 -439.741 -217.331 938.905'></coordinate>
    				</indexedLineSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 0 0' emissiveColor='1 0 0'></material>
    				</appearance>
    				<indexedLineSet coordIndex='0 1 -1'>
    					<coordinate point='-472.666 -271.482 951.188 -472.67 -271.479 951.19'></coordinate>
    				</indexedLineSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 0 0' emissiveColor='1 0 0'></material>
    				</appearance>
    				<indexedLineSet coordIndex='0 1 2 -1'>
    					<coordinate
    						point='-473.214 -280.824 951.352 -473.537 -281.375 952.535 -473.727 -281.696 952.814'>
    					</coordinate>
    				</indexedLineSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 1 1' emissiveColor='1 1 1'></material>
    				</appearance>
    				<indexedLineSet coordIndex='0 1 2 3 -1'>
    					<coordinate point='0 0 -900 269.166 378.181 900 299.073 420.201 1100 430.602 605 1979.57'>
    					</coordinate>
    				</indexedLineSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 1 1' emissiveColor='1 1 1'></material>
    				</appearance>
    				<indexedLineSet coordIndex='0 1 2 3 4 5 6 -1'>
    					<coordinate
    						point='0 0 -900 50.307 220.256 900 50.651 221.762 912.308 52.6658 252.52 951.672 89.5772 336.796 927.879 87.6965 384.993 900 79.1115 605 772.738'>
    					</coordinate>
    				</indexedLineSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 0 0' emissiveColor='1 0 0'></material>
    				</appearance>
    				<indexedLineSet coordIndex='0 1 -1'>
    					<coordinate point='89.5772 336.796 927.879 89.5871 336.798 927.884'></coordinate>
    				</indexedLineSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 0 0' emissiveColor='1 0 0'></material>
    				</appearance>
    				<indexedLineSet coordIndex='0 1 2 -1'>
    					<coordinate point='52.6658 252.52 951.672 52.5549 252.637 952.319 52.5966 252.568 952.348'>
    					</coordinate>
    				</indexedLineSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 0 0' emissiveColor='1 0 0'></material>
    				</appearance>
    				<indexedLineSet coordIndex='0 1 -1'>
    					<coordinate point='50.651 221.762 912.308 50.649 221.571 912.508'></coordinate>
    				</indexedLineSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 1 1' emissiveColor='1 1 1'></material>
    				</appearance>
    				<indexedLineSet coordIndex='0 1 2 3 -1'>
    					<coordinate point='0 0 -900 173.044 -331.584 900 192.271 -368.427 1100 278.793 -534.219 2000'>
    					</coordinate>
    				</indexedLineSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 1 1' emissiveColor='1 1 1'></material>
    				</appearance>
    				<indexedLineSet coordIndex='0 1 2 3 4 5 6 -1'>
    					<coordinate
    						point='0 0 -900 351.502 -330.937 900 353.806 -333.106 911.797 353.657 -338.405 951.704 336.08 -343.324 1052.31 349.434 -355.262 1100 601.427 -580.549 2000'>
    					</coordinate>
    				</indexedLineSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 0 0' emissiveColor='1 0 0'></material>
    				</appearance>
    				<indexedLineSet coordIndex='0 1 -1'>
    					<coordinate point='336.08 -343.324 1052.31 335.943 -343.272 1052.38'></coordinate>
    				</indexedLineSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 0 0' emissiveColor='1 0 0'></material>
    				</appearance>
    				<indexedLineSet coordIndex='0 1 -1'>
    					<coordinate point='353.657 -338.405 951.704 353.666 -338.41 951.706'></coordinate>
    				</indexedLineSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 0 0' emissiveColor='1 0 0'></material>
    				</appearance>
    				<indexedLineSet coordIndex='0 1 -1'>
    					<coordinate point='353.806 -333.106 911.797 353.819 -333.109 911.798'></coordinate>
    				</indexedLineSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 1 1' emissiveColor='1 1 1'></material>
    				</appearance>
    				<indexedLineSet coordIndex='0 1 2 3 4 -1'>
    					<coordinate
    						point='0 0 -900 535.587 -184.184 900 565.206 -194.37 999.543 600 -217.586 997.892 605 -220.922 997.654'>
    					</coordinate>
    				</indexedLineSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 0 0' emissiveColor='1 0 0'></material>
    				</appearance>
    				<indexedLineSet coordIndex='0 1 -1'>
    					<coordinate point='565.206 -194.37 999.543 565.142 -194.221 1000.67'></coordinate>
    				</indexedLineSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 1 1' emissiveColor='1 1 1'></material>
    				</appearance>
    				<indexedLineSet coordIndex='0 1 2 3 4 5 -1'>
    					<coordinate
    						point='0 0 -900 -447.638 481.638 900 -462.737 497.884 960.7514 -437.947 528.641 989.047 -400.586 445.578 900 71.9544 -605 -226.263'>
    					</coordinate>
    				</indexedLineSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 0 0' emissiveColor='1 0 0'></material>
    				</appearance>
    				<indexedLineSet coordIndex='0 1 2 -1'>
    					<coordinate point='-437.947 528.641 989.047 -437.759 529.042 989.432 -437.757 529.055 989.423'>
    					</coordinate>
    				</indexedLineSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 0 0' emissiveColor='1 0 0'></material>
    				</appearance>
    				<indexedLineSet coordIndex='0 1 2 -1'>
    					<coordinate point='-462.737 497.884 960.7514 -463.252 497.784 961.327 -463.232 497.895 961.441'>
    					</coordinate>
    				</indexedLineSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 1 1' emissiveColor='1 1 1'></material>
    				</appearance>
    				<indexedLineSet coordIndex='0 1 2 3 4 -1'>
    					<coordinate
    						point='0 0 -900 -275.572 429.983 900 -281.808 439.713 940.7532 -322.4 368.03 900 -605 -131.032 616.421'>
    					</coordinate>
    				</indexedLineSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 0 0' emissiveColor='1 0 0'></material>
    				</appearance>
    				<indexedLineSet coordIndex='0 1 2 -1'>
    					<coordinate point='-281.808 439.713 940.7532 -281.853 440.231 942.042 -281.86 440.241 942.033'>
    					</coordinate>
    				</indexedLineSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 1 1' emissiveColor='1 1 1'></material>
    				</appearance>
    				<indexedLineSet coordIndex='0 1 2 3 -1'>
    					<coordinate point='0 0 -900 -229.728 314.015 900 -255.254 348.906 1100 -370.118 505.913 2000'>
    					</coordinate>
    				</indexedLineSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 1 1' emissiveColor='1 1 1'></material>
    				</appearance>
    				<indexedLineSet coordIndex='0 1 2 3 4 5 6 7 8 9 10 -1'>
    					<coordinate
    						point='0 0 -900 -523.116 -340.454 900 -529.088 -344.34 920.549 -527.702 -346.999 926.123 -496.291 -375.465 915.58 -548.531 -299.346 948.622 -562.766 -150.977 953.774 -494.475 -193.248 929.956 -529.219 -203.454 935.211 -600 -244.127 981.468 -605 -247 984.735'>
    					</coordinate>
    				</indexedLineSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 0 0' emissiveColor='1 0 0'></material>
    				</appearance>
    				<indexedLineSet coordIndex='0 1 -1'>
    					<coordinate point='-529.219 -203.454 935.211 -529.219 -203.454 935.211'></coordinate>
    				</indexedLineSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 0 0' emissiveColor='1 0 0'></material>
    				</appearance>
    				<indexedLineSet coordIndex='0 1 -1'>
    					<coordinate point='-494.475 -193.248 929.956 -494.469 -193.249 929.955'></coordinate>
    				</indexedLineSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 0 0' emissiveColor='1 0 0'></material>
    				</appearance>
    				<indexedLineSet coordIndex='0 1 -1'>
    					<coordinate point='-562.766 -150.977 953.774 -562.774 -150.961 953.777'></coordinate>
    				</indexedLineSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 0 0' emissiveColor='1 0 0'></material>
    				</appearance>
    				<indexedLineSet coordIndex='0 1 -1'>
    					<coordinate point='-548.531 -299.346 948.622 -548.532 -299.346 948.623'></coordinate>
    				</indexedLineSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 0 0' emissiveColor='1 0 0'></material>
    				</appearance>
    				<indexedLineSet coordIndex='0 1 -1'>
    					<coordinate point='-496.291 -375.465 915.58 -496.127 -375.636 915.514'></coordinate>
    				</indexedLineSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 0 0' emissiveColor='1 0 0'></material>
    				</appearance>
    				<indexedLineSet coordIndex='0 1 2 -1'>
    					<coordinate
    						point='-527.702 -346.999 926.123 -527.784 -347.087 926.85 -527.799 -347.039 926.967'>
    					</coordinate>
    				</indexedLineSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 0 0' emissiveColor='1 0 0'></material>
    				</appearance>
    				<indexedLineSet coordIndex='0 1 -1'>
    					<coordinate point='-529.088 -344.34 920.549 -529.332 -344.261 920.695'></coordinate>
    				</indexedLineSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 1 1' emissiveColor='1 1 1'></material>
    				</appearance>
    				<indexedLineSet coordIndex='0 1 2 3 4 5 6 7 8 9 10 11 12 -1'>
    					<coordinate
    						point='0 0 -900 69.9045 -68.5845 900 71.2403 -69.8951 934.397 68.672 -70.1176 949.118 89.4909 -74.3881 929.663 105.193 -100.7567 903.952 103.698 -104.351 950.595 99.4389 -122.939 929.03 102.087 -129.3 931.074 101.939 -128.861 929.342 145.794 -139.264 938.489 123.49 -117.821 900 -605 582.539 -357.122'>
    					</coordinate>
    				</indexedLineSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 0 0' emissiveColor='1 0 0'></material>
    				</appearance>
    				<indexedLineSet coordIndex='0 1 -1'>
    					<coordinate point='145.794 -139.264 938.489 145.795 -139.265 938.49'></coordinate>
    				</indexedLineSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 0 0' emissiveColor='1 0 0'></material>
    				</appearance>
    				<indexedLineSet coordIndex='0 1 -1'>
    					<coordinate point='101.939 -128.861 929.342 101.938 -128.86 929.342'></coordinate>
    				</indexedLineSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 0 0' emissiveColor='1 0 0'></material>
    				</appearance>
    				<indexedLineSet coordIndex='0 1 -1'>
    					<coordinate point='102.087 -129.3 931.074 102.088 -129.302 931.076'></coordinate>
    				</indexedLineSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 0 0' emissiveColor='1 0 0'></material>
    				</appearance>
    				<indexedLineSet coordIndex='0 1 -1'>
    					<coordinate point='99.4389 -122.939 929.03 99.4384 -122.939 929.029'></coordinate>
    				</indexedLineSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 0 0' emissiveColor='1 0 0'></material>
    				</appearance>
    				<indexedLineSet coordIndex='0 1 -1'>
    					<coordinate point='103.698 -104.351 950.595 103.698 -104.348 950.608'></coordinate>
    				</indexedLineSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 0 0' emissiveColor='1 0 0'></material>
    				</appearance>
    				<indexedLineSet coordIndex='0 1 -1'>
    					<coordinate point='105.193 -100.7567 903.952 105.208 -100.7588 903.909'></coordinate>
    				</indexedLineSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 0 0' emissiveColor='1 0 0'></material>
    				</appearance>
    				<indexedLineSet coordIndex='0 1 -1'>
    					<coordinate point='89.4909 -74.3881 929.663 89.4928 -74.3857 929.662'></coordinate>
    				</indexedLineSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 0 0' emissiveColor='1 0 0'></material>
    				</appearance>
    				<indexedLineSet coordIndex='0 1 2 3 -1'>
    					<coordinate
    						point='68.672 -70.1176 949.118 68.2677 -70.0947 950.482 68.0323 -69.4486 950.613 68.0263 -69.4489 950.613'>
    					</coordinate>
    				</indexedLineSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 0 0' emissiveColor='1 0 0'></material>
    				</appearance>
    				<indexedLineSet coordIndex='0 1 -1'>
    					<coordinate point='71.2403 -69.8951 934.397 71.2563 -69.897 934.402'></coordinate>
    				</indexedLineSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 1 1' emissiveColor='1 1 1'></material>
    				</appearance>
    				<indexedLineSet coordIndex='0 1 2 3 4 -1'>
    					<coordinate
    						point='0 0 -900 461.988 -341.137 900 470.7531 -347.593 934.064 413.995 -287.226 900 -424.581 605 396.523'>
    					</coordinate>
    				</indexedLineSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 0 0' emissiveColor='1 0 0'></material>
    				</appearance>
    				<indexedLineSet coordIndex='0 1 2 -1'>
    					<coordinate point='470.7531 -347.593 934.064 471.226 -348.019 935.368 471.216 -348.536 935.673'>
    					</coordinate>
    				</indexedLineSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 1 1' emissiveColor='1 1 1'></material>
    				</appearance>
    				<indexedLineSet coordIndex='0 1 2 3 -1'>
    					<coordinate point='0 0 -900 591.767 -185.235 900 600 -187.812 925.042 605 -189.377 940.25'>
    					</coordinate>
    				</indexedLineSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 1 1' emissiveColor='1 1 1'></material>
    				</appearance>
    				<indexedLineSet coordIndex='0 1 2 3 4 5 6 7 -1'>
    					<coordinate
    						point='0 0 -900 -54.7428 -266.392 900 -55.8312 -271.689 935.788 -37.5495 -144.915 975.857 -24.7501 -107.483 994.573 -45.5781 -131.061 959.035 -72.5792 -137.277 972.654 -73.648 -188.349 918.136'>
    					</coordinate>
    				</indexedLineSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 0 0' emissiveColor='1 0 0'></material>
    				</appearance>
    				<indexedLineSet coordIndex='0 1 -1'>
    					<coordinate point='-73.648 -188.349 918.136 -73.6423 -188.339 918.066'></coordinate>
    				</indexedLineSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 0 0' emissiveColor='1 0 0'></material>
    				</appearance>
    				<indexedLineSet coordIndex='0 1 -1'>
    					<coordinate point='-72.5792 -137.277 972.654 -72.5841 -137.275 972.66'></coordinate>
    				</indexedLineSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 0 0' emissiveColor='1 0 0'></material>
    				</appearance>
    				<indexedLineSet coordIndex='0 1 -1'>
    					<coordinate point='-45.5781 -131.061 959.035 -45.5758 -131.064 959.025'></coordinate>
    				</indexedLineSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 0 0' emissiveColor='1 0 0'></material>
    				</appearance>
    				<indexedLineSet coordIndex='0 1 -1'>
    					<coordinate point='-24.7501 -107.483 994.573 -24.6588 -107.28 994.716'></coordinate>
    				</indexedLineSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 0 0' emissiveColor='1 0 0'></material>
    				</appearance>
    				<indexedLineSet coordIndex='0 1 -1'>
    					<coordinate point='-37.5495 -144.915 975.857 -37.5499 -144.914 975.857'></coordinate>
    				</indexedLineSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 0 0' emissiveColor='1 0 0'></material>
    				</appearance>
    				<indexedLineSet coordIndex='0 1 2 -1'>
    					<coordinate
    						point='-55.8312 -271.689 935.788 -55.9283 -272.288 936.834 -55.9103 -272.406 936.913'>
    					</coordinate>
    				</indexedLineSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 1 1' emissiveColor='1 1 1'></material>
    				</appearance>
    				<indexedLineSet coordIndex='0 1 2 3 -1'>
    					<coordinate point='0 0 -900 -174.563 -513.47 900 -193.959 -570.522 1100 -205.68 -605 1220.86'>
    					</coordinate>
    				</indexedLineSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 1 1' emissiveColor='1 1 1'></material>
    				</appearance>
    				<indexedLineSet coordIndex='0 1 2 3 -1'>
    					<coordinate point='0 0 -900 106.733 -589.349 900 108.662 -600 932.53 109.567 -605 947.801'>
    					</coordinate>
    				</indexedLineSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 1 1' emissiveColor='1 1 1'></material>
    				</appearance>
    				<indexedLineSet coordIndex='0 1 2 3 4 5 6 7 8 9 -1'>
    					<coordinate
    						point='0 0 -900 -119.556 353.942 900 -123.91 366.83 965.543 -129.98 375.079 1001.19 -116.456 353.239 1062.23 -114.87 347.357 1070.95 -111.482 344.973 1073.46 -101.881 330.532 1096.36 -100.808 326.864 1100 164.599 -580.261 2000'>
    					</coordinate>
    				</indexedLineSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 0 0' emissiveColor='1 0 0'></material>
    				</appearance>
    				<indexedLineSet coordIndex='0 1 -1'>
    					<coordinate point='-101.881 330.532 1096.36 -101.88 330.534 1096.36'></coordinate>
    				</indexedLineSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 0 0' emissiveColor='1 0 0'></material>
    				</appearance>
    				<indexedLineSet coordIndex='0 1 -1'>
    					<coordinate point='-111.482 344.973 1073.46 -111.461 344.97 1073.45'></coordinate>
    				</indexedLineSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 0 0' emissiveColor='1 0 0'></material>
    				</appearance>
    				<indexedLineSet coordIndex='0 1 -1'>
    					<coordinate point='-114.87 347.357 1070.95 -114.957 347.322 1071.04'></coordinate>
    				</indexedLineSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 0 0' emissiveColor='1 0 0'></material>
    				</appearance>
    				<indexedLineSet coordIndex='0 1 -1'>
    					<coordinate point='-116.456 353.239 1062.23 -116.454 353.246 1062.23'></coordinate>
    				</indexedLineSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 0 0' emissiveColor='1 0 0'></material>
    				</appearance>
    				<indexedLineSet coordIndex='0 1 -1'>
    					<coordinate point='-129.98 375.079 1001.19 -130.196 375.399 1001.41'></coordinate>
    				</indexedLineSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 0 0' emissiveColor='1 0 0'></material>
    				</appearance>
    				<indexedLineSet coordIndex='0 1 -1'>
    					<coordinate point='-123.91 366.83 965.543 -123.908 366.83 965.544'></coordinate>
    				</indexedLineSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 1 1' emissiveColor='1 1 1'></material>
    				</appearance>
    				<indexedLineSet coordIndex='0 1 2 3 4 5 6 7 8 -1'>
    					<coordinate
    						point='0 0 -900 -309.821 -490.869 900 -310.7528 -492.305 905.266 -343.028 -512.991 963.355 -317.414 -541.568 986.931 -372.112 -476.935 1017.09 -381.963 -517.274 1053.92 -388.096 -534.28 1100 -413.603 -605 1291.62'>
    					</coordinate>
    				</indexedLineSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 0 0' emissiveColor='1 0 0'></material>
    				</appearance>
    				<indexedLineSet coordIndex='0 1 -1'>
    					<coordinate point='-381.963 -517.274 1053.92 -381.963 -517.274 1053.92'></coordinate>
    				</indexedLineSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 0 0' emissiveColor='1 0 0'></material>
    				</appearance>
    				<indexedLineSet coordIndex='0 1 -1'>
    					<coordinate point='-372.112 -476.935 1017.09 -372.121 -476.911 1017.09'></coordinate>
    				</indexedLineSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 0 0' emissiveColor='1 0 0'></material>
    				</appearance>
    				<indexedLineSet coordIndex='0 1 -1'>
    					<coordinate point='-317.414 -541.568 986.931 -317.206 -541.804 987.026'></coordinate>
    				</indexedLineSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 0 0' emissiveColor='1 0 0'></material>
    				</appearance>
    				<indexedLineSet coordIndex='0 1 2 -1'>
    					<coordinate
    						point='-343.028 -512.991 963.355 -343.728 -512.99 963.918 -343.619 -513.011 964.077'>
    					</coordinate>
    				</indexedLineSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 0 0' emissiveColor='1 0 0'></material>
    				</appearance>
    				<indexedLineSet coordIndex='0 1 -1'>
    					<coordinate point='-310.7528 -492.305 905.266 -310.675 -492.303 905.306'></coordinate>
    				</indexedLineSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 1 1' emissiveColor='1 1 1'></material>
    				</appearance>
    				<indexedLineSet coordIndex='0 1 2 3 4 5 -1'>
    					<coordinate
    						point='0 0 -900 83.0509 521.405 900 84.6937 531.719 935.605 74.4648 515.937 900.654 74.3166 515.342 900 -204.642 -605 -331.548'>
    					</coordinate>
    				</indexedLineSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 0 0' emissiveColor='1 0 0'></material>
    				</appearance>
    				<indexedLineSet coordIndex='0 1 -1'>
    					<coordinate point='74.4648 515.937 900.654 74.4647 515.937 900.654'></coordinate>
    				</indexedLineSet>
    			</shape>
    			<shape>
    				<appearance>
    					<material diffuseColor='1 0 0' emissiveColor='1 0 0'></material>
    				</appearance>
    				<indexedLineSet coordIndex='0 1 2 -1'>
    					<coordinate point='84.6937 531.719 935.605 84.8167 532.173 937.044 84.3212 532.593 937.616'>
    					</coordinate>
    				</indexedLineSet>
    			</shape>
    		</scene>
    	</x3d>
    </body>
</div>