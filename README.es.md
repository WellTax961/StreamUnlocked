<div align="left">
  <a href="README.en.md"><strong>View in English</strong></a>
</div>

# ¡Bienvenido/a a StreamUnlocked!

Aquí te guiaré, paso a paso, por el proceso técnico para descargar streams de video (HLS/DASH) protegidos con DRM para tu uso personal y sin conexión. El propósito de esta guía es puramente educativo, como permitir la creación de respaldos personales o facilitar la investigación en seguridad.

> [!WARNING]
> **Descargo de Responsabilidad:** El que puedas descargar contenido no significa que tengas el derecho legal de hacerlo. Respeta siempre las leyes de derechos de autor y los términos de servicio de cada plataforma. Te ofrezco esta guía únicamente con fines educativos. Procede bajo tu propio riesgo.

---

## 📋 Tabla de Contenido

- [Prerrequisitos](#-prerrequisitos)
- [Guía Paso a Paso](#-guía-paso-a-paso)
  - [1. Capturar el PSSH (Init Data)](#1-capturar-el-pssh-init-data)
  - [2. Encontrar la URL de la Licencia](#2-encontrar-la-url-de-la-licencia)
  - [3. Obtener las Claves de Descifrado](#3-obtener-las-claves-de-descifrado)
  - [4. Localizar el Archivo Manifiesto (.m3u8 o .mpd)](#4-localizar-el-archivo-manifiesto-m3u8-o-mpd)
  - [5. Descargar y Descifrar el Stream](#5-descargar-y-descifrar-el-stream)
- [Licencia](#-licencia)

---

## 🛠️ Prerrequisitos

Antes de empezar, necesitarás tener instaladas y configuradas las siguientes herramientas.

1.  **Tampermonkey**
    -   Es un gestor de scripts para tu navegador. Lo puedes conseguir en [tampermonkey.net](https://www.tampermonkey.net/).

2.  **Script EME Logger**
    -   Un script que se conecta a las Extensiones de Medios Cifrados (EME) de tu navegador para registrar las claves de descifrado. Lo puedes instalar desde [Greasy Fork](https://greasyfork.org/es/scripts/373903-eme-logger).

3.  **N_m3u8DL-RE**
    -   Una herramienta de línea de comandos muy potente y eficiente para bajar streams HLS. Te recomiendo descargar la última versión desde su [repositorio oficial en GitHub](https://github.com/nilaoda/N_m3u8DL-RE/releases).

4.  **FFmpeg**
    -   El kit de herramientas esencial para procesar video y audio. Lo necesitarás para multiplexar (o sea, juntar) los archivos de video y audio. Descárgalo en [ffmpeg.org](https://ffmpeg.org/download.html).

5.  **Bento4**
    -   Un conjunto de utilidades para MP4 y DRM, fundamental para el proceso de descifrado. Puedes bajar los binarios desde la [página de Bento4](https://www.bento4.com/downloads/).

> [!IMPORTANT]
> Para que puedas usar `FFmpeg` y `Bento4` sin problemas desde la terminal, es **fundamental** que agregues sus carpetas de instalación a la variable de entorno `PATH` de tu sistema.

---

## 🚀 Guía Paso a Paso

Sigue estos pasos con atención para que logres descargar y descifrar el stream que te interesa.

### 1. Capturar el PSSH (Init Data)

El PSSH (Protection System Specific Header) es un bloque de datos que el sistema DRM necesita para poder pedir la licencia de descifrado.

-   Con Tampermonkey y el script EME Logger ya activos, entra en la página del servicio de streaming y reproduce el video.
-   Abre las Herramientas de Desarrollador de tu navegador (con `F12` o `Ctrl+Shift+I`) y ve a la pestaña **Consola**.
-   Busca un mensaje del script EME Logger que diga **Init Data**. Verás una cadena de texto muy larga en base64.
    ```
    EME Logger: Init Data: AAAAMnBzc2gAAAAA7e+LqXnWSs6jyCfc1R0h7QXXXXXXXX...
    ```
-   Copia esa cadena de `Init Data` completa. Ese es tu PSSH.

### 2. Encontrar la URL de la Licencia

Esta es la dirección a la que el reproductor le envía el PSSH para conseguir la licencia.

-   En las Herramientas de Desarrollador, ve ahora a la pestaña **Red**.
-   En el campo para filtrar, busca palabras como `license` o `widevine`.
-   Deberías ver una solicitud a una URL del tipo `https://[dominio-del-servicio].com/api/widevine/license`.
-   Haz clic derecho sobre esa solicitud, elige **Copiar > Copiar como fetch** y pega el contenido en un bloc de notas. Lo usarás en un momento.

### 3. Obtener las Claves de Descifrado

¡Manos a la obra! Vamos a cambiar el PSSH y la licencia por las claves que necesitamos.

-   Ve a un servicio de resolución de DRM como [cdrm-project.com](https://cdrm-project.com/).
-   Haz clic en el botón **Paste from fetch** y pega lo que copiaste en el paso anterior. Esto debería rellenar automáticamente la URL de la licencia y otros datos.
-   Si el campo `PSSH` está vacío, pega manualmente la cadena `Init Data` que conseguiste antes.
-   Dale a **Submit**. Si todo sale bien, el servicio te devolverá una o más claves con el formato `KID:KEY`.
    ```
    SUCCESS:
    edbec2b3d11d44d18549b5ee4912fe6d:1a93bf9ab00e539954d1e4e2eb503b78
    ```
-   Copia estas claves. ¡Ya casi lo tienes!

### 4. Localizar el Archivo Manifiesto (.m3u8 o .mpd)

Este archivo es la "lista de reproducción" que le dice al programa dónde están todos los pedacitos de video y audio.

-   Vuelve a la pestaña **Red** de las Herramientas de Desarrollador.
-   Filtra por `.m3u8` (si es un stream HLS) o `.mpd` (si es DASH). Quizás tengas que recargar la página o reiniciar el video para que aparezca.
-   Cuando encuentres la URL del manifiesto principal, haz clic derecho sobre ella y elige **Copiar > Copiar dirección del enlace**.

### 5. Descargar y Descifrar el Stream

Llegó el momento de usar `N_m3u8DL-RE` para juntar todo.

-   Abre tu terminal (Símbolo del sistema, PowerShell o Terminal).
-   Usa la siguiente estructura de comando, reemplazando los textos en mayúsculas con la información que has recolectado.

    ```bash
    # Estructura del comando
    N_m3u8DL-RE "URL_DEL_MANIFIESTO" --key KID:KEY --save-name "nombre_de_tu_archivo"
    
    # --- Ejemplo ---
    # Esto descargará y descifrará el stream, guardando el archivo final como "Mi-Video-Genial.mp4"
    N_m3u8DL-RE "https://ruta-hacia-tu-video/manifest.m3u8?token=..." \
      --key edbec2b3d11d44d18549b5ee4912fe6d:1a93bf9ab00e539954d1e4e2eb503b78 \
      --save-name "Mi-Video-Genial"
    ```

-   Cuando el proceso termine, encontrarás un archivo `.mp4` completamente descifrado en la carpeta de salida. ¡Pruébalo con un reproductor como VLC para confirmar que funciona!

---
## 📄 Licencia

He publicado este proyecto bajo la Licencia MIT. Puedes ver todos los detalles en el archivo [LICENSE](LICENSE).
