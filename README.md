# Debian12LightConf
Configuración de Debian 12 para equipos de muy bajos recursos.

# Guía de Instalación Minimalista Linux 32-bits en este caso
**Hardware:** Intel Atom (32-bits) / 2GB RAM
**Objetivo:** Sistema base en consola pura con entorno gráfico a demanda (Openbox), gestión de red terminal-gráfica y VPN WireGuard.

---

## 1. Sistema Base (Debian 12 Bookworm)
* **Imagen ISO:** Descargar la versión `netinst` para arquitectura **i386** (32 bits).
* **Software Selection (Instalación):** * **Desmarcar** todos los entornos de escritorio (GNOME, XFCE, KDE, etc.).
  * **Dejar marcada** únicamente la opción de "Standard system utilities".

---

## 2. Servidor Gráfico y Gestor de Ventanas (Openbox)
Para levantar la interfaz gráfica solo cuando sea necesario y consumir la mínima RAM posible.

**Instalación:**
```bash
sudo apt update && sudo apt upgrade
sudo apt install xserver-xorg xinit openbox xterm
```

**Configuración del arranque (`startx`):**
Indicar al sistema qué entorno iniciar por defecto:
```bash
echo "exec openbox-session" > ~/.xinitrc
```
*(Uso: Escribe `startx` en la consola para entrar al entorno gráfico. Para salir y volver a la consola pura, haz clic derecho en el fondo de Openbox y selecciona "Log out").*

---

## 3. Gestión de Red (WiFi) y DNS
Uso de NetworkManager para gestionar la red con una interfaz gráfica desde la propia terminal (`nmtui`).

**Instalación:**
```bash
sudo apt install network-manager
```

**Desbloquear tarjeta WiFi (Si se configuró en la instalación):**
1. Editar el archivo de interfaces:
   ```bash
   sudo nano /etc/network/interfaces
   ```
2. Comentar (añadir `#` al inicio) todas las líneas relacionadas con la tarjeta inalámbrica (ej. `wlan0`), dejando intactas `auto lo` e `iface lo inet loopback`.
3. Reiniciar el servicio de red:
   ```bash
   sudo systemctl restart networking
   ```

**Configurar DNS de Quad9:**
1. Ejecutar `nmtui`.
2. Ir a **Modificar una conexión** > Seleccionar tu WiFi > **Configuración IPv4**.
3. Añadir Servidores DNS: `9.9.9.9` y `149.112.112.112`.
4. Marcar la casilla **"Ignorar los DNS obtenidos automáticamente"**.
5. Guardar y salir.

---

## 4. Aplicaciones Ultraligeras
Aplicaciones seleccionadas para operar con 2GB de RAM en un entorno de 32 bits:

```bash
# Navegadores Web (Extremo vs Moderno equilibrado) y firefox si fuese necesario
sudo apt install dillo
sudo apt install epiphany-browser
sudo apt install firefox-esr

# Ofimática ligera (Documentos y Hojas de cálculo)
sudo apt install abiword gnumeric

# Editor de texto gráfico simple
sudo apt install mousepad
```

---

## 5. Teclas de Función (Volumen y Brillo)
Mapear las teclas `Fn` directamente en Openbox sin demonios de escritorio en segundo plano.

**Instalar herramientas base:**
```bash
sudo apt install alsa-utils brightnessctl
```

**Configurar atajos en Openbox:**
```bash
mkdir -p ~/.config/openbox
cp /etc/xdg/openbox/rc.xml ~/.config/openbox/
nano ~/.config/openbox/rc.xml
```

Añadir lo siguiente dentro de la sección `<keyboard>`:
```xml
    <keybind key="XF86AudioRaiseVolume">
      <action name="Execute"><command>amixer set Master 5%+</command></action>
    </keybind>
    <keybind key="XF86AudioLowerVolume">
      <action name="Execute"><command>amixer set Master 5%-</command></action>
    </keybind>
    <keybind key="XF86AudioMute">
      <action name="Execute"><command>amixer set Master toggle</command></action>
    </keybind>

    <keybind key="XF86MonBrightnessUp">
      <action name="Execute"><command>brightnessctl set +10%</command></action>
    </keybind>
    <keybind key="XF86MonBrightnessDown">
      <action name="Execute"><command>brightnessctl set 10%-</command></action>
    </keybind>
```
Aplicar los cambios:
```bash
openbox --reconfigure
```

---

## 6. VPN: WireGuard (Cliente pfSense con PSK)
Conexión de grado empresarial a firewall pfSense con cero consumo de RAM en reposo.

**Instalación:**
```bash
sudo apt install wireguard resolvconf wireguard-tools
```

**Generar Claves de Cliente:**
```bash
# Claves pública y privada
wg genkey | tee clave_privada | wg pubkey > clave_publica

# Clave precompartida (Seguridad extra simétrica)
wg genpsk > clave_precompartida
```

**Crear Archivo de Configuración:**
```bash
sudo nano /etc/wireguard/wg0.conf
```

**Plantilla (`wg0.conf`):**
```ini
[Interface]
PrivateKey = [TU_CLAVE_PRIVADA_BASE64]
Address = 10.0.0.2/32

[Peer]
PublicKey = [CLAVE_PUBLICA_PFSENSE_BASE64]
PresharedKey = [TU_CLAVE_PRECOMPARTIDA_BASE64]
Endpoint = [IP_PFSENSE]:51820
AllowedIPs = 0.0.0.0/0
PersistentKeepalive = 25
```

**Proteger el archivo (Crítico):**
```bash
sudo chmod 600 /etc/wireguard/wg0.conf
```

---

## 7. Batería y Alias (Atajos de Terminal)
Facilitar el uso de la VPN y ver la batería sin escribir comandos largos.

**Instalar lector de batería ACPI:**
```bash
sudo apt install acpi
```

**Configurar alias de usuario:**
```bash
nano ~/.bashrc
```

Añadir al final del archivo:
```bash
# Atajos de VPN
alias vpn-on='sudo wg-quick up wg0'
alias vpn-off='sudo wg-quick down wg0'

# Atajo de Batería
alias bat='acpi -b'
```

Aplicar los cambios en la sesión actual:
```bash
source ~/.bashrc
```

---

## 8. Solución de Problemas Frecuentes
* **Comando wg-quick no encontrado:** Usar la ruta completa (`/usr/bin/wg-quick`) o verificar la instalación de `wireguard-tools`.
* **Luz de Bloq Mayús apagada en TTY/Openbox:** Instalar controladores de entrada `sudo apt install xserver-xorg-input-libinput xserver-xorg-input-evdev` o forzar el mapeo con `xset led named "Caps Lock"`.
