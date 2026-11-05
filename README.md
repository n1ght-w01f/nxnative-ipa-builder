# 🎮 Switch → IPA Builder (nxnative en GitHub Actions)

Este repositorio **no contiene ninguna parte de ningún juego** ni claves: es
una **plantilla de compilación** que usa [nxnative](https://github.com/seedds/nxnative)
para convertir **tus propios juegos de Nintendo Switch** (`.nsp` / `.nsz` /
`.xci`, junto con tus `prod.keys`) en una **IPA sin firmar** lista para
instalarse con AltStore / Sideloadly / LiveContainer en iPhone, iPad o Mac.

Todo el proceso ocurre en un runner `macos-latest` **efímero** en GitHub
Actions: los archivos sensibles se descargan en memoria del runner, se
compilan y se destruyen al acabar. **Nunca se sube nada al historial de Git.**

---

## 🚀 Guía rápida (3 pasos)

### 1️⃣ Haz Fork (o clona) este repo

1. Pulsa **Fork** arriba a la derecha (o **Import repository** si prefieres
   tenerlo privado desde el principio).
2. En tu fork ya está incluido el workflow de compilación.
   > ⚠️ Si lo clonas en local y lo subes a GitHub, comprueba que tu `.gitignore`
   > está intacto: así es imposible subir por accidente un `.nsp` o `prod.keys`.

### 2️⃣ Consigue enlaces de descarga **directos** para tus archivos

El workflow descarga tus archivos por `curl -L` desde una URL. Necesitas
**dos URLs**: la de tu archivo de juego y la de tu `prod.keys`. Opciones fáciles:

| Servicio | Cómo hacerlo |
|---|---|
| **Cloudflare R2** (recomendado) | Sube el archivo a un bucket público (o activa *Presigned URL* con caducidad). La URL resultante (`https://<account>.r2.dev/...` o la presignada) es un enlace directo. |
| **Google Drive** | Sube el archivo, comparte con "cualquiera con el enlace", copia el `FILE_ID` de `https://drive.google.com/file/d/FILE_ID/view` y usa `https://drive.google.com/uc?export=download&id=FILE_ID`. Si pesa mucho aparecerá una página de confirmación: descárgalo una vez en tu navegador y copia el `confirm=XXXX` de la URL real. |
| **transfer.sh** | Desde tu terminal: `curl --upload-file prod.keys https://transfer.sh/prod.keys` — devuelve la URL directa. Útil para archivos pequeños como las keys. |
| **ngrok / cloudflared** (túnel local) | Sirve el archivo en tu PC: `python3 -m http.server 8080` y luego `cloudflared tunnel --url http://localhost:8080`. La URL pública que imprime (p. ej. `https://algo.trycloudflare.com`) hace de enlace directo: `.../prod.keys`. |

> 💡 **Recomendación final:** usa una URL **temporal o presignada** con
> caducidad corta (R2/S3, transfer.sh) cuando puedas. Así el enlace muere
> después de la compilación.

### 3️⃣ Lanza el workflow y descarga tu IPA

1. En GitHub, entra en tu repo → pestaña **Actions** → a la izquierda
   selecciona **«Compilar juego de Switch a IPA (nxnative)»**.
2. Pulsa **Run workflow** (→ *Run workflow* sobre el desplegable `main`).
3. Rellena los campos:
   - **`nsp_url`** → enlace directo de tu juego base (`.nsp`, `.xci`, `.nsz`,
     `.xcz`, `.nup` o incluso un `.zip` con la carpeta del juego).
   - **`extra_urls`** (opcional) → enlaces directos de **updates y DLC del
     mismo juego**, separados por espacios. Van a la misma carpeta y
     nxnative detecta las versiones por title ID.
   - **`keys_url`** (opcional) → enlace directo de tu `prod.keys`.
     **Solo los juegos de Switch 1 la necesitan** (los de Switch 2 vía
     `.nup`/dump no). Escribe `none` para ignorar el Secret guardado.
   - **`game_name`** (opcional) → nombre identificador, p. ej. `mario-wonder`.
   - **`language`** (opcional) → `es`, `ja`, `en-GB`... (vacío = inglés).
   - **`package`** (opcional) → versión a empaquetar, ej. `v0` (vacío = la
     más reciente que haya en la carpeta).
   - **`gpu`** (opcional) → archivos de pipeline GPU extra, ej. `g18p,g17p`
     (recomendado por la documentación de nxnative para Breath of the Wild).
   - **`app_name`** (opcional) → nombre que verás en el home del iPhone.
   - **`show_fps`** → muestra el contador de FPS en la esquina.

> 🔒 **Recomendado en repos públicos:** en un repo público cualquiera puede
> ver los *inputs* de cada ejecución en la pestaña *Actions*. Para que tus
> URLs no se vean en absoluto, guárdalas una vez en
> **Settings → Secrets and variables → Actions → New repository secret**
> con los nombres **`KEYS_URL`** y **`NSP_URL`** (los campos del workflow
> pueden quedar vacíos y el workflow usará los Secrets automáticamente).
4. Pulsa **Run workflow** y espera ⏳ (**30–60 min** la primera vez; las
   siguientes usan mejor la caché). Al terminar, en la misma ejecución verás
   en **Artifacts** el resultado con nombre `<game_name>-ipa`. Descárgalo
   (tiene **retención de 1 día** — se borra solo), descomprime e instala la
   IPA con AltStore, Sideloadly o LiveContainer.
   > 📌 Si tu herramienta de sideload ofrece **«Increased Memory Limit»**,
   > actívala: los juegos grandes lo necesitan.

---

## 🧩 Juegos soportados y casos especiales

### Updates y DLC (varios archivos del mismo juego)

El script de nxnative acepta una **carpeta con todos los archivos del juego**:
base + updates + DLC (`.nsp`, `.xci`, `.nsz`, `.xcz`). En el workflow solo
tienes que poner la URL del **base** en `nsp_url` y el resto (updates/DLC)
en **`extra_urls`** separados por espacios:

- `nsp_url` → `https://tunel.trycloudflare.com/BreathOfTheWild%20%5Bbase%5D.xci`
- `extra_urls` → `https://tunel.trycloudflare.com/BOTW%20%5Bv208896%5D.nsp https://tunel.trycloudflare.com/DLC.nsp`

> 💡 Si en vez de archivos sueltos tienes una **carpeta de dump** con el
> juego (`base/`, `updates/<versión>/`, `dlc/...`), comprímela en un `.zip`,
> súbelo y ponlo en `nsp_url`: el workflow lo descomprime solo dentro de la
> carpeta del juego.

### Switch 2 (formato NUP) — sin prod.keys

Los juegos de **Switch 2** se compilan desde un único contenedor **`.nup`**
(o una carpeta de sus archivos) y **NO necesitan `prod.keys`**:

- `nsp_url` → `https://tunel.trycloudflare.com/MarioKartWorld.nup`
- `keys_url` → déjalo vacío, o escribe **`none`** si tienes el Secret
  `KEYS_URL` guardado y no quieres que se use en esa build.

> ⚠️ Estado del proyecto: los juegos de Switch 2 están **testeados en Mac**
> (`--platform mac-app`); en iPhone/iPad el proyecto los marca como
> *untested*. El workflow produce la IPA igualmente.

### ¿FPS? No hay que hardcodear nada

- El frame rate **se elige en la propia app** (pantalla de ajustes que nxnative
  genera: frame rate y render scale) y muchos juegos leen el límite del código
  del juego.
- Para juegos como **Breath of the Wild** la tabla oficial de nxnative indica
  **«60 fps with 1.9.0»**: la clave es incluir el **update 1.9.0** en
  `extra_urls` junto al juego base, y empaquetar esa combinación.
- El input **`show_fps`** añade el contador de FPS en pantalla para verificarlo.
- Para BotW la documentación recomienda además `gpu` = `g18p,g17p` (incluye el
  archivo de pipelines para más GPUs aparte de la A19/A19 Pro).

---

## 🧩 ¿Y si el artefacto viene dividido en varios `.zip`?

Cuando el juego ocupa **más de 2 GB**, el workflow lo trocea en volúmenes ZIP
de 2000 MB: `mario-odyssey_parts.zip`, `mario-odyssey_parts.z01`,
`.z02`, ... Para **unirlos** y recuperar la IPA:

**Windows (con 7-Zip — más fácil):**
1. Descarga **todas** las partes en la misma carpeta.
2. Clic derecho sobre `mario-odyssey_parts.zip` → *7-Zip → Extraer en...*
   (7-Zip entiende los volúmenes automáticamente).

**Windows (símbolo del sistema, sin 7-Zip):**
```bat
copy /b mario-odyssey_parts.z01+mario-odyssey_parts.z02+...+mario-odyssey_parts.zip mario-odyssey_joined.zip
```
*(los volúmenes `.z01`, `.z02`... en orden, añadiendo el `.zip` el último)*

**macOS / Linux:**
```bash
zip -s 0 mario-odyssey_parts.zip --out mario-odyssey_full.zip
unzip mario-odyssey_full.zip
```

---

## ⚠️ Aviso legal / Disclaimer

- Este repositorio **NO aloja, distribuye ni enlaza** en su historial ni en
  sus artefactos ningún archivo con copyright: ni ROMs `.nsp`, ni `prod.keys`,
  ni contenido de Nintendo. Solo contiene una *plantilla de compilación*.
- Para usarlo necesitas ser legítimo propietario de tus juegos (copia de
  seguridad propia) y de tus claves. **No lo uses para compartir o
  distribuir contenido que no poseas.**
- Todo el material descargado durante una ejecución se destruye con el
  runner; el artefacto tiene una retención máxima de **1 día**.
- Este proyecto no está afiliado a Nintendo ni a ningún editor.
- `nxnative` es un proyecto **independiente** (licencia MIT) y este template
  tampoco es oficial de Nintendo.

---

## 🛠️ Soporte técnico

- Errores de compilación de un juego concreto → abre un *Issue* en el repo
  de [nxnative](https://github.com/seedds/nxnative/issues) (donde los
  mantenedores cuidan del motor).
- Errores de esta plantilla → *Issues* de este repositorio.
