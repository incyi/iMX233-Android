# iMX233-Android
Android build on iMX233 instructions (in Spanish)

The website linuxencaja.net is since 2 years down. And I wanted to build Android for my iMX233 OLinuXino board from Olimex. So I started to search for instructions and found this in Spanish. First I want to try it for myself. Then I will translate it to English.

Copied from : http://linuxencaja.net/wiki/Implementaci%C3%B3n_de_Android_en_procesador_i.MX233
https://web.archive.org/web/20120609062535/http://linuxencaja.net/wiki/Implementación_de_Android_en_procesador_i.MX233

Implementación de Android en procesador i.MX233
Contents
1 Implementación de Android en procesador i.MX233
1.1 Importante
1.2 Prerrequisitos para la implementación
1.3 Descomprimiendo imx-android y herramientas utiles
1.4 Obteniendo Código Fuente
1.5 Aplicar Patches
1.6 Construcción del Uboot
1.7 Imagen de Android
1.8 Configuración del Kernel para Android y generación del Zimage
1.9 Implementación del Android rootFS
2 Copia a SD
2.1 Carga Zimage
2.2 Carga RootFS Android
2.3 Configuración del Busybox
2.4 Referencias
Implementación de Android en procesador i.MX233
Se ha realizado la implementación del sistema Android en el procesador i.MX233 de Freescale realizando pruebas en la tarjeta de desarrollo Chumby Hacker Board. Con ello se pretende extender la funcionalidad de la tarjeta para ejecutar aplicaciones Java en modo consola. Para poder realizar dicha implementación, fué necesario adaptar el kernel de Linux para que soportara la funcionalidad del sistema Android y además fué necesario también compilar el sistema de archivos raiz de dicho para una arquitectura ARM similar a la arquitectura en la que se está trabajando.

Importante
Para realizar estas instrucciones se recomienda antes haber cargado Linux ejecutando y compilando las instrucciones de esta página.

A continuación se muestran los pasos realizados para poder compilar dicho sistema operativo para el procesador:

Prerrequisitos para la implementación
Inicialmente es necesario utilizar un Kernel de Linux con los patches necesarios para funcionar con el procesador i.MX233, se puede utilizar el kernel de la tarjeta TuxRail. Los pasos para los drivers de la tarjeta microSD y adaptación del kernel de Linux se encuentran en esta página.
Descargar e instalar los siguientes paquetes:
sudo apt-get install nfs-kernel-server patch g++ rpm zlib1g-dev m4 bison libncurses5-dev gettext build-essential tcl intltool libxml2-dev minicom tftpd  xinetd curl
Descargar imx-android-R7 de esta página.
Descomprimiendo imx-android y herramientas utiles
cd /opt (o  cualquier otro directorio que le guste)
sudo tar xzvf imx-android-r7.tar.gz
cd imx-android-r7/code
sudo tar xzvf R7.tar.gz
Asumiendo que se descomprimió en /opt:

cd /opt/imx-android-r7/tool
sudo tar xzvf gcc-4.1.2-glibc-2.5-nptl-3.tar.gz -C /opt
A continuacion se abre el archivo .bashrc

gedit ~/.bashrc
Y se agregan las siguientes dos lineas:

export ARCH=arm
export CROSS_COMPILE=/opt/gcc-4.1.2-glibc-2.5-nptl-3/arm-none-linux-gnueabi/bin/arm-none-linux-gnueabi-
Obteniendo Código Fuente
Para obtener el código fuente seguimos los siguientes pasos (El ultimo paso puede demorar varias horas descargando. Asegurarse de tener una buena conexión a internet):

cd ~
mkdir myandroid
cd myandroid
curl http://android.git.kernel.org/repo > ./repo   
chmod a+x ./repo
./repo init -u git://android.git.kernel.org/platform/manifest.git -b eclair
cp /opt/imx-android-r7/code/R7/default.xml .repo/manifests/default.xml 
./repo sync
Ademas, obtener un código fuente limpio de kernel.org:

cd myandroid
git clone git://git.kernel.org/pub/scm/linux/kernel/git/stable/linux-2.6.28.y.git kernel_imx
Descargar el código fuente de U-BOOT

cd myandroid/bootable/bootloader
git clone git://git.denx.de/u-boot.git uboot-imx
Aplicar Patches
cd ~/myandroid
. /opt/imx-android-r7/code/R7/and_patch.sh
help
Se debe ver la función "c_patch" disponible en la consola.

c_patch /opt/imx-android-r7/code/R7 imx_r7
Si todo sale bien, este mensaje se oservará:

Success: Now you can build android code for FSL i.MX platform
Los patches quedan situados en "/opt/imx-android-r7/code/R7".

Construcción del Uboot
Esta construcción es necesario para que los patches se apliquen:

cd ~/myandroid/bootable/bootloader/uboot-imx
echo $ARCH && echo $CROSS_COMPILE
make mx51_bbg_android_config
make
Imagen de Android
La imagen del sistema de archivos de android se construye con los siguientes pasos:

cd ~/myandroid
make PRODUCT-imx51_BBG-eng | tee build_imx51_BBG_android.log
Configuración del Kernel para Android y generación del Zimage
Para este caso, se usara el kernel de Tuxrail donde los pasos para su configuración básica se encuentran en esta pagina.

Se ingresa al directorio donde se encuentra el kernel:

cd camino-a-kernel-tuxrail/

y se ingresa al menu de configuración

