# Guía Completa de Configuración para Openbox en Debian 13

Este documento recopila todos los pasos, comandos y configuraciones necesarios para dejar Openbox completamente funcional con gestos de panel táctil, instalación de paquetes y gestión avanzada de ventanas mediante atajos de teclado.

---

## 1. Configuración del Touchpad (libinput)

Por defecto, Openbox no gestiona el touchpad, esto se debe hacer directamente en el servidor gráfico (X11) mediante `libinput`.

### Objetivo:
Habilitar "Tap-to-click" (tocar para hacer clic), clic con varios dedos y Desplazamiento Natural (invertir el scroll).

### Pasos:
1. Crear el directorio de configuración si no existe:
   ```bash
   sudo mkdir -p /etc/X11/xorg.conf.d

```

2. Crear/Editar el archivo de configuración:
```bash
sudo nano /etc/X11/xorg.conf.d/90-touchpad.conf

```


3. Añadir el siguiente contenido:
```text
Section "InputClass"
    Identifier "libinput touchpad catchall"
    MatchIsTouchpad "on"
    MatchDevicePath "/dev/input/event*"
    Driver "libinput"

    # Activar el tap-to-click
    Option "Tapping" "on"

    # Mapeo: 1 dedo = Izquierdo, 2 = Derecho, 3 = Central
    Option "TappingButtonMap" "lmr"

    # Desplazamiento Natural (invertir scroll arriba/abajo)
    Option "NaturalScrolling" "true"
EndSection

```


4. **Aplicar cambios:** Es necesario reiniciar la sesión gráfica o el ordenador:
```bash
sudo reboot

```



---

## 2. Instalación de Archivos `.deb`

Al no tener una tienda de software gráfica instalada por defecto, la terminal es la mejor opción.

### Método Recomendado (`apt`)

Este método es superior porque resuelve las dependencias automáticamente de los repositorios de Debian.

```bash
cd ~/Descargas
sudo apt install ./nombre_del_archivo.deb

```

*Problema común evitado:* Olvidar el `./` hace que `apt` busque en internet en lugar del archivo local. Siempre usa `./`.

### Método Clásico (`dpkg`)

Si se usa `dpkg`, es posible que falten dependencias.

```bash
cd ~/Descargas
sudo dpkg -i nombre_del_archivo.deb

```

*Problema de dependencias:* Si el comando anterior falla indicando que faltan dependencias, se soluciona forzando la instalación de lo faltante con:

```bash
sudo apt --fix-broken install

```

---

## 3. Atajos de Teclado y Tiling en Openbox

Toda la configuración del teclado y ratón de Openbox reside en el archivo `rc.xml`.

### Preparación del entorno

Primero, aseguramos que exista una copia local del archivo de configuración para no modificar la del sistema:

```bash
mkdir -p ~/.config/openbox
cp -n /etc/xdg/openbox/rc.xml ~/.config/openbox/

```

*(El flag `-n` previene sobreescribir el archivo si ya existía).*

Para editar el archivo:

```bash
nano ~/.config/openbox/rc.xml

```

*Todos los atajos deben ir dentro de la sección `<keyboard>`.*

### A. Atajos de Aplicaciones

*(La tecla `W` representa la tecla Windows / Super).*

```xml
    <keybind key="W-t">
      <action name="Execute">
        <command>terminator</command>
      </action>
    </keybind>

    <keybind key="W-f">
      <action name="Execute">
        <command>firefox</command>
      </action>
    </keybind>

    <keybind key="W-e">
      <action name="Execute">
        <command>thunar</command>
      </action>
    </keybind>

    <keybind key="W-o">
      <action name="Execute">
        <command>obsidian</command>
      </action>
    </keybind>

```

### B. Acomodo de Ventanas (Tiling a Mitades)

Replica el comportamiento de Windows + Flechas.

```xml
    <keybind key="W-Left">
      <action name="UnmaximizeFull"/>
      <action name="MoveResizeTo">
        <x>0</x><y>0</y><height>100%</height><width>50%</width>
      </action>
    </keybind>

    <keybind key="W-Right">
      <action name="UnmaximizeFull"/>
      <action name="MoveResizeTo">
        <x>-0</x><y>0</y><height>100%</height><width>50%</width>
      </action>
    </keybind>

    <keybind key="W-Up">
      <action name="Maximize"/>
    </keybind>

    <keybind key="W-Down">
      <action name="Unmaximize"/>
    </keybind>

```

### C. Acomodo de Ventanas (Cuadrantes)

Usando el bloque de teclas Q, W, A, S para las esquinas.

```xml
    <keybind key="W-q">
      <action name="UnmaximizeFull"/>
      <action name="MoveResizeTo">
        <x>0</x><y>0</y><height>50%</height><width>50%</width>
      </action>
    </keybind>

    <keybind key="W-w">
      <action name="UnmaximizeFull"/>
      <action name="MoveResizeTo">
        <x>-0</x><y>0</y><height>50%</height><width>50%</width>
      </action>
    </keybind>

    <keybind key="W-a">
      <action name="UnmaximizeFull"/>
      <action name="MoveResizeTo">
        <x>0</x><y>-0</y><height>50%</height><width>50%</width>
      </action>
    </keybind>

    <keybind key="W-s">
      <action name="UnmaximizeFull"/>
      <action name="MoveResizeTo">
        <x>-0</x><y>-0</y><height>50%</height><width>50%</width>
      </action>
    </keybind>

```

### Aplicar cambios de Openbox

A diferencia del Touchpad que requiere reinicio, Openbox puede recargar su configuración en vivo desde la terminal:

```bash
openbox --reconfigure

```
