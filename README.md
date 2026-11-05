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
3. Rellena los 3 campos:
   - **`keys_url`** → enlace directo de tu `prod.keys`.
   - **`nsp_url`** → enlace directo de tu juego.
   - **`game_name`** (opcional) → nombre bonito, p. ej. `mario-odyssey`.
4. Pulsa **Run workflow** y espera ⏳ (**30–60 min** la primera vez; las
   siguientes usan mejor la caché). Al terminar, en la misma ejecución verás
   en **Artifacts** el resultado con nombre `<game_name>-ipa`. Descárgalo
   (tiene **retención de 1 día** — se borra solo), descomprime e instala la
   IPA con AltStore, Sideloadly o LiveContainer.
   > 📌 Si tu herramienta de sideload ofrece **«Increased Memory Limit»**,
   > actívala: los juegos grandes lo necesitan.

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