alias crossmake='make ARCH=arm CROSS_COMPILE=/home/<usuario>/myandroid/prebuilt/linux-x86/toolchain/arm-eabi-4.4.0/bin/arm-eabi-'
crossmake menuconfig
Y a continuación entramos a "device drivers" con la tecla 'enter'. Luego se selecciona "staging drivers" con 'y'. Se debería observar algo así:

 │ │   [ ] Accessibility support  --->                                   │ │  
 │ │    < > InfiniBand support (NEW)  --->                               │ │  
 │ │    [ ] EDAC (Error Detection And Correction) reporting (NEW)  --->  │ │  
 │ │    <*> Real Time Clock  --->                                        │ │  
 │ │    [ ] DMA Engine support  --->                                     │ │  
 │ │    [ ] Auxiliary Display support  --->                              │ │  
 │ │    < > Userspace I/O drivers  --->                                  │ │  
 │ │        TI VLYNQ  --->                                               │ │  
 │ │    [*] Staging drivers  --->                                        │ │  
 │ │    [*] X86 Platform Specific Device Drivers (NEW)  --->   

Luego entramos a "Staging drivers" con 'enter'. A continuación, entramos al menú de Android oprimiendo 'enter' en "Android". Seleccionamos "Android" con 'y' y a continuación escogemos todas las características con 'y' nuevamente. observándose lo siguiente:

 │ │    [*]   Android Binder IPC Driver                                  │ │  
 │ │    <*>   Android log driver                                         │ │  
 │ │    [*]   Android RAM buffer console                                 │ │  
 │ │    [*]     Enable verbose console messages on Android RAM console  │ │  
 │ │    [*]     Start Android RAM console early                          │ │  
 │ │    (0)       Android RAM console virtual address (NEW)              │ │  
 │ │    (0)       Android RAM console buffer size (NEW)                  │ │  
 │ │    [*]   Timed output class driver (NEW)                            │ │  
 │ │    <*>     Android timed gpio driver                                │ │  
 │ │    [*]   Android Low Memory Killer                                  │ │  
Finalmente salimos y guardamos todos los cambios.

Compilamos el kernel y generamos el archivo Zimage:

crossmake -j3
Implementación del Android rootFS
Primero abrimos init.rc:

gedit ~/myandroid/out/target/product/generic/root/init.rc
Se comentan las siguientes lineas con '#':

mount yaffs2 mtd @ system / system
mount yaffs2 mtd @ system / system ro remount
mount yaffs2 mtd @ userdata / data nosuid nodev
mount yaffs2 mtd @ cache / cache nosuid nodev
Luego, se descargan los arhivos mm/ashmem.c y include/linux/ashmem.h de esta pagina. Estos archivos se deben pegar en las mismas carpetas (mm e include/linux) en la copia del kernel de nuestro disco duro (para este caso seria la ubicación del kernel de tuxrail). Además, se debe agregar la siguiente linea de código al mm.h ubicada en el directorio "camino-al-kernel/include/linux":

  void shmem_set_file (struct vm_area_struct * vma, struct file * file);
Por otro lado agregamos las siguientes lineas en el archivo shmem.c ubicada en "camino-al-kernel/mm":

 
   void shmem_set_file (struct vm_area_struct * vma, struct file * file)
            {
                if (vma-> vm_file)
                    fput (vma-> vm_file);
                vma-> vm_file = file;
                vma-> vm_ops = & shmem_vm_ops;
            }
En el Makefile ubicado en "mm/" se agrega "ashmem.o" (sin comillas) en la seccion obj-y y se recompila el kernel:

cd camino-al-kernel
crossmake -j3
Copia a SD
Carga Zimage
Para grabar la imagen del kernel y el sistema de archivos se generan las particiones usando "fdisk" como se muestra en [http://linuxencaja.net/wiki/Linux_Boot#Particiones_MicroSD esta pagina]. Además, se carga el kernel como se ilustra en esta pagina y utilizando el siguiente commandline (linux_prep/stmp378x_dev.txt):

console=ttyAM0,115200 init=/init root=/dev/mmcblk0p2 ro rootfstype=ext2 rootdelay=1 ssp1=mmc line=1 mem=64M androidboot.console=ttyAM0
Carga RootFS Android
Nos ubicamos en el directorio que contiene a root:

cd myandroid/out/target/product/generic
Copiamos las carpetas "system" y "data" a root:

sudo cp -r system data root/
Finalmente se copia el contenido de la carpeta "root" en la partición ext3 dedicada a almacenar el sistema de archivos:

cd root
sudo cp -avr * /media/etiqueta-particion-sd
Configuración del Busybox
Una vez que android esta en funcionamiento de la tarjeta, se puede ver que la consola es muy básica. Para mejorarla se necesita instalar el paquete "Busybox" el cual contiene sh que es la consola de todo kernel de linux. El ejecutable "Busybox" se puede obtener de culaquier rootFS (como el de Freescale o Tuxrail) en /bin o desde la pagina de busybox. Este ejecutable se debe copiar en "/root/system/bin":

cp Busybox /media/etiqueta-sd/system/bin
Y se edita el archivo "init.rc":

gedit /media/etiqueta-particion-sd/init.rc
Y la linea:

service console /system/bin/sh
se cambia por:

service console /system/bin/busybox sh
Referencias
I.MX Android R7 User guide - Toolchain and android source installation.
Blog - ChinaUnix - Android porting to i.MX233 (Chinese): link. (http://blog.chinaunix.net/uid/20146040.html)
Busybox Android - Installation of busybox: link. (http://www.busybox.net/FAQ.html)

