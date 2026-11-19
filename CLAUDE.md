# qBittorrent de Luis (fork para Ubuntu)

Fork personal del qBittorrent oficial con cambios propios, compilado como AppImage para Ubuntu.

- Fork: https://github.com/luisp193/qBittorrent (rama `master`)
- Oficial (`upstream`): https://github.com/qbittorrent/qBittorrent
- Copia local: `C:\Users\luis.perez\Documents\IA-PROGRAMACION\qBittorrent`
- Remotos git: `origin` = fork, `upstream` = oficial
- **Base: versiones estables oficiales** (tags `release-X.Y.Z`, que salen de las ramas `vX_Y_x`), no el `master` en desarrollo del oficial. Desde V3 la base es `release-5.3.0`.
- La rama `respaldo-desarrollo` del fork guarda V1–V2, que estaban sobre el `master` en desarrollo (5.4 alpha). Solo es copia de seguridad.

## Objetivo

- Tener un qBittorrent propio para Ubuntu con cambios de Luis que **no se pierdan** con las actualizaciones de Ubuntu (el AppImage es un archivo suelto; `apt` no lo toca).
- Ir **simplificando** la interfaz. Para simplificar, preferir **ocultar** en la interfaz (menús, columnas, pestañas, opciones) en vez de borrar código interno: así hay menos conflictos al juntar versiones nuevas del oficial.
- Los textos nuevos de la interfaz van directamente en **español** dentro de `tr("...")` (no se mantienen traducciones).

## Compilación (no hay compilador en este PC)

Se compila en GitHub Actions con `.github/workflows/build_ubuntu_appimage.yaml`:

- Se lanza en cada push a `master` (y a mano con *Run workflow*). Tarda unos 15 min.
- Ubuntu `ubuntu-latest` (24.04), Qt 6.6.3, libtorrent 2.0.15, Boost 1.77, con interfaz gráfica (`GUI=ON`).
- Publica una release `V<n>` con:
  - `qBittorrent_V<n>-x86_64.AppImage` (con versión)
  - `qBittorrent-x86_64.AppImage` (nombre fijo). Enlace estable:
    https://github.com/luisp193/qBittorrent/releases/latest/download/qBittorrent-x86_64.AppImage
- El nombre que se ve en el menú de Ubuntu es `qBittorrent_V<n>` (el workflow cambia `Name=` del `.desktop`).
- El AppImage necesita Ubuntu 24.04 o posterior. Si da error de FUSE: `sudo apt install libfuse2t64`.
- Los workflows del oficial (`ci_*.yaml`, `stale_bot.yaml`) están **desactivados** en el fork desde la web de GitHub; no tocar sus archivos.

## Versión

- El número vive en `version.txt` (raíz). **Súbelo +1 en cada cambio de código** antes de hacer push.
- Cambios que no son de código (este `CLAUDE.md`) no lanzan compilación (`paths-ignore`) y no suben versión.

## Cambios propios hechos

| Versión | Cambio | Archivos |
|---|---|---|
| V1 | Workflow de compilación AppImage con versión en el nombre | `.github/workflows/build_ubuntu_appimage.yaml`, `version.txt` |
| V2 | Descargas borradas → categoría **BORRADAS**: revisión a los 2 min de arrancar y cada 24 h, y opción **"Comprobar borrado"** en el menú del botón derecho | `src/gui/transferlistwidget.cpp/.h` |
| V3 | Cambios propios recolocados sobre la versión estable oficial 5.3.0, y revisión mensual automática | `.github/workflows/sync_upstream_monthly.yaml` |

### Detalle de "BORRADAS" (V2)

Código en `src/gui/transferlistwidget.cpp`: `isTorrentDeleted()`, `moveDeletedTorrents()`, `checkDeletedAllTorrents()`, `checkDeletedSelectedTorrents()`; temporizador en el constructor de `TransferListWidget`.

- Una descarga cuenta como borrada si **ninguno** de los archivos que ya tenía descargados (progreso > 0) existe en el disco. Si se borra solo una parte, no se marca.
- No se marca si no existe ni la carpeta de destino (p. ej. disco externo desconectado), ni si está sin metadatos, comprobándose o moviéndose.
- Al marcarla: se crea la categoría `BORRADAS` si no existe, se desactiva la gestión automática (para que no mueva archivos), se pone la categoría, se **detiene** y se anota en el Registro.
- "Comprobar borrado" hace lo mismo con las seleccionadas y muestra un mensaje con el resultado.

## Revisión mensual automática

`.github/workflows/sync_upstream_monthly.yaml` se ejecuta el **día 1 de cada mes a las 06:00 UTC**. También se puede lanzar a mano con *Run workflow*.

1. Busca el tag estable más nuevo del oficial (`release-X.Y.Z`, sin beta ni rc).
2. Si ya está incluido en `master`, no hace nada.
3. Si no lo está, lo junta (`git merge`) con `master`, sube `version.txt` +1, hace commit `V<n>: qBittorrent X.Y.Z con cambios propios` y push. Ese push lanza la compilación del AppImage.
4. Después desactiva cualquier workflow del oficial que haya quedado activo (solo deben funcionar `build_ubuntu_appimage.yaml` y `sync_upstream_monthly.yaml`).
5. **Si hay conflicto**, deja todo como estaba, abre un **issue** "Conflicto al juntar la versión oficial …" con la lista de archivos, y la ejecución sale en rojo (GitHub avisa por correo). Hay que resolverlo a mano (ver abajo).

Usa el secreto **`SYNC_TOKEN`**, un token *fine-grained* de Luis solo para `luisp193/qBittorrent` con permisos Contents, Workflows y Actions en lectura y escritura. Hace falta porque el `GITHUB_TOKEN` normal no puede subir cambios en `.github/workflows` ni lanzar la compilación. **Caduca**: cuando caduque, la revisión fallará con "Falta el secreto" o con un error 401/403, y habrá que crear uno nuevo y sustituirlo.

Si una versión nueva del oficial necesita otras versiones de Qt, libtorrent o Boost, la compilación fallará. En ese caso hay que actualizar las variables `env` de `build_ubuntu_appimage.yaml`, copiándolas de su `ci_ubuntu.yaml`.

## Pendiente / ideas acordadas

- Simplificar la interfaz (Luis dirá qué le sobra).

## Cómo juntar una versión nueva del oficial a mano

```bash
git fetch upstream --tags
git merge release-X.Y.Z     # resolver conflictos si los hay, respetando los cambios propios
# subir version.txt +1, commit y push → GitHub compila
```
