<div align="left">
  <a href="README.es.md"><strong>Ver en Español</strong></a>
</div>

# Welcome to StreamUnlocked!

In this guide, I'll walk you through the technical process of downloading DRM-protected video streams (HLS/DASH) for your own personal, offline use. My goal is to provide this information for educational purposes, like creating personal backups or for security research.

> [!WARNING]
> **Disclaimer:** The ability to download content does not grant you the legal right to do so. Always respect the copyright laws in your jurisdiction and the terms of service of the streaming platform. I provide this guide for educational purposes only. Proceed at your own risk.

---

## 📋 Table of Contents

- [Prerequisites](#-prerequisites)
- [Step-by-Step Guide](#-step-by-step-guide)
  - [1. Capture the PSSH (Init Data)](#1-capture-the-pssh-init-data)
  - [2. Find the License URL](#2-find-the-license-url)
  - [3. Obtain the Decryption Keys](#3-obtain-the-decryption-keys)
  - [4. Locate the Manifest File (.m3u8 or .mpd)](#4-locate-the-manifest-file-m3u8-or-mpd)
  - [5. Download & Decrypt the Stream](#5-download--decrypt-the-stream)
- [License](#-license)

---

## 🛠️ Prerequisites

Before you begin, you'll need to have the following tools installed and configured.

1.  **Tampermonkey**
    -   A user script manager for your browser. You can get it at [tampermonkey.net](https://www.tampermonkey.net/).

2.  **EME Logger Script**
    -   A user script that hooks into your browser's Encrypted Media Extensions (EME) to log decryption keys. You can install it from [Greasy Fork](https://greasyfork.org/en/scripts/373903-eme-logger).

3.  **N_m3u8DL-RE**
    -   A powerful and efficient command-line tool for downloading HLS streams. You should get the latest release from the [official GitHub repo](https://github.com/nilaoda/N_m3u8DL-RE/releases).

4.  **FFmpeg**
    -   This is the essential toolkit for video and audio processing, which you'll need for muxing (combining) media files. Download it from [ffmpeg.org](https://ffmpeg.org/download.html).

5.  **Bento4**
    -   A suite of tools for MP4 and DRM, which is necessary for the decryption process. You can download the binaries from the [Bento4 website](https://www.bento4.com/downloads/).

> [!IMPORTANT]
> For `FFmpeg` and `Bento4` to work correctly from the command line, you **must** add their installation directories to your system's `PATH` environment variable.

---

## 🚀 Step-by-Step Guide

Follow these steps carefully, and you'll be able to download and decrypt your target stream.

### 1. Capture the PSSH (Init Data)

The PSSH (Protection System Specific Header) contains metadata the DRM system needs to request a decryption license.

-   With Tampermonkey and the EME Logger script active, go to the streaming service and start playing the video.
-   Open your browser's Developer Tools (`F12` or `Ctrl+Shift+I`) and switch to the **Console** tab.
-   Look for a log from the EME Logger script that contains the **Init Data**. It will be a base64 string.
    ```
    EME Logger: Init Data: AAAAMnBzc2gAAAAA7e+LqXnWSs6jyCfc1R0h7QXXXXXXXX...
    ```
-   Copy this entire `Init Data` string. This is your PSSH.

### 2. Find the License URL

This is the endpoint where the player sends the PSSH to get a license.

-   In the Developer Tools, switch to the **Network** tab.
-   Use the filter box and search for terms like `license` or `widevine`.
-   You should see a request made to a URL like `https://[service-domain].com/api/widevine/license`.
-   Right-click this request, choose **Copy > Copy as fetch**, and paste it into a temporary text file. You'll need it soon.

### 3. Obtain the Decryption Keys

Here, we'll use the PSSH and license URL to get the keys.

-   Go to a DRM solver service like [cdrm-project.com](https://cdrm-project.com/).
-   Click the **Paste from fetch** button and paste the content you copied in the last step. This should auto-fill the License URL and other request data.
-   If it's not already there, manually paste your **PSSH** (the `Init Data` string) into the `PSSH` field.
-   Click **Submit**. If it works, the service will show you one or more decryption keys in the `KID:KEY` format.
    ```
    SUCCESS:
    edbec2b3d11d44d18549b5ee4912fe6d:1a93bf9ab00e539954d1e4e2eb503b78
    ```
-   Copy these keys. You're almost there!

### 4. Locate the Manifest File (.m3u8 or .mpd)

The manifest file is a playlist that tells the downloader where to find all the individual video and audio segments.

-   Go back to the **Network** tab in your Developer Tools.
-   Filter for `.m3u8` (for HLS streams) or `.mpd` (for DASH streams). You might need to reload the page or restart the video for it to appear.
-   Find the main manifest URL, right-click it, and choose **Copy > Copy link address**.

### 5. Download & Decrypt the Stream

Now it's time to put it all together with `N_m3u8DL-RE`.

-   Open your terminal (Command Prompt, PowerShell, or Terminal).
-   Use the following command structure, replacing the placeholders with the info you've gathered.

    ```bash
    # Basic command structure
    N_m3u8DL-RE "YOUR_MANIFEST_URL" --key KID:KEY --save-name "your_file_name"
    
    # --- Example ---
    # This will download and decrypt the stream, saving the final file as "My-Awesome-Video.mp4"
    N_m3u8DL-RE "https://path-to-your-video/manifest.m3u8?token=..." \
      --key edbec2b3d11d44d18549b5ee4912fe6d:1a93bf9ab00e539954d1e4e2eb503b78 \
      --save-name "My-Awesome-Video"
    ```

-   Once the process is complete, you'll find a fully decrypted `.mp4` file in the output directory. Test it with a media player like VLC to make sure it works!

---
## 📄 License

I've licensed this project under the MIT License. You can see the [LICENSE](LICENSE) file for details.
