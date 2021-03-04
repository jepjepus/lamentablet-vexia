# lamentablet-vexia cargador 9v

Primeras investigaciones.

La tablet Vexia solo permite arrancar en modo UEFI, nada de modo Legacy. Para rizar más el rizo, la BIOS es UEFI 32 bits. Necesita archivos especiales para arranque.

Al arrancar en modo uefi,hay que usar el archivo "bootia32.efi" en las tablet de 9v, colocados en una carpeta llamada EFI, y dentro de esta otra llamada BOOT.

Los drivers son casi imposibles de compilar o encontrar hoy día, tras muchas pruebas para encontrar los drivers del wifi, sonido o táctil se opta por usar el kernel que se compiló en su día con las tablets vexia con el sistema Guadalinex Edu. En las tablet de cargador negro de 9v se usó el kernel 3.4.43.

El driver táctil no es un driver del kernel, es un driver del XORG gráfico, viene configurado en el kernel pero necesita un paquete que incluye la configuración y el driver en sí, alojado en los repositorios de guadalinex EDU. Si instalas el paquete tal cual, el sistema no arranca, pues ese driver fue compilado para un XORG anterior. Es imposible volver a un XORG anterior en minino, por lo tanto se ha optado por compilar el driver MULTITOUCH para el XORG de minino (también se puede instalar con un apt-get install xserver-xorg-video-multitouch). El único problema es que esta acción hace que el táctil funcione parecido a un touchpad, no aparece el cursor donde pulsas en la pantalla taćtil, tienes que arrastrarlo por la pantalla. El doble click se genera pulsando tres clicks, y el doble click sirve para arrastrar objetos. Tiene más "gestos", algunos de los descubiertos por ahora, arrastrar con dos dedos hacia abajo para navegar por documentos y la web.

## Instrucciones creación USB live: 

Descargamos este repositorio en un sistema linux.

    sudo apt-get update -y
    
    sudo apt-get install git -y

    git clone https://github.com/aosucas499/lamentablet-vexia

 1. Con gparted se debe de crear una partición de 512mb, formateada en fat32 y con las banderas "boot" y "esp"en el usb. (sdX1, siendo la x=b, c, d...)

2. Crear una segunda partición cubriendo el total que quede del usb. Formateada en "ext4" (sdX2, siendo la x= la misma que en paso anterior)

