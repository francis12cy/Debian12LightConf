# Documentación: Solución de Screen Tearing en Debian 13 (Openbox + NVIDIA Optimus)

Esta documentación recopila el diagnóstico, la solución implementada, los comandos clave y la resolución de problemas relacionados con la desincronización vertical (*screen tearing*) en un entorno de escritorio ligero **Openbox** sobre **Debian 13**, utilizando un ordenador portátil con arquitectura gráfica híbrida (**Intel/AMD + NVIDIA GTX 1650**).

---

## 1. Diagnóstico del Problema

El *screen tearing* (desgarro de pantalla) se manifiesta como un corte horizontal en las ventanas o en el navegador al hacer scroll o mover un elemento rápidamente. Una parte de la pantalla parece redibujarse con retraso respecto a la otra.

En este entorno específico, el problema responde a dos motivos estructurales:
1. **Openbox es un gestor de ventanas puro:** Por diseño y minimalismo, Openbox no incluye un compositor de ventanas integrado. Al no haber composición, no hay sincronización vertical (V-Sync) nativa para coordinar los fotogramas con la tasa de refresco del monitor.
2. **Arquitectura NVIDIA Optimus (Gráficas Híbridas):** En la gran mayoría de ordenadores portátiles modernos, la pantalla integrada está conectada físicamente a la tarjeta de vídeo integrada de bajo consumo (Intel o AMD). La tarjeta potente (NVIDIA GTX 1650) actúa como un acelerador secundario bajo demanda. Debido a esto, la utilidad `nvidia-settings` oculta la sección **X Server Display Configuration**, impidiendo forzar la opción *Force Full Composition Pipeline* desde el driver propietario de NVIDIA.

---

## 2. Solución Aplicada

Dado que la interfaz gráfica principal es gestionada a través de la tarjeta integrada, la solución óptima es instalar un compositor externo ligero que se encargue de la sincronización por hardware. El estándar actual para X11 es **Picom**.

### Paso 1: Instalación de Picom
Se descargan e instalan los paquetes del compositor desde las ramas oficiales de Debian:

```

```text
File written successfully to solucion_tearing_openbox.md

```bash
sudo apt update
sudo apt install picom

```

### Paso 2: Ejecución de prueba en tiempo real

Para verificar que solucione el desgarro de inmediato sin necesidad de reiniciar, se lanza el proceso con parámetros específicos de aceleración:

```bash
picom --backend glx --vsync -b

```

* **`--backend glx`**: Fuerza al compositor a utilizar OpenGL (aceleración por hardware), fundamental para resolver el desgarro de forma eficiente en configuraciones híbridas.
* **`--vsync`**: Activa el refresco de sincronización vertical estricto.
* **`-b`**: Ejecuta el comando en segundo plano (*daemon*), permitiendo cerrar o seguir usando esa misma terminal.

### Paso 3: Automatización en el inicio del sistema

Para que la solución sea permanente y cargue de forma automática al iniciar sesión:

1. Se edita el archivo de autoarranque de Openbox:
```bash
nano ~/.config/openbox/autostart

```


2. Se añade al final del archivo la orden de Picom. Es obligatorio añadir el símbolo `&` al terminar la línea:
```bash
# Iniciar compositor para evitar Screen Tearing
picom --backend glx --vsync -b &

```



---

## 3. Configuración de Archivos

### Archivo: `~/.config/openbox/autostart`

Un ejemplo base de cómo debe lucir este archivo para asegurar un correcto inicio de los servicios esenciales:

```bash
# Configuración del teclado / idioma (ejemplo)
setxkbmap es &

# Gestor de fondo de pantalla (ejemplo)
feh --bg-scale /ruta/a/tu/imagen.jpg &

# Panel de escritorio (ejemplo si usas tint2)
tint2 &

# --- SOLUCIÓN DE TEARING ---
# El símbolo '&' al final evita el bloqueo de la sesión
picom --backend glx --vsync -b &

```

