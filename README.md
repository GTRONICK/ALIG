# INSTALACIÓN DE ARCHLINUX CON SOPORTE UEFI #

Ingrese a https://gtronick.github.io/ALIG/ para ver la versión web.

## **GTRONICK** ##  
Jaime Quiroga 
 
Editado por última vez: **05/04/2018**

El presente documento no pretende ser una guía completa para la instalación de ArchLinux. Es una guía rápida para acelerar el proceso de instalación. Para más detalles, consultar la **Wiki** de ArchLinux, y su guía de instalación.


1. Configurar la BIOS de tu equipo para permitir el arranque desde un dispositivo USB, y el arranque EFI. Si la instalación se está haciendo en VirtualBox, configurar la máquina virtual para permitir el arranque con EFI. Seleccionar la máquina virtual, propiedades, System, Enable EFI.

2. Iniciar la máquina y seleccionar el disco de instalación

3. Seleccionar:

        Arch Linux Arch ISO x86_64 UEFI USB

4. Una vez que ha iniciado, entrar a la wiki (Opcional)

5. Para verificar que estamos en modo UEFI, ejecutar el siguiente comando: 

        ls /sys/firmware/efi/efivars

    *Si se muestra contenido en la carpeta efivars, quiere decir que arrancamos el sistema correctamente en modo UEFI.*


6. Verificar conexión a internet haciendo ping a: archlinux.org (o cualquier otra página o IP)

        ping archlinux.org

7. En caso de tener sólo wifi, usar:


        ip link (Para listar las interfaces. Ubicar la de Wifi, generalmente es wlp2s0)
        wifi-menu -o wlp2s0

    *Seleccionar la red, e ingresar contraseña.*

8. Actualizar el reloj del sistema: 

        timedatectl set-ntp true

9. Verificar con: (opcional)

        timedatectl status

10. Identificar los discos con: 

        lsblk

11. Crear una nueva tabla de particiones GPT en /dev/sda con:

        gdisk /dev/sda

        w (Para escribir los cambios)
        Y (Para aceptar los cambios)

12. Verificar nuevamente con: 

        gdisk /dev/sda

    *Se debe listar "GPT Present" al final de la lista.*

13. Crear partición /boot con:

        n (Crea una nueva partición)
        (Dejar número de la partición por defecto, presionando ENTER)
        (Dejar por defecto el sector inicial, presionando ENTER)
        (Para el sector final, escribir +512M y presionar ENTER)
        (Escribir EF00 cuando se pida código de partición y luego ENTER)
        w (Para escribir los cambios y luego ENTER)
        y (Para aceptar los cambios y luego ENTER)

14. Crear particion swap con:

        gdisk /dev/sda
        n
        ENTER
        ENTER
        +2G
        8200
        W
        Y
        
15. Crear particion / con:

        gdisk /dev/sda
        n
        ENTER
        ENTER
        +10G
        8304
        W
        Y

16. Crear partición /home con:

        gdisk /dev/sda
        n
        ENTER
        ENTER
        ENTER
        8302
        W
        Y

17. Verificar con:

        lsblk

18. Formatear partición /boot con:

        mkfs.fat -F32 /dev/sda1

19. Formatear particion swap con:

        mkswap /dev/sda2

20. Activar swap con:

        swapon /dev/sda2

21. Formatear particion / con:

        mkfs.ext4 /dev/sda3

22. Formatear partición /home con:

        mkfs.ext4 /dev/sda4

23. Montar particion / en /mnt con:
        
        mount /dev/sda3 /mnt

24. Crear directorio para /boot con:

        mkdir -p /mnt/boot

25. Montar partición /boot con:

        mount /dev/sda1 /mnt/boot

26. Crear directorio para /home con:

        mkdir -p /mnt/home

27. Montar partición /home con:

        mount /dev/sda4 /mnt/home

