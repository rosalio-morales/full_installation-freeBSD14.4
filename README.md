# full_installation-freeBSD14.4 ===> Interfaz grafica (GUI)

# Crear un script de shell (full_installation-freeBSD14.4.sh).
# Copiar y pegar el codigo de abajo tal cual ==> Nota: ingresar nuestro nombre de usuario en $USER_NAME='Aqui dentro' en el numero # 6, si no, no te va a arrancar el escritorio (GUI).
# Darle los permisos para ejecutarse (sudo o doas chmod +x full_installation-freeBSD14.4.sh).
# Ejecutar el script (sudo o doas ./full_installation-freeBSD14.4.sh).


# full_installation-freeBSD14.4 ===> Graphical User Interface (GUI)

# Make a shell script  (full_installation-freeBSD14.4.sh).
# Copy && paste code below ==> Warning: write our user into  $USER_NAME='your user here' at topic # 6, if not, Graphical User Interface won't work (GUI).
# Provide user permissions to script (sudo or doas chmod +x full_installation-freeBSD14.4.sh).
# Execute script (sudo o doas ./full_installation-freeBSD14.4.sh).



#!/bin/sh

# 1. Actualizar repositorios e instalar paquetes necesarios.
pkg update
pkg install -y xorg xfce lightdm lightdm-gtk-greeter drm-61-kmod dsbmd dsbmc bluedevil \
android-file-transfer kdeconnect-kde dbus pipewire wireplumber \
libva-intel-driver wine-proton winetricks \
firefox caffeine

# 1.1 Crear directorio de autoinicio para plumber (bluetooth para auriculares inalambricos).
mkdir -p ~/.config/autostart

# 1.2 Crear archivos de inicio para pipewire y wireplumber.
ln -s /usr/local/share/applications/pipewire.desktop ~/.config/autostart/
ln -s /usr/local/share/applications/pipewire-pulse.desktop ~/.config/autostart/
ln -s /usr/local/share/applications/wireplumber.desktop ~/.config/autostart/

# 1.3 Configuracion para que el sonido no "chasquee" o tenga lag
echo " # Mejora el tiempo de respuesta del audio \ hw.usb.uaudio.buffer_ms=2" | tee > /boot/loader.conf;

# 2. Configurar el cargador de modulos (kernel) ==> Nota: i915kms se carga mejor via rc.conf en versiones recientes.
sysrc kld_list+="i915kms"
sysrc kld_list+="linux64"

# 3. Configurar /etc/rc.conf para servicios esenciales.
sysrc dbus_enable="YES"
sysrc lightdm_enable="YES"
sysrc lightdm-gtk-greeter_enable="YES"
sysrc dsbmd_enable="YES"
sysrc autounmountd_enable=YES"
sysrc linux_enable=YES"

# 4. Configurar Bluetooth para mi Hono x7a
sysrc hcsecd_enable="YES"
sysrc sdpd_enable="YES"
sysrc bluetooth_enable="YES"

# 5. Configurar puntos de montaje necesarios en /etc/fstab
echo "proc          /proc          procfs     rw      0    0" >> /etc/fstab
echo "fdescfs       /dev/fd        fdescfs    rw      0    0" >> /etc/fstab

# 6. Configurar reglas de dispositivos (permisos USB/Video para mi usuario) ==> Poner nuestro nombre de usuario dentro de USER_NAME=" ".
USER_NAME="Aqui pon tu nombre de usuario"

cat <<EOF > /etc/devfs.rules
[localrules=10]
add path 'da*' mode 0660 group operator
add path 'usb/*' mode 0660 group operator
add path 'usbctl' mode 0660 group operator
add path 'ugen*' mode 0660 group operator
add path 'video*' mode 0660 group video
add path 'drm*' mode 0660 group video
EOF

sysrc devfs_system_ruleset="localrules"

# 7. Agregar usuario a grupos criticos
pw groupmod wheel -m $USER_NAME
pw groupmod operator -m $USER_NAME
pw groupmod video -m $USER_NAME

# 8. Actualizar sistema.
pkg update && pkg upgrade && pkg autoremove

echo " ------------------------------------------------------------ "
echo "Cofiguracion completada. REINICIAR para iniciar GUI.           "
echo "Al entrar, abre 'KDE Connect' para vincular el Honor x7a.     "
echo " ------------------------------------------------------------ " 