### Archivo Opcional: `~/.config/picom/picom.conf`

Picom funciona perfectamente con sus valores internos por defecto. No obstante, si se desea personalizar el comportamiento o añadir efectos estéticos (sombras, opacidades), se puede generar un archivo local:

```bash
mkdir -p ~/.config/picom
cp /etc/xdg/picom.conf ~/.config/picom/picom.conf

```

Dentro de ese archivo generado, los parámetros clave de renderizado deben permanecer configurados obligatoriamente así:

```conf
backend = "glx";
vsync = true;

```

---

## 4. Errores Frecuentes y Solución de Problemas

### Error 1: El escritorio se congela o no termina de cargar al iniciar sesión

* **Causa:** Se omitió colocar el símbolo `&` al final de la línea en `~/.config/openbox/autostart`. Sin este ampersand, Openbox interpreta el comando en primer plano, quedándose atascado esperando a que Picom termine para poder seguir cargando el resto de componentes del sistema.
* **Solución:** Presionar `Ctrl + Alt + F3` para saltar a una consola TTY en modo texto. Iniciar sesión con tus credenciales y corregir el archivo ejecutando:
```bash
nano ~/.config/openbox/autostart

```


Asegurarse de poner el `&` al final, guardar con `Ctrl+O` y salir con `Ctrl+X`. Reiniciar el gestor de login con `sudo systemctl restart lightdm` (o el gestor que utilices).

### Error 2: "X Server Display Configuration" sigue sin aparecer en nvidia-settings

* **Explicación:** No es un error subsanable ni un fallo de controladores. Es la consecuencia directa del diseño de hardware de los portátiles Optimus. La GPU integrada manda sobre la pantalla principal. No intentes alterar manualmente el archivo `/etc/X11/xorg.conf` para resolver esto, ya que podrías romper el inicio del entorno gráfico.

### Error 3: Caída de rendimiento (FPS) o micro-tirones al ejecutar videojuegos

* **Síntoma:** Al abrir un juego pesado a pantalla completa, el rendimiento se degrada o se percibe inestabilidad debido a que Picom intenta procesar la composición sobre el juego.
* **Solución:** Habilitar una directiva que desactive la composición automáticamente en aplicaciones que tomen el control total de la pantalla. Para ello, edita `~/.config/picom/picom.conf` y busca/añade la siguiente propiedad:
```conf
unredir-if-possible = true;

```



### Error 4: Parpadeos (Flickering) en el navegador web

* **Causa:** Incompatibilidad puntual entre la aceleración de hardware propia de navegadores basados en Chromium/Firefox y el motor de renderizado GLX del compositor.
* **Solución:** Si experimentas parpadeos parásitos extraños, puedes probar a cambiar el motor de Picom al modo nativo de X11 modificando el comando a:
```bash
picom --backend xrender --vsync -b

```


*(Nota: Intenta mantener `glx` prioritariamente, ya que suele ser sustancialmente más eficiente eliminando el tearing).*

---

## 5. Tabla de Comandos Útiles de Mantenimiento

| Comando | Función / Descripción |
| --- | --- |
| `pkill picom` | Mata el proceso de Picom de forma inmediata (útil si se cuelga o para probar configuraciones limpias). |
| `picom --backend glx --vsync -b` | Lanza Picom manualmente de forma óptima en segundo plano. |
| `glxinfo | grep OpenGL` | Permite verificar si Debian está haciendo uso correcto de la aceleración por hardware (requiere el paquete `mesa-utils`). |
| `openbox --reconfigure` | Recarga las configuraciones nativas de Openbox sin necesidad de cerrar la sesión activa. |
| """ |  |

file_path = "solucion_tearing_openbox.md"
with open(file_path, "w", encoding="utf-8") as f:
f.write(markdown_content)

print(f"File written successfully to {file_path}")
