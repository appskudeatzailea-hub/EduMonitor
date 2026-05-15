# Guía de Desarrollo — EduMonitor

Cómo editar el código, empaquetar la extensión y publicar actualizaciones **manteniendo el mismo ID** para que Google Workspace siga funcionando.

---

## 1. El ID de la extensión

El ID de una extensión de Chrome **no está en el código**. Se genera a partir de la **clave privada** (archivo `.pem`) que se usa al empaquetar la extensión por primera vez.

- Si usas **la misma clave `.pem`** cada vez que empaquetas → el ID **siempre será el mismo**.
- Si pierdes la clave `.pem` o usas una diferente → obtendrás un **ID nuevo** y la política de Google Workspace dejará de funcionar.

**El ID actual es:** `afpmpjemhekncdidgjpcobbibnfpelkn`

---

## 2. Requisitos previos

- Google Chrome / Chromium instalado
- La extensión en modo desarrollador (`chrome://extensions` > activar **Modo desarrollador**)
- Acceso a la clave `.pem` original (sin ella **no puedes mantener el mismo ID**)

---

## 3. Estructura del proyecto

```
EduMonitor - Kopia/
├── manifest.json          # Configuración de la extensión (versión, permisos, etc.)
├── background.js          # Service worker (procesos en segundo plano)
├── content.js             # Script de contenido (análisis de páginas)
├── shared.js              # Utilidades compartidas
├── admin.html             # Panel de administración
├── admin.js               # Lógica del panel de administración
├── dashboard.html         # Panel de monitoreo
├── dashboard.js           # Lógica del dashboard
├── popup.html             # Menú emergente
├── popup.js               # Lógica del popup
├── styles.css             # Estilos
├── chart.min.js           # Librería de gráficos
├── icons/                 # Iconos de la extensión
│   ├── icon16.png
│   ├── icon48.png
│   └── icon128.png
├── update.xml             # Archivo de actualización para Google Workspace
├── MANUAL.md              # Manual de uso
└── BUILD.md               # Esta guía
```

---

## 4. Editar el código

### 4.1 Modificar archivos existentes

Edita directamente los archivos con tu editor de preferencia (VS Code, Sublime, etc.).

**Reglas importantes:**
- **No cambies el `manifest_version`** (debe ser `3` para Chrome actual).
- **Si cambias la versión** en `manifest.json`, refleja ese cambio también en `update.xml`.
- **No muevas archivos de carpeta** sin actualizar las rutas en `manifest.json`.
- Después de editar, recarga la extensión en `chrome://extensions` para ver los cambios.

### 4.2 Versiones (cómo actualizar)

Cada vez que publiques una nueva versión:

1. Incrementa el número de versión en `manifest.json`:
   ```json
   "version": "1.0.1"
   ```
2. Actualiza `update.xml` con la nueva versión:
   ```xml
   <updatecheck codebase='https://github.com/akram-2008/whitelist/releases/download/EduMonitor/EduMonitor.crx' version='1.0.1' />
   ```

### 4.3 Probar cambios localmente

1. Ve a `chrome://extensions`.
2. Activa **Modo desarrollador** (esquina superior derecha).
3. Haz clic en **"Cargar descomprimida"**.
4. Selecciona la carpeta del proyecto.
5. Después de cada cambio, pulsa el icono **↻ (Recargar)** en la tarjeta de la extensión.

---

## 5. Empaquetar como .crx

### 5.1 Primera vez (genera la clave .pem)

```powershell
# En Chrome, ve a chrome://extensions > Modo desarrollador
# Haz clic en "Empaquetar extensión"
# Raíz de la extensión: selecciona la carpeta del proyecto
# Clave privada: déjalo en blanco (se generará una nueva)
```

Chrome generará:
- `EduMonitor.crx` — el archivo empaquetado
- `EduMonitor.pem` — **LA CLAVE PRIVADA. GUÁRDALA EN UN LUGAR SEGURO.**

### 5.2 Actualizaciones (manteniendo el mismo ID)

```powershell
# chrome://extensions > Modo desarrollador > Empaquetar extensión
# Raíz de la extensión: selecciona la carpeta del proyecto
# Clave privada: selecciona el archivo EduMonitor.pem original
```

El `.crx` generado tendrá el **mismo ID** que la versión anterior.

### 5.3 Alternativa: línea de comandos (Chrome/Chromium)