28. Instalar los paquetes base:

        pacstrap /mnt

    *Esto iniciará la instalación de los paquetes base (191.35 MiB aprox.)*

29. Generar fstab con:

        genfstab -U /mnt >> /mnt/etc/fstab

30. Verificar con:

        cat /mnt/etc/fstab

31. Iniciar sesión como root en la instalación con:

        arch-chroot /mnt /bin/bash

32. Generar locales:

        nano /etc/locale.gen

    *Descomentar las líneas de interés quitando el símbolo #, en este caso:*

        en_US.UTF-8 UTF-8

    *Guardar presionando Ctrl + X, luego Y y finalmente ENTER*
        
33. Construir el soporte de idioma con: 

        locale-gen

34. Crear el archivo de configuración correspondiente con:

        nano /etc/locale.conf

    *Agregar el siguiente contenido:*

      LANG=en_US.UTF-8

    *Guardar presionando Ctrl + X, luego Y y finalmente ENTER*

35. Ajustar zona horaria:

        tzselect
        2 
        ENTER
        14 (Número correspondiente a la zona)
        ENTER
        1 (Número correspondiente a la subzona)
        ENTER

36. Crear el link simbólico para hacer el cambio permanente:

        ln -s /usr/share/zoneinfo/<ZONA>/<SUB_ZONA> /etc/localtime

    *donde < ZONA > puede ser America y < SUB_ZONA > puede ser Bogota.*

-----------------------------------------------------------------------------------------------------------------------------
## INSTALACIÓN DEL BOOTLOADER, USAR SOLO UNO DE LOS DOS MOSTRADOS A CONTINUACIÓN: ##
### SYSTEMD-BOOT (Recomendado): ###

37. Instalar systemd-boot con:

        bootctl --path=/boot install

38. Generar archivo de configuración de systemd-boot con:
        
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

40. Abrir el archivo generado con:

        nano /boot/loader/entries/arch.conf

    Se debe agregar lo siguiente, de manera que el serial generado, quede después de PARTUUID y antes de rw, como sigue:

        title ArchLinux
        linux /vmlinuz-linux
        initrd /initramfs-linux.img
        options root=PARTUUID=14420948-2cea-4de7-b042-40f67c618660 rw

    *Guardar presionando Ctrl + X, luego Y y finalmente ENTER*

## GRUB:

41. Instalar GRUB con:
        
        grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=grub

    *Si se reporta un error, que indica que /boot no parece ser una partición EFI, verificar que esté correctamente montada en /mnt/boot. Para hacer esto, escribir exit para acceder a la consola del live system. Luego, ejecutar:*
        
        mkdir -p /mnt/boot
        mount /dev/sda1 /mnt/boot
        arch-chroot /mnt/ /bin/bash

    *Repetir el comando de instalación grub-install....*

42. Generar archivo de configuración de grub con:
        
        grub-mkconfig -o /boot/grub/grub.cfg

-----------------------------------------------------------------------------------------------------------------------------

43. Configuración de red:

    *Agregar el nombre del host a /etc/hostname, por ejemplo con:*

        echo gtronick > /etc/hostname

44. Agregar el hostname a /etc/hosts, donde <myHostName> es el nombre de host escogido e ingresado en /etc/hostname, por ejemplo:
        
        127.0.0.1        localhost.localdomain        localhost
        ::1              localhost.localdomain        localhost
        127.0.1.1        gtronick.localdomain	      gtronick

45. Instalar paquetes para el controlador WiFi:

        pacman -S iw wpa_supplicant dialog

46. Ajustar contraseña para  root, con:

        passwd

    *Ingresar nueva contraseña*   
    *Repetir la contraseña*


47. Salir de la sesión, desmontar particiones:

        exit
        umount -R /mnt
        umount -R /mnt/boot #si existe o aún está montado

48. Antes de reiniciar, verificar que se hayan desmontado todas las particiones de /dev/sda, con

        lsblk

49. Por último reiniciar con:

        reboot