3. Como ya conocemos si el usb es "sdb", sdc", etc...; vamos a una terminal y montamos la primera partición "sdX1" en una carpeta del sistema para poder copiar los archivos de este repositorio en esa partición.

    ```bash
    mkdir efiusb 
 
    sudo mount -t vfat /dev/sdX1 efiusb (X= b, c, d. Dependiendo de cómo se montó el usb. Usar df o gparted para saberlo)
    
    cd lamentablet-vexia/boot/boot-usb-live
    
    sudo cp -r * /home/$USER/efiusb
    
    sudo umount /dev/sdX1 (X= b, c, d. Dependiendo de cómo se montó el usb. Usar df o gparted para saberlo)
    
    cd /home/$USER
    
    sudo rm -r efiusb 
    
    sudo rm -r lamentablet-vexia
    
    
4. Instalamos una versión Live de linux en el usb, en este caso se ha probado ["minino-tde"](https://github.com/aosucas499/minino-TDE), en la segunda partición, la formateada en "ext4" (sdX2). Tenemos que utilizar unetbootin, ya que deja seleccionar en qué partición se instala y no borraríamos la primera partición de arranque UEFI. No funciona si grabas la ISO con rufus, etcher o cualquier otro. También hay que decir que la primera partición está configurada con un grub que solo funcionaría de esta manera.

    ```bash
    cd /home/$USER
    
    wget https://github.com/unetbootin/unetbootin/releases/download/700/unetbootin-linux64-700.bin
    
    chmod +x unetbootin-linux64-700.bin
    
    sudo ./unetbootin-linux64-700.bin
    
  Al abrirse el programa unetbootin, seleccionamos la distribución live, seleccionando en "disco imagen" y en los tres puntitos ... para seleccionar la .ISO.
  
  En el apartado "unidad", seleccionamos la segunda partición, formateada en "ext4" y que anteriormente sería "sdX2". Para terminar pulsamos en "instalar".
 
 # Instrucciones de instalación de MININO #TDE en disco
 
 1. Introducimos el usb en la tablet, mejor en la parte izquierda, por OTG y arrancamos la tablet pulsando la teca "ESC" hasta que accede a la Bios. Buscamos la pestaña BOOT y buscamos el arranque "UEFI USB"
    Sino aparece, cosa que ocurre con la tablet de cargador negro de 9v, pulsamos en UEFI: Built-in EFI shell y seguimos las siguiente indicaciones:
    
    a) Esperamos que terminen los 5 segundos y en la línea de comandos introducimos fs1: y pulsamos Enter. Para introducir los dos puntos tenemos que usar la tecla Mayúscula Derecha y la tecla ñ, ya que aparece como si fuese un teclado inglés.
    
    b) Introducimos en la terminal: cd EFI y pulsamos la tecla ENTER.
    
    c) Introducimos en la terminal: cd BOOT y pulsamos la tecla ENTER.
    
    d)Introducimos en la terminal: bootia32.efi (si la tablet tiene el cargador negro de 9v) y pulsamos la tecla ENTER o BOOTx64.efi (si la tablet tiene el cargador blanco de 5v)
    
    e) Pulsamos en la opción "minino live" y arrancará el sistema live.
    
2. Minino install:

Primeramente tendremos que preparar el disco y las particiones. Abrimos la aplicación Gparted y todas borramos las particiones de la tablet salvo la primera y la última. Vamos a formatearlas y cambiarle alguna etiqueta, de la siguiente manera:

    + Partición mmcblk0p1 --- formatear en fat 32 con nombre de etiqueta "EFI" y le asignamos solamente las banderas o flags "boot y "esp". 
    
    + Borramos todas las particiones restantes, de la 2 a la 13 
    
    + Partición mmcblk0p14 (en tablet cargador negro 9v) ----- formateamos en fat "ext4" con nombre de etiqueta "MININO" y no tocamos banderas/flags
    AVISO: Podemos redimensionar esta partición para aprovechar el tamaño de las particiones borradas, pero nunca borrarla y crearla otra vez ya que el arranque       uefi que genera el kernel de guadalinex busca una partición mmcblk014. 
    
Posteriormente buscamos la aplicación "instalador de minino" y seleccionamos una instalación desatendida y manual que haga la instalación del sistema a la partición "mmcblk0p14-MININO"

