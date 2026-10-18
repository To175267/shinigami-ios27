# Shinigami — iOS 27 Ramdisks (A13 Bionic)

Construye los ramdisks SSH de **iOS 27** para la generación **A13 Bionic**
(iPhone 11 / 11 Pro / 11 Pro Max / SE2) **sin tener una Mac**, usando un
runner macOS de GitHub Actions.

---

## ¿Qué es esto?

El ramdisk es un **mini-SO que se carga en RAM** para hablar con el iPhone por SSH
(`root` / `alpine` en `127.0.0.1:2222`) y desde ahí hacer:

- **Backup / Restore** de los registros de activación
- **Activación HELLO no-SN**
- **Bloqueo de OTA**
- **MDM**
- **Borrado (erase)**

El ramdisk necesita, por cada device y versión de iOS: `iBSS`/`iBEC` parcheados,
`kernelcache`, `DeviceTree`, `ramdisk`, `trustcache` y los firmwares
(ANE/AOP/AVE/GFX/ISP/SIO) + `sep-firmware`.

**Armar eso a mano requiere macOS** (el ramdisk es APFS y hay que expandirlo).
Este repo lo hace **en la nube**.

---

## 🚫 A12 no tiene iOS 27

| Chip | Device | Max iOS |
|---|---|---|
| **A12** | iPhone XR / XS / XS Max · iPad mini 5 · Air 3 · iPad 8 | **18.7.x** |
| **A13** | **iPhone 11 / 11 Pro / 11 Pro Max / SE2** | **27.x** ✅ |

Apple **no liberó** iOS 27 para A12 — no hay nada que construir para esos equipos.

---

## 🚀 Cómo usarlo (paso a paso)

### 1. Crear el repo
En GitHub: **New repository** → nombre (ej. `shinigami-ios27`) → **Public**
(público = runners gratis) → **Create**.

### 2. Subir este workflow
En el repo: **Add file** → **Upload files** → arrastrá la carpeta
`.github` (o creá el archivo a mano en
`.github/workflows/build-ios27.yml` y pegá el contenido).

> ⚠️ La ruta importa: tiene que quedar en **`.github/workflows/`**, si no
> GitHub no lo detecta.

### 3. Correrlo
Pestaña **Actions** → **Build iOS 27 Ramdisks** → botón **Run workflow**:
- **version**: `27.0.1` (la release de iOS 27; si querés otra, cambiala)
- **devices**: dejá el default (los 4 A13)

→ **Run workflow**. Se lanzan **4 jobs en paralelo** (uno por device).

### 4. Esperar
Entre **15 y 40 minutos** por device. El límite por job es 6 h (sobra).

### 5. Bajar el resultado
Cuando terminen, entrá al run → sección **Artifacts** →
`bootchain-iPhone12,1-27.0.1` → **Download**.

Dentro vas a encontrar:

```
bootchain/n104-27.0.1-XXXX-ramdisk/
├── chain.info
├── iBSS.patched.bin
├── iBEC.img4
├── devicetree.img4
├── kernelcache.img4
├── ramdisk.img4
├── trustcache.img4
├── ANE.img4 · AOP.img4 · AVE.img4 · GFX.img4 · ISP.img4 · SIO.img4
└── sep-firmware.img4
```

---

## 🔧 Convertirlo para el server (Shinigami)

El artifact viene en el formato del **builder**; el server
(`bypass/ramdisk2/bootchain.php`) espera su propio layout. La conversión:

1. **Renombrar** los archivos a la convención del server
   (`ramdisk.img4` → `27.0.1.dmg.img4`, etc.).
2. **Armar el `manifest.json`** (roles + comandos + `sha256`).
3. **Agregar el logo** (`applelogo_*.im4p.img4`) — el runtime lo reemplaza si
   activás `IREALM_CUSTOM_LOGO=1`.
4. **Empaquetar** en `bootchain-v2/packages/<paquete>/` + `.zip` + `.files.json`.

> Ese paso lo hacemos localmente (tenemos los scripts `pack-logo.ps1`,
> `make-logo-img4.ps1` y el generador de manifiestos).

---

## ⚠️ Notas técnicas

| Tema | Detalle |
|---|---|
| **Runner Intel** | Los binarios de `tools/darwin/` son x86_64 → usamos `macos-13`. Si GitHub lo retira, cambiá a `macos-latest` y agregá `softwareupdate --install-rosetta --agree-to-license`. |
| **Disco** | El runner tiene ~14 GB. El builder usa `pzb` (baja **solo** lo necesario del IPSW), así que entra. |
| **El device NO se usa acá** | El build no toca el iPhone. El device solo se usa para **arrancar** el ramdisk (`boot.sh`, local, con Pwned DFU / Pi Pico). |
| **Costo** | Repo **público** → runners macOS gratis. Repo privado → consume minutos (macOS cuenta **x10**). |
| **Pwned DFU** | Para ARRANCAR el ramdisk en el device igual necesitás **usbliter8 / Pi Pico** (eso es local, no se puede hacer en la nube). |

---

## Fuente

- Builder: [`Rust505/iOS-ramdisk-builder-A12-A13`](https://github.com/Rust505/iOS-ramdisk-builder-A12-A13)
- Descarga de IPSWs: [`api.ipsw.me`](https://api.ipsw.me)
