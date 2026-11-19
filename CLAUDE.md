# qBittorrent de Luis (fork para Ubuntu)

Fork personal del qBittorrent oficial con cambios propios, compilado como AppImage para Ubuntu.

- Fork: https://github.com/luisp193/qBittorrent (rama `master`)
- Oficial (`upstream`): https://github.com/qbittorrent/qBittorrent
- Copia local: `C:\Users\luis.perez\Documents\IA-PROGRAMACION\qBittorrent`
- Remotos git: `origin` = fork, `upstream` = oficial

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

### Detalle de "BORRADAS" (V2)

Código en `src/gui/transferlistwidget.cpp`: `isTorrentDeleted()`, `moveDeletedTorrents()`, `checkDeletedAllTorrents()`, `checkDeletedSelectedTorrents()`; temporizador en el constructor de `TransferListWidget`.

- Una descarga cuenta como borrada si **ninguno** de los archivos que ya tenía descargados (progreso > 0) existe en el disco. Si se borra solo una parte, no se marca.
- No se marca si no existe ni la carpeta de destino (p. ej. disco externo desconectado), ni si está sin metadatos, comprobándose o moviéndose.
- Al marcarla: se crea la categoría `BORRADAS` si no existe, se desactiva la gestión automática (para que no mueva archivos), se pone la categoría, se **detiene** y se anota en el Registro.
- "Comprobar borrado" hace lo mismo con las seleccionadas y muestra un mensaje con el resultado.

## Pendiente / ideas acordadas

- **Revisión mensual automática** de versiones nuevas del oficial: juntar la última versión estable (tags `release-x.y.z`) con `master`, subir `version.txt` y compilar; si hay conflicto, abrir un issue para resolverlo a mano. Necesita un token de GitHub con permiso `workflow` (lo crea Luis).
- Simplificar la interfaz (Luis dirá qué le sobra).

## Cómo juntar una versión nueva del oficial a mano

```bash
git fetch upstream --tags
git merge release-X.Y.Z     # resolver conflictos si los hay, respetando los cambios propios
# subir version.txt +1, commit y push → GitHub compila
```