3. Instalamos el cargador de arranque UEFI REfind que nos facilitará la vida. Si podemos tener internet con un cable usb y el teléfono móvil creando un punto de acceso, ejecutaremos la primera línea, en caso contrario tendremos que descargar este repositorio en un pendrive y posteriormente agregarlo a la tablet por usb en la carpeta usuario  y empezaremos en la segunda línea.
    
     ```bash
     
     git clone https://github.com/aosucas499/lamentablet-vexia
     
     cd lamentablet-vexia/boot
     
     unzip refin*.zip
     
     rm *.zip
     
     cd refin*
     
     sudo ./refind-install --usedefault /dev/mmcblk0p1 --alldrivers
     
Pulsamos la tecla "Y", después la tecla intro y se habŕa quedado instalado en la partición de arranque para arrancar minino. 

4. Configuramos un poco la imagen y arranque de Refind.

        mkdir mmcblk0p1 
 
        sudo mount -t vfat /dev/mmcblk0p1 mmcblk0p1
        
        cd ..
        
        sudo cp splash.png refind*/mmcblk0p1/EFI/BOOT
    
        sudo sed -i "s/timeout 20/timeout 8/g" refind*/mmcblk0p1/EFI/BOOT/refind.conf
        
        sudo sed -i "s/#banner mybanner.png/banner splash.png/g" refind*/mmcblk0p1/EFI/BOOT/refind.conf

        sudo umount /dev/mmcblk0p1
        
        cd /home/$USER
  
        sudo rm -r lamentablet-vexia
        
        
        
Reinciamos el equipo y entramos en la bios, pulsando la tecla "ESC" hasta que accede a ella. Buscamos la pestaña "Boot option priorities" y ponemos como primera opción "UEFI OS".

Reiniciamos y entrará en minino que aún no dispone de drivers para la tablet, por lo que en el siguiente paso actualizaremos el kernel usado en guadalinex.


## Instalar kernel de Guadalinex Edu para que funcionen los drivers de wifi, sonido, táctil...

Mencionado al principio el problema por el que se toma esta decisión, los pasos son instalar un script que se encarga de instalar el kernel de guadalinex, adaptar el arranque UEFI de Minino para que lo acepte el Bootloader REFIND, reparar el problema de la cámara web y el táctil de la pantalla.

Si podemos tener internet con un cable usb y el teléfono móvil creando un punto de acceso ejecutaremos la primera línea, en caso contrario tendremos que descargar este repositorio en un pendrive y posteriormente agregarlo a la tablet por usb en la carpeta usuario y empezaremos en la segunda línea.

    git clone https://github.com/aosucas499/lamentablet-vexia
       
    cd lamentablet-vexia
    
    chmod +x install-minino-config.sh
    
    ./install-minino-config.sh
    
Reiniciamos y tendremos la tablet con los drivers necesarios para su funcionamiento.

## Pequeños arreglos para el uso en tablet

1. Añadimos a la barra de aplicaciones inferior el teclado virtual florence por si se usa en modo tablet y no tenemos teclado.

2. Añadimos el autologin con la aplicación "customize-minino" por si arrancamos en modo tablet y no podemos introducir el login.


Ya solo nos queda crear una imagen con CLONEZILLA como se hizo con guadalinex para distribuir.

## Creación usb-live clonezilla soporte UEFI

1. Arrancamos con el usb-live de minino del punto anterior "Instrucciones creación USB live" y arrancamos como para una instalación pero sin instalar.

2. Necesitamos tener internet con un cable usb y el teléfono móvil creando un punto de acceso. Al tener conexión introducimos:

        sudo apt-get update
        
        sudo apt-get install clonezilla -y
        
        sudo umount /dev/mmcblk0p14
        
        sudo clonezilla
        
3. Creamos una copia del disco de la tablet. Necesitaremos un pendrive de más de 32gb que introduciremos en el lateral del teclado y que será el repositorio de imágenes de clonezilla. La imagen local / sources será mmcblk0.

# lamentablet-vexia cargador 5v

Primeras investigaciones.

La tablet Vexia solo permite arrancar en modo UEFI, nada de modo Legacy. La BIOS es UEFI 64 bits. 

Al arrancar en modo uefi,hay que usar el archivo BOOTx64.efi en las de 5v, colocados en una carpeta llamada EFI, y dentro de esta otra llamada BOOT. Al tener arranque con 64bits permite arrancar cualquier sistema de hoy día con soporte UEFI a diferencia del modelo de cargador negro de 9v.

Los drivers son casi imposibles de compilar o encontrar hoy día, tras muchas pruebas para encontrar los drivers del wifi, sonido o táctil se opta por usar el kernel que se compiló en su día con las tablets vexia con el sistema Guadalinex Edu. En las tablet de cargador blanco de 5v el 3.10.20. Por ahora es imposible cargar el sistema añadiendo el kernel como se hizo con el otro modelo, por lo que se intenta utilizar otro kernel y añadir los drivers.

El driver táctil no es un driver del kernel, es un driver del XORG gráfico, viene configurado en el kernel pero necesita un paquete que incluye la configuración y el driver en sí, alojado en los repositorios de guadalinex EDU. Si instalas el paquete tal cual, el sistema no arranca, pues ese driver fue compilado para un XORG anterior. Es imposible volver a un XORG anterior en minino, por lo tanto se ha optado por compilar el driver MULTITOUCH para el XORG de minino (también se puede instalar con un apt-get install xserver-xorg-video-multitouch). El único problema es que esta acción hace que el táctil funcione parecido a un touchpad, no aparece el cursor donde pulsas en la pantalla taćtil, tienes que arrastrarlo por la pantalla. El doble click se genera pulsando tres clicks, y el doble click sirve para arrastrar objetos. Tiene más "gestos", algunos de los descubiertos por ahora, arrastrar con dos dedos hacia abajo para navegar por documentos y la web.

## Instrucciones creación USB live: 

Descargamos este repositorio en un sistema linux.

    sudo apt-get update -y
    
    sudo apt-get install git -y

    git clone https://github.com/aosucas499/lamentablet-vexia

 1. Con gparted se debe de crear una partición de 512mb, formateada en fat32 y con las banderas "boot" y "esp"en el usb. (sdX1, siendo la x=b, c, d...)

2. Crear una segunda partición cubriendo el total que quede del usb. Formateada en "ext4" (sdX2, siendo la x= la misma que en paso anterior)

3. Como ya conocemos si el usb es "sdb", sdc", etc...; vamos a una terminal y montamos la primera partición "sdX1" en una carpeta del sistema para poder copiar los archivos de este repositorio en esa partición.

    ```bash
    mkdir efiusb 
 
    sudo mount -t vfat /dev/sdX1 efiusb (X= b, c, d. Dependiendo de cómo se montó el usb. Usar df o gparted para saberlo)
    
    cd lamentablet-vexia/boot/boot-usb-live
    
    sudo cp -r * /home/$USER/efiusb
    
    sudo umount /dev/sdX1 (X= b, c, d. Dependiendo de cómo se montó el usb. Usar df o gparted para saberlo)
    
    cd /home/$USER
    
    sudo rm -r efiusb 
    
    sudo rm -r lamentablet-vexia
    
    
