# INSTALACIÓN DE ARCHLINUX CON SOPORTE UEFI #
Ingrese a https://gtronick.github.io/ALIG/ para ver la versión web.

----
*Use el comando:*

    elinks https://gtronick.github.io/ALIG/
    
*Para acceder a esta guía desde el Live system de ArchLinux.*

----
Sitio web de **GTRONICK**: [http://gtronick.com](http://gtronick.com)    
Autor: Jaime Quiroga  
Editado por última vez: **07/05/2020 10:15 AM**

El presente documento no pretende ser una guía completa para la instalación de ArchLinux. Es una guía rápida para acelerar el proceso de instalación. Para más detalles, consultar la [**Wiki**](https://wiki.archlinux.org/index.php/Installation_guide) de ArchLinux, y su guía de instalación.


1. Configurar la BIOS de tu equipo para permitir el arranque desde un dispositivo USB, y el arranque EFI. Si la instalación se está haciendo en VirtualBox, configurar la máquina virtual para permitir el arranque con EFI. Seleccionar la máquina virtual, propiedades, System, Enable EFI.

2. Iniciar la máquina y seleccionar el disco de instalación

3. Seleccionar:

        Arch Linux Arch ISO x86_64 UEFI USB

4. Cuando termine de iniciar, cargar la distribución de teclado correspondiente. Por defecto, la distribución es US (Inglés). Para listar las distribuciones de teclado disponibles usar:

        ls /usr/share/kbd/keymaps/**/*.map.gz
        
   *Si se desea cargar la distribución para un teclado en español por ejemplo, usar:*
   
        loadkeys es
        
5. Para verificar que estamos en modo UEFI, ejecutar el siguiente comando: 

        ls /sys/firmware/efi/efivars

    *Si se muestra contenido en la carpeta efivars, quiere decir que arrancamos el sistema correctamente en modo UEFI.*


6. Verificar la conexión a Internet haciendo ping a: archlinux.org (o cualquier otra página o IP)

        ping archlinux.org

7. En caso de tener sólo wifi, usar:

        wifi-menu

    *Seleccionar la red, e ingresar contraseña.*

8. Activar la sincronización del reloj del sistema con Internet: 

        timedatectl set-ntp true

9. Verificar: (opcional)

        timedatectl status

10. Identificar los discos: 

        lsblk

11. Crear una nueva tabla de particiones GPT en /dev/sda:

        gdisk /dev/sda

        w (Para escribir los cambios)
        Y (Para aceptar los cambios)

12. Verificar nuevamente: 

        gdisk /dev/sda

    *Se debe listar "GPT Present" al final de la lista.*

13. Crear partición /boot :

        n (Crea una nueva partición)
        (Dejar número de la partición por defecto, presionando ENTER)
        (Dejar por defecto el sector inicial, presionando ENTER)
        (Para el sector final, escribir +512M y presionar ENTER)
        (Escribir EF00 cuando se pida código de partición y luego ENTER)
        w (Para escribir los cambios y luego ENTER)
        y (Para aceptar los cambios y luego ENTER)

14. Crear particion swap :

        gdisk /dev/sda
        n
        ENTER
        ENTER
        +2G
        8200
        W
        Y
        
15. Crear particion / :

        gdisk /dev/sda
        n
        ENTER
        ENTER
        +10G
        8304
        W
        Y

16. Crear partición /home :

        gdisk /dev/sda
        n
        ENTER
        ENTER
        ENTER
        8302
        W
        Y

17. Verificar:

        lsblk

18. Formatear partición /boot :

        mkfs.fat -F32 /dev/sda1

19. Formatear particion swap :

        mkswap /dev/sda2

20. Activar swap :

        swapon /dev/sda2

21. Formatear particion / :

        mkfs.ext4 /dev/sda3

22. Formatear partición /home :

        mkfs.ext4 /dev/sda4

23. Montar particion / en /mnt :
        
        mount /dev/sda3 /mnt

24. Crear directorio para /boot :

        mkdir -p /mnt/boot

25. Montar partición /boot :

        mount /dev/sda1 /mnt/boot

26. Crear directorio para /home :

        mkdir -p /mnt/home

27. Montar partición /home :

        mount /dev/sda4 /mnt/home

28. Instalar los paquetes base:

        pacstrap /mnt base base-devel nano linux linux-firmware

    *Esto iniciará la instalación de los paquetes base (250 MiB aprox.) si se tiene CPU Intel, es recomendable instalar intel-ucode*

29. Generar fstab:

        genfstab -U /mnt >> /mnt/etc/fstab

30. Verificar:

        cat /mnt/etc/fstab

31. Iniciar sesión como root en la instalación:

        arch-chroot /mnt /bin/bash

32. Generar locales:

        nano /etc/locale.gen

    *Descomentar las líneas de interés quitando el símbolo #, en este caso:*

        en_US.UTF-8 UTF-8

    *Guardar presionando Ctrl + X, luego Y y finalmente ENTER*
        
33. Construir el soporte de idioma: 

        locale-gen

34. Crear el archivo de configuración correspondiente:

        nano /etc/locale.conf

    *Agregar el siguiente contenido:*

      LANG=en_US.UTF-8

    *Guardar presionando Ctrl + X, luego Y y finalmente ENTER*

35. Ajustar zona horaria, en este ejemplo se usa America/Bogotá:

        tzselect
        2 
        ENTER
        14 (Número correspondiente a la zona)
        ENTER
        1 (Número correspondiente a la subzona)
        ENTER

36. Borrar el archivo de configuración anterior y crear el link simbólico para hacer el cambio permanente:

        rm /etc/localtime
        ln -s /usr/share/zoneinfo/<ZONA>/<SUB_ZONA> /etc/localtime

    *donde < ZONA > puede ser America y < SUB_ZONA > puede ser Bogota.*
    
37. Instalar **systemd-boot** (Sólo si no se va a usar GRUB, de lo contrario saltar al paso **40**):

        bootctl --path=/boot install

38. Generar archivo de configuración de systemd-boot:
        
        nano /boot/loader/loader.conf

    Agregar el siguiente contenido:

        default arch
        timeout 0
        editor 0

    *Guardar presionando Ctrl + X, luego Y y finalmente ENTER*

39. Generar el archivo de la entrada por defecto para systemd-boot:

        echo $(blkid -s PARTUUID -o value /dev/sda3) > /boot/loader/entries/arch.conf

    Esto generará un archivo de nombre arch.conf en la ruta especificada, con un contenido similar a:

        14420948-2cea-4de7-b042-40f67c618660

40. Abrir el archivo generado:

        nano /boot/loader/entries/arch.conf

    Se debe agregar lo siguiente, de manera que el serial generado, quede después de PARTUUID y antes de rw, como sigue:

        title ArchLinux
        linux /vmlinuz-linux
        initrd /intel-ucode.img 
        initrd /initramfs-linux.img
        options root=PARTUUID=14420948-2cea-4de7-b042-40f67c618660 rw

    **La línea: initrd /intel-ucode.img, sólo se debe poner cuando se ha instalado intel-ucode en el paso 28! **
    *Guardar presionando Ctrl + X, luego Y y finalmente ENTER*

41. Instalar **GRUB** (sólo si no instaló systemd-boot, de lo contrario saltar al paso **43**):
        
        grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=grub

    *Si se reporta un error, que indica que /boot no parece ser una partición EFI, verificar que esté correctamente montada en /mnt/boot. Para hacer esto, escribir exit para acceder a la consola del live system. Luego, ejecutar:*
        
        mkdir -p /mnt/boot
        mount /dev/sda1 /mnt/boot
        arch-chroot /mnt/ /bin/bash

    *Repetir el comando de instalación grub-install....*

42. Generar archivo de configuración de grub:
        
        grub-mkconfig -o /boot/grub/grub.cfg

43. Configuración de red:

    *Agregar el nombre del host a /etc/hostname, por ejemplo con:*

        echo gtronick > /etc/hostname

44. Agregar el hostname a /etc/hosts, por ejemplo:
        
        127.0.0.1        localhost.localdomain        localhost
        ::1              localhost.localdomain        localhost
        127.0.1.1        gtronick.localdomain	      gtronick

45. Instalar paquetes para el controlador WiFi y otros paquetes para la postinstalación:

        pacman -S iw wpa_supplicant dialog vi vim sudo elinks

46. Ajustar contraseña para  root:

        passwd

    *Ingresar nueva contraseña*   
    *Repetir la contraseña*


47. Salir de la sesión, desmontar particiones:

        exit
        umount -R /mnt
        umount -R /mnt/boot #si existe o aún está montado

48. Antes de reiniciar, verificar que se hayan desmontado todas las particiones de /dev/sda:

        lsblk

49. Reiniciar con:

        reboot

50. Después de reiniciar el equipo con ArchLinux instalado, crear un nuevo usuario, por ejemplo:

        useradd -m myUser
        
51. Asignar una contraseña al nuevo usuario creado:

        passwd myUser
        
52. Dar permisos de uso para Sudo al nuevo usuario:

        visudo
        
    *Buscar la línea  ROOT  ALL=(ALL) ALL y justo debajo de esta, agregar nuestro usuario, por ejemplo:*
        
        myUser   ALL=(ALL) ALL
    
    *Para editar el documento, presionar la tecla i. Después de esto ya podremos agregar texto normalmente.*
    *Para guardar los cambios, presionar ESC, luego escribir :wq y finalmente ENTER.*
    
53. Probar la conexión de red con:

        ping www.archlinux.org
        
54. Si sólo se dispone de WiFi, y se genera error al intentar conectar a la red con wifi-menu, verifique que la interfaz se encuentra abajo, puede usar:

        ip link set <interface> down
        
 *donde < interface > puede ser por ejemplo wlp2s0, lo que quedaría como: ip link set wlp2s0 down, después de esto, intente nuevamente conectarse con wifi-menu*

Finalmente, como sugerencia adicional, se recomienda instalar los siguientes paquetes cuando se haya reiniciado el sistema:

*>> Para montaje de discos NTFS y otros:*
gvfs 
gvfs-nfs 
ntfs-3g

*>> Para manejo de adb en dispositivos Android:*
adb

*>> Para clonado de repositorios y otros paquetes del AUR:*
git


Visita mi canal de YouTube para ver la instalación en video y otros tutoriales: [GTRONICK](https://www.youtube.com/user/GTRONICK)
