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
- Ubuntu `ubuntu-latest` (desde noviembre de 2026 es **26.04**; antes era 24.04), Qt 6.6.3, libtorrent 2.0.15, Boost 1.77, con interfaz gráfica (`GUI=ON`).
- Publica una release `V<n>` con:
  - `qBittorrent_V<n>-x86_64.AppImage` (con versión)
  - `qBittorrent-x86_64.AppImage` (nombre fijo). Enlace estable:
    https://github.com/luisp193/qBittorrent/releases/latest/download/qBittorrent-x86_64.AppImage
- El nombre que se ve en el menú de Ubuntu es `qBittorrent_V<n>` (el workflow cambia `Name=` del `.desktop`).
- El AppImage necesita **Ubuntu 26.04 o posterior** (desde V4): al compilarse en 26.04 lleva una `libglib` que pide GLIBC 2.43, y en 24.04 no arranca. Luis usa Ubuntu 26.04.1 y decidió dejarlo así. Si algún día hace falta que funcione en 24.04, poner `runs-on: ubuntu-24.04` en `build_ubuntu_appimage.yaml`. Si da error de FUSE: `sudo apt install libfuse2t64`.
- La web https://luis-gloria.com/ (repo `luisp193/luisp193.github.io`) ofrece la descarga con el enlace estable y dice "Ubuntu 26.04 o superior": si cambia el requisito, cambiarlo también allí.
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
| V4 | Más ligero para ver películas con VLC: prioridad baja de disco y CPU, E/S de disco "simple pread/pwrite" por defecto y **modo película** (tortuga automática con VLC abierto) | `src/app/application.cpp`, `src/base/bittorrent/sessionimpl.cpp` |
| V5 | BORRADAS pasa de categoría a **estado "Borradas"** en el panel Estado (la descarga conserva su categoría) | `src/base/torrentfilter.cpp/.h`, `src/gui/transferlistfilters/statusfilterwidget.cpp/.h`, `src/gui/transferlistfilters/tagfilterproxymodel.cpp/.h`, `src/gui/transferlistwidget.cpp/.h` |
| V6 | Una descarga en estado "Archivos faltantes" (tras reiniciar con los archivos borrados el progreso vuelve a 0) también cuenta como borrada si no existe ningún archivo | `src/gui/transferlistwidget.cpp` |

### Detalle de "más ligero / modo película" (V4)

- Al arrancar (solo Linux), antes de crear los hilos de libtorrent: `nice 10` y E/S de disco en clase **idle** (`ioprio_set`). qBittorrent solo usa el disco cuando VLC u otro programa no lo necesita. Funciones `applyLowProcessPriority()` en `application.cpp`.
- Tipo de E/S de disco por defecto = `SimplePreadPwrite` en vez de `Default` (mmap). Evita los picos de memoria y de escritura a disco de libtorrent 2.0 que congelan el sistema. Se puede cambiar en Opciones → Avanzado → "Tipo de E/S de disco".
- **Modo película**: cada 5 s mira en `/proc/*/comm` si hay un proceso `vlc`. Si lo hay, activa los límites de velocidad alternativos (la tortuga, por defecto 10 KiB/s) y lo anota en el Registro; al cerrar VLC los quita (solo si los puso él). Al salir de qBittorrent también los quita. Funciones `isVlcRunning()`, `startVlcWatcher()`, `restoreVlcWatcherSpeedLimit()`.

### Detalle de "Borradas" (V2, estado desde V5)

Código en `src/gui/transferlistwidget.cpp`: `isTorrentDeleted()`, `hasFilesAgain()`, `migrateOldDeletedCategory()`, `moveDeletedTorrents()`, `restoreUndeletedTorrents()`, `checkDeletedAllTorrents()`, `checkDeletedSelectedTorrents()`; temporizador en el constructor de `TransferListWidget`.

- Una descarga cuenta como borrada si **ninguno** de los archivos que ya tenía descargados (progreso > 0, o todos si está en estado "Archivos faltantes") existe en el disco. Si se borra solo una parte, no se marca.
- No se marca si no existe ni la carpeta de destino (p. ej. disco externo desconectado), ni si está sin metadatos, comprobándose o moviéndose.
- Al marcarla: se **detiene**, se le pone la etiqueta interna `BORRADAS` (`TorrentFilter::DeletedTag`) y se anota en el Registro. **No se le cambia la categoría.**
- **Estado "Borradas"**: `TorrentFilter::Deleted` (último antes de `_Count`), con contador en el panel Estado (`statusfilterwidget.cpp`). Las borradas **solo salen en "Todos" y en "Borradas"**: `matchStatus()` las excluye de los demás estados (Completados, Detenido, etc.).
- La etiqueta interna no se ve en el panel Etiquetas (`TagFilterProxyModel::filterAcceptsRow`) ni en el submenú Etiquetas, y "Quitar todas" las etiquetas la conserva. Sí aparece en la columna Etiquetas.
- "Comprobar borrado" (botón derecho) marca las seleccionadas que estén borradas y **saca de Borradas** las que vuelvan a tener algún archivo en el disco (siguen detenidas), y muestra un mensaje con el resultado.
- Migración: hasta V4 se usaba la **categoría** `BORRADAS`. En la revisión (a los 2 min y cada 24 h) las descargas de esa categoría pasan al estado Borradas (sin categoría) y la categoría se elimina.

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
