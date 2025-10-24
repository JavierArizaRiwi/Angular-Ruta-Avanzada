# 🧩 Guía de Solución – Error ENOSPC en Angular / Vite / Webpack

Este error ocurre en Linux cuando el sistema alcanza el **límite máximo de “file watchers”** (inotify).  
Estos watchers son los procesos que observan los cambios en tus archivos mientras trabajas con Angular, React, Vite, etc.

---

## ⚠️ Error común

```bash
Error: ENOSPC: System limit for number of file watchers reached
Watchpack Error (watcher): Error: ENOSPC
```

---

## 🧠 Causa

Cada proyecto Angular (o Vite, Webpack, etc.) abre **miles de watchers** para detectar cambios en tus archivos.  
Linux impone un límite bajo por defecto (~8192). Si abres varios proyectos o IDEs, se alcanza ese tope.

---

## ✅ Solución rápida (temporal)

Ejecuta los siguientes comandos en la terminal:

```bash
sudo sysctl fs.inotify.max_user_watches=524288
sudo sysctl fs.inotify.max_user_instances=1024
```

Esto aumentará los límites **hasta el próximo reinicio**.

Puedes verificar los valores actuales con:

```bash
cat /proc/sys/fs/inotify/max_user_watches
cat /proc/sys/fs/inotify/max_user_instances
```

---

## 🔒 Solución permanente

Edita el archivo `/etc/sysctl.conf` y agrega al final:

```bash
fs.inotify.max_user_watches=524288
fs.inotify.max_user_instances=1024
```

Luego aplica los cambios con:

```bash
sudo sysctl -p
```

---

## 🧰 Alternativa: usar “polling”

Si trabajas con entornos de pocos recursos o no puedes modificar el sistema:

```bash
export CHOKIDAR_USEPOLLING=1
export CHOKIDAR_INTERVAL=200
ng serve
```

Esto fuerza a Angular a revisar los archivos de forma periódica (menos eficiente, pero funcional).

---

## ⚙️ Opcional: optimizar VS Code

Reduce el consumo de watchers agregando esto en `settings.json`:

```json
"files.watcherExclude": {
  "**/node_modules/**": true,
  "**/.git/**": true,
  "**/dist/**": true
}
```

---

## 🔁 Reinicia y prueba

1. Guarda los cambios en `sysctl.conf`  
2. Ejecuta `sudo sysctl -p`  
3. Reinicia tu servidor Angular:  
   ```bash
   ng serve
   ```

El error **ENOSPC** desaparecerá ✅

---

## 🧩 Referencia

- [Documentación oficial de Angular – Watchpack Errors](https://angular.io/cli/serve)
- [Linux inotify – kernel docs](https://man7.org/linux/man-pages/man7/inotify.7.html)

---

**Autor:** Javier Ariza  
**Última actualización:** Octubre 2025  