```powershell
# Con Chrome instalado en la ubicación típica:
& "C:\Program Files\Google\Chrome\Application\chrome.exe" --pack-extension="C:\ruta\al\proyecto" --pack-extension-key="C:\ruta\EduMonitor.pem"
```

---

## 6. Publicar una actualización

### 6.1 Subir el .crx a GitHub Releases

1. Ve a tu repositorio: https://github.com/akram-2008/whitelist
2. Crea un nuevo **Release**:
   - Tag: `EduMonitor-v1.0.1`
   - Título: `EduMonitor v1.0.1`
   - Adjunta el archivo `EduMonitor.crx`
3. La URL del .crx será:
   ```
   https://github.com/akram-2008/whitelist/releases/download/EduMonitor-v1.0.1/EduMonitor.crx
   ```
   o si siempre usas el mismo nombre de archivo:
   ```
   https://github.com/akram-2008/whitelist/releases/download/EduMonitor/EduMonitor.crx
   ```

### 6.2 Actualizar update.xml

Edita `update.xml` para reflejar la nueva versión y la URL del .crx:

```xml
<?xml version='1.0' encoding='UTF-8'?>
<gupdate xmlns='http://www.google.com/update2/response' protocol='2.0'>
  <app appid='afpmpjemhekncdidgjpcobbibnfpelkn'>
    <updatecheck codebase='https://github.com/akram-2008/whitelist/releases/download/EduMonitor/EduMonitor.crx' version='1.0.1' />
  </app>
</gupdate>
```

### 6.3 Subir update.xml

El `update.xml` debe estar accesible públicamente. Opciones:
- **GitHub Pages:** súbelo a la rama `gh-pages` de tu repositorio.
- **Servidor del centro:** súbelo a un servidor web accesible.
- **GitHub Releases:** también puedes subirlo como adjunto en el Release.

En Google Admin Console, la URL de `update.xml` es la que configuraste al forzar la instalación.

---

## 7. Flujo completo de actualización

```
1. Editas el código (background.js, content.js, etc.)
2. Incrementas "version" en manifest.json
3. Pruebas localmente (Cargar descomprimida)
4. Empaquetas .crx usando la MISMA clave .pem
5. Actualizas update.xml con nueva versión y URL
6. Subes .crx a GitHub Releases
7. Subes update.xml al servidor público
8. Chrome detecta la actualización automáticamente
   (Chrome comprueba update.xml periódicamente)
```

---

## 8. Backup de la clave .pem

**Sin el archivo `.pem` original, NO puedes mantener el mismo ID.**

Recomendaciones:
- Guarda `EduMonitor.pem` en un gestor de contraseñas (Bitwarden, 1Password).
- Sube una copia cifrada a un lugar seguro.
- Si trabajas en equipo, comparte la clave de forma segura (nunca por email sin encriptar).
- **No incluyas el `.pem` en el repositorio público de GitHub.**

---

## 9. Resolución de problemas

### El ID cambió después de empaquetar

→ Usaste una clave `.pem` diferente o nueva. Recupera la clave original y vuelve a empaquetar.

### Google Workspace no actualiza la extensión

- Verifica que `update.xml` es accesible públicamente (abre la URL en el navegador).
- Comprueba que el `appid` en `update.xml` coincide con el ID actual de la extensión.
- Comprueba que la `version` en `update.xml` es mayor que la instalada.
- Chrome comprueba actualizaciones cada ~5 horas. Puedes forzarlo en `chrome://extensions` > **Actualizar**.

### Error "Invalid extension" al empaquetar

- Revisa que `manifest.json` tiene el formato JSON correcto (sin comas finales, sin comentarios).
- Asegúrate de que todas las rutas a los archivos existen (iconos, scripts, etc.).

### La extensión no carga después de una actualización

- Revisa los logs de errores en `chrome://extensions` > tarjeta de EduMonitor > **Errores**.
- Abre `chrome://extensions` y haz clic en **Actualizar**.
- Si el error persiste, desinstala y vuelve a forzar la instalación desde Google Admin Console.

---

## 10. Referencia

- [Chrome Extensions Documentation](https://developer.chrome.com/docs/extensions/)
- [Google Workspace Admin Help](https://support.google.com/chrome/a/answer/6306504)
- [Repositorio de listas blancas](https://github.com/akram-2008/whitelist)
- [ID de extensiones de Chrome](https://developer.chrome.com/docs/extensions/mv3/manifest/key/)