4. Instalamos una versión Live de linux en el usb, en este caso se ha probado ["minino-tde"](https://github.com/aosucas499/minino-TDE), en la segunda partición, la formateada en "ext4" (sdX2). Tenemos que utilizar unetbootin, ya que deja seleccionar en qué partición se instala y no borraríamos la primera partición de arranque UEFI. No funciona si grabas la ISO con rufus, etcher o cualquier otro. También hay que decir que la primera partición está configurada con un grub que solo funcionaría de esta manera.

    ```bash
    cd /home/$USER
    
    wget https://github.com/unetbootin/unetbootin/releases/download/700/unetbootin-linux64-700.bin
    
    chmod +x unetbootin-linux64-700.bin
    
    sudo ./unetbootin-linux64-700.bin
    
  Al abrirse el programa unetbootin, seleccionamos la distribución live, seleccionando en "disco imagen" y en los tres puntitos ... para seleccionar la .ISO.
  
  En el apartado "unidad", seleccionamos la segunda partición, formateada en "ext4" y que anteriormente sería "sdX2". Para terminar pulsamos en "instalar".
  
   # Instrucciones de instalación de MININO #TDE en disco
 
 1. Introducimos el usb en la tablet, mejor en la parte izquierda, por OTG y arrancamos la tablet. Normalmente arranca automáticamente, pues esta tablet si tiene BIOS de 64bits, en caso de que no arranque el usb,  pulsamos la teca "ESC" hasta que accede a la Bios. Buscamos la pestaña BOOT y buscamos el arranque "UEFI USB"
   
2. Minino install:

Primeramente tendremos que preparar el disco y las particiones. Abrimos la aplicación Gparted y todas borramos las particiones de la tablet salvo la primera y la última. Vamos a formatearlas y cambiarle alguna etiqueta, de la siguiente manera:

    + Partición mmcblk0p1 --- formatear en fat 32 con nombre de etiqueta "EFI" y le asignamos solamente las banderas o flags "boot y "esp". 
    
    + Borramos todas las particiones restantes, de la 2 a la 15
    
    + Partición mmcblk0p16 (en tablet cargador blanco 5v) ----- formateamos en fat "ext4" con nombre de etiqueta "MININO" y no tocamos banderas/flags
    AVISO: Podemos redimensionar esta partición para aprovechar el tamaño de las particiones borradas, pero nunca borrarla y crearla otra vez ya que el arranque       uefi que genera el kernel de guadalinex busca una partición mmcblk016. 
    
Posteriormente buscamos la aplicación "instalador de minino" y seleccionamos una instalación desatendida y manual que haga la instalación del sistema a la partición "mmcblk0p16-MININO"

### Excepción en el arranque en la tablet de cargador blanco 5v
Tras instalar minino, el cargador refind no reconoce el grub de minino en esta tablet por lo que arrancamos con el live de minino y cuando cargue el grub, pulsamos la tecla ESC, a continuación instroducimos los siguientes comandos para que arranque el grub instalado y podamos acceder y cambiar el kernel y el boot uefi en minino:

        set root=(hd1,gpt16)

        linux /vmlinuz root=/dev/mmcblk0p16 i915.modeset=1

        initrd /initrd.img

        boot
        
### Instalar gestor de arranque compatible con la tablet de cargador blanco 5v

En el punto anterior hemos apuntado como arrancar ya que esta tablet posee solamente arranque uefi de 64 bits y no detecta de primeras el sistema MININO de 32 bits, por lo tanto, para hacerlo permanente arrancaremos un sistema de 64 bits y actualizaremos el arranque en modo uefi (cosa que minino versión TDE 32 bits no puede hacer).
        
Podemos utilizar el usb que construimos anteriormente en el punto  "Instrucciones creación USB live", pero en el punto 4, usando unetbootin, en lugar de instalar MININO TDE, instalaremos en la segunda partición, MININO 64bits, el enlace está [aquí](https://minino.galpon.org/ISO/minino-queiles-64.iso).
Por supuesto también valdría, cualquier distribución actual como Ubuntu, Linux Mint...pero su versión de 64bits, cosa que no necesitaría de hacer uso de las instrucciones del punto "Instrucciones creación USB live", simplemente, grabar en un USB con la herramienta "etcher".

Arrancamos el USB en modo live y cuando accedamos al sistema, (conectamos el teléfono móvil, creando un puto de acceso por usb si el sistema no detecta la wifi) y abrimos una terminal e introducimos los siguientes comandos:

    apt-get update
    
    sudo mount --bind /dev /media/minino/MININO/dev
    
    sudo mount --bind /proc /media/minino/MININO/proc
    
    sudo mount --bind /sys /media/minino/MININO/sys
    
    sudo chroot /media/minino/MININO
    
    nano /etc/resolv.conf
    
    (cambiamos la dirección IP por 8.8.8.8, y guardamos pulsando Ctrl y la tecla o, para salir Ctrl y la tecla x)
    
    mount /dev/mmcblk0p1 /boot/efi
    
    rm -r /boot/efi/*
    
    apt-get update -y

    apt-get install grub-efi-amd64 -y
    
    mount -t efivarfs efivarfs /sys/firmware/efi/efivars
    
    grub-install /dev/mmcblk0p1
    
    update-grub
    
    update-grub2
    
    exit
    
    sudo umount /media/minino/MININO/dev
    
    sudo umount /media/minino/MININO/proc
    
    sudo umount /media/minino/MININO/sys
    
    sudo umount /media/minino/MININO/
    
    sudo reboot


## Actualizar kernel 3.10.20_edu de Guadalinex (debería de funcionar todo, pero los módulos iban firmados al generar el kernel y da fallo al cargar el módulo de wifi y cámara.) 

    git clone https://github.com/aosucas499/lamentablet-vexia
       
    cd lamentablet-vexia
    
    chmod +x install-minino-config-5v-white.sh
    
    ./install-minino-config-5v-white.sh 
    
    sudo reboot
   

## Actualizar kernel 4.9 (para investigar, es posible desarrolar el módulo del wifi)

    sudo apt-get update -y
    
    sudo apt-get install i2c-tools hwinfo lshw pciutils usbutils evtest crda xinput pavucontrol -y
    
    sudo apt-get -y install linux-image-4.9.0-0.bpo.12-686-pae linux-headers-4.9.0-0.bpo.12-686-pae firmware-intel-sound
    
    sudo update-grub2
    
    sudo reboot
    
 *Módulo wifi (si acabamos de hacer los pasos anteriores, reiniciamos y accedemos a el kernel 4.9)
 
    git clone https://github.com/314942468GitHub/rtl8723bs
    
    cd rtl8723bs
    
    sudo apt-get install autoconf make automake gcc -y
    
    sudo make
    
    sudo make install
    
    sudo modprobe r8723bs
    
    cd /home/$USER/
    
    sudo rm -r rtl8723bs
    
    
## Actualizar kernel 5.10 (compatible con batería, wifi y driver de sonido, aunque no suena)

Cambiamos el repositorio de jessie por el de buster DEBIAN.

    sudo apt-get update -y
    
    sudo apt-get install i2c-tools hwinfo lshw pciutils usbutils evtest crda xinput pavucontrol -y

    sudo mv /etc/apt/sources.list /etc/apt/sources.list-jessie
    
    sudo echo "deb http://deb.debian.org/debian buster-backports main contrib non-free" > sources.list

    sudo mv sources.list /etc/apt/

    sudo apt-get update -y
    
    sudo apt-get install -y firmware-linux firmware-realtek firmware-intel-sound linux-image-5.10.0-0.bpo.3-686 
    
    sudo update-grub2
    
    sudo mv /etc/apt/sources.list /etc/apt/sources.list-buster
    
    sudo mv /etc/apt/sources.list-jessie /etc/apt/sources.list

    sudo apt-get update -y
    
 *Arreglo del sonido supuestamente

    cd /home/$USER
    
    echo "blacklist snd_hdmi_lpe_audio" > blacklist_snd_hdmi_lpe_audio.conf
    
    sudo mv blacklist_snd_hdmi_lpe_audio.conf /etc/modprobe.d
    
    git clone https://github.com/plbossart/UCM
    
    cd UCM
    
    sudo cp -rf bytcr-rt5640 /usr/share/alsa/ucm
    
    cd ..
    
    sudo rm -r UCM
    
    sudo reboot
    
   
## Reconstruir el kernel (por ahora la única solución posible para obtener todos los módulos funcionando, generar un kernel que arranque y tenga activados todos los módulos)

Descargarmos el kernel que queramos de aquí en format .gz [https://mirrors.edge.kernel.org/pub/linux/kernel/](https://mirrors.edge.kernel.org/pub/linux/kernel/) y ejecutamos:

    sudo apt-get install git build-essential kernel-package fakeroot libncurses5-dev libssl-dev ccache bison flex

    cd Descargas

    tar zxvf linux*

    cd linux*

    cp /boot/config-$(uname -r) .config

    make oldconfig

    make menuconfig (elegir las opciones o módulos nuevos a incluir)

    make clean

    make -j 4 deb-pkg LOCALVERSION=-dre
    
    cd ..
    
    sudo dpkg -i linux-libc*
    
    sudo dpkg -i linux-firmware*
    
    sudo dpkg -i linux-headers*
    
    sudo dpkg -i linux-image*
    
    
 

    
    
    
  
  
