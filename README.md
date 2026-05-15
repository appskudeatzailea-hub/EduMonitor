# Manual de Uso — EduMonitor

**Versión:** 1.0.0  
**ID de extensión:** `afpmpjemhekncdidgjpcobbibnfpelkn`  
**Repositorio:** https://github.com/akram-2008/whitelist

---

## 1. Introducción

EduMonitor es una extensión de Chrome para **control parental educativo y monitoreo de actividad** en Chromebooks. Analiza en tiempo real el contenido de cada página web que visita el estudiante y bloquea aquellas que detecta como no educativas (juegos, entretenimiento, distracciones).

La extensión funciona completamente en el navegador, sin necesidad de servidores externos ni backend.

---

## 2. Instalación desde Google Workspace

Para desplegar EduMonitor en todos los Chromebooks del centro educativo:

1. **Archivo `update.xml`:** alójalo en un servidor accesible públicamente (GitHub Pages, servidor del centro, etc.).
2. **Google Admin Console:**
   - Ve a **Dispositivos > Chrome > Aplicaciones > Extensiones > Usuarios y navegadores**.
   - Haz clic en **Añadir > Añadir desde el ID de la extensión**.
   - ID: `afpmpjemhekncdidgjpcobbibnfpelkn`
   - En **Política de instalación**: selecciona **Forzar instalación**.
   - En **URL de actualización**: introduce la URL donde alojaste `update.xml` (ej. `https://raw.githubusercontent.com/akram-2008/whitelist/refs/heads/main/update.xml`).
3. **Permisos:** la extensión necesita permisos sobre `<all_urls>` para analizar el contenido. Se aceptan automáticamente al ser forzada desde la consola de administración.

---

## 3. Arquitectura y reglas de bloqueo

### Content-First Architecture

Cuando el estudiante abre una página web, el **content script** de EduMonitor analiza en tiempo real:

- **Título y metadatos** (`<title>`, `<meta>`)
- **Texto del cuerpo** (hasta 4000 caracteres)
- **Canvas y WebGL** (frameworks de juegos como Phaser, PixiJS, Three.js)
- **Audio y vídeo** (WebAudio, elementos `<audio>`/`<video>`)
- **Clases e IDs del DOM** (palabras clave como `game`, `play`, `juego`)
- **URL y ruta** (patrones como `/game/`, `/play/`, `/learn/`)
- **Pantalla completa** activa

Con todo ello calcula tres métricas:

| Métrica | Descripción |
|---|---|
| `gameScore` | Puntuación de contenido tipo juego (0-100+) |
| `eduScore` | Puntuación de contenido educativo (0-100+) |
| `dl` | Nivel de peligro/detección (1-10) |

### Reglas de bloqueo

| Regla | Condición | Descripción |
|---|---|---|
| **A** | `gameScore >= 15` y `eduScore === 0` y `dl >= 7` | Contenido claramente de juego, sin señales educativas |
| **B** | `gameScore >= 25` y `dl >= 5` | Alta concentración de señales de juego |
| **C** | `matchedPattern` y `gameScore >= 10` | Dominio en lista de patrones conocidos |

Si alguna regla se cumple, la página se bloquea inmediatamente y se muestra la pantalla de bloqueo al estudiante.

### Patrones conocidos de juegos (preconfigurados)

```
sites.google.com
scratch.mit.edu
itch.io
newgrounds.com
armorgames.com
kongregate.com
crazygames.com
y8.com
miniclip.com
```

---

## 4. Pantalla de bloqueo

Cuando una página es bloqueada, el estudiante ve:

![Pantalla de bloqueo](Pantalla de bloqueo con información de la regla aplicada)

- **Regla aplicada** (A, B o C)
- **Reason** (scores que activaron el bloqueo)
- Botón **"Ver detalles técnicos"** — muestra gameScore, eduScore, dl y las señales detectadas
- Botón **"Solicitar desbloqueo"** — envía una solicitud al administrador
- Botón **"Volver atrás"** — navega a la página anterior

---

## 5. Solicitar desbloqueo

El estudiante puede pulsar **"Solicitar desbloqueo"** en la pantalla de bloqueo. Esto:

1. Guarda la solicitud en el almacenamiento local de Chrome.
2. La solicitud queda visible en el panel de administración (`admin.html`).
3. **Límite:** máximo 1 solicitud cada 5 minutos por URL.

El estudiante verá el mensaje **"Solicitud enviada al administrador"**.

---

## 6. Panel de administración

Accede a `admin.html` desde el menú de extensiones de Chrome o desde el popup.

### 6.1 Inicio de sesión

- Se requiere una **contraseña de administrador** (fija, preconfigurada).
- Tras 5 intentos fallidos en 1 minuto, el acceso se bloquea temporalmente.
- Una vez autenticado, la sesión se mantiene activa mientras dure el service worker.

### 6.2 Solicitudes de desbloqueo

La tabla muestra:

| Columna | Descripción |
|---|---|
| **URL** | URL que el estudiante intentó visitar |
| **Regla** | Regla que activó el bloqueo (A, B o C) |
| **Game** | gameScore en el momento del bloqueo |
| **Edu** | eduScore en el momento del bloqueo |
| **DL** | Nivel de peligro |
| **Hace** | Tiempo transcurrido desde la solicitud |
| **Acciones** | Botones para aceptar o ignorar |

**Botones de acción:**

- **"Añadir a blanca"** — añade la URL a la lista blanca local y resuelve la solicitud. El estudiante ya no volverá a ser bloqueado en esa URL.
- **"Ignorar"** — resuelve la solicitud sin añadir a la lista blanca. El estudiante podrá solicitar de nuevo tras 5 minutos.

Las solicitudes se actualizan automáticamente cada 10 segundos.

### 6.3 Lista blanca local

- Añade URLs o dominios (ej. `accounts.google.com`, `docs.google.com`).
- Las URLs en lista blanca **nunca** serán bloqueadas, independientemente de su contenido.
- Puedes eliminar entradas con el icono `×`.
- Se combina con la lista blanca remota (GitHub) para comprobar si una URL está permitida.

### 6.4 Lista blanca remota (GitHub)

- La extensión obtiene automáticamente una lista blanca desde un archivo alojado en GitHub.
- URL por defecto: `https://raw.githubusercontent.com/akram-2008/whitelist/refs/heads/main/whitelist.txt`
- Muestra el estado, número de entradas, URL de origen y última sincronización.
- Botón **"Sincronizar ahora"** para forzar una actualización.
- Se sincroniza automáticamente cada 15 minutos.

### 6.5 Historial de sincronización

Registro de las últimas 20 sincronizaciones con información de:

- Hora de sincronización
- Estado (éxito/sin cambios/error)
- Número de entradas
- Duración
- Mensaje de error (si lo hubo)

---

## 7. Lista blanca remota desde GitHub

Puedes mantener una lista blanca centralizada para todos los Chromebooks del centro.

### Formato del archivo `whitelist.txt`

```
# Lista blanca centralizada - EduMonitor
# Una entrada por línea (URL completa o dominio)
accounts.google.com
docs.google.com
drive.google.com
classroom.google.com
khanacademy.org
```

Reglas:
- Una URL o dominio por línea.
- Líneas que empiezan con `#` se tratan como comentarios.
- Máximo **200 entradas**.
- Máximo **500 caracteres** por línea.
- Solo entradas que empiecen con letra o número.

### Funcionamiento

- La extensión comprueba si hay cambios usando **ETag** (304 Not Modified).
- Se aplica **backoff exponencial** en caso de error (1 min, 5 min, 15 min, 30 min, 1 h).
- La sincronización fallida se reintenta automáticamente.
- Las listas local y remota se combinan para determinar si una URL está permitida.

---

## 8. Panel de monitoreo (`dashboard.html`)

El dashboard muestra estadísticas de uso del estudiante:

- **Tiempo de uso** dividido en educativo, distracción y neutral (hoy y total histórico).
- **Dominios más visitados** ordenados por tiempo total.
- **Gráfico diario** de los últimos 14 días (educativo vs. distracción).
- **Actividad reciente** — últimas visitas registradas.

Los datos se conservan durante **30 días** (configurable).

---

## 9. Popup (`popup.html`)

Desde el icono de la extensión en la barra de Chrome:

- Tiempo educativo de hoy
- Tiempo de distracción de hoy
- Total de bloqueos realizados
- Top 5 dominios más visitados hoy
- Acceso directo al dashboard
- Acceso directo al panel de administración
- Botón de sincronización manual de lista remota

---

## 10. Preguntas frecuentes

### ¿EduMonitor envía datos a internet?

Solo para descargar la lista blanca remota desde GitHub (si está configurada). No se envían datos del estudiante a ningún servidor.

### ¿Se puede cambiar la contraseña de administrador?

No desde la interfaz. La contraseña está fijada en el código. Para cambiarla, contacta con el desarrollador.

### ¿Qué ocurre si el estudiante desinstala la extensión?

Si está forzada desde Google Admin Console, el estudiante no podrá desinstalarla.

### ¿Funciona en incógnito?

Sí, si se configura en Google Admin Console para que se ejecute en modo incógnito.

### ¿Cuánto tiempo se guardan los datos?

30 días por defecto. Los datos antiguos se eliminan automáticamente cada día.

### ¿Puedo añadir más dominios educativos o de distracción?

Actualmente la clasificación usa dominios preconfigurados. Si necesitas personalizarlos, contacta con el desarrollador.

### Si un sitio legítimo es bloqueado por error (falso positivo):

1. El estudiante solicita desbloqueo desde la pantalla de bloqueo.
2. El administrador ve la solicitud en el panel.
3. Añade el dominio a la lista blanca local.
4. Opcionalmente, añádelo también a la lista blanca remota de GitHub para todos los Chromebooks.

---

## 11. Solución de problemas

### La extensión no bloquea páginas

- Verifica que la extensión está instalada correctamente (sin errores en `chrome://extensions`).
- Comprueba que la lista blanca remota se ha sincronizado.
- Asegúrate de que la URL no está en ninguna lista blanca.

### El panel de administración no carga

- Limpia la caché del navegador.
- Asegúrate de haber iniciado sesión como administrador.
- Abre `chrome://extensions` y recarga la extensión.

### La lista blanca remota no se actualiza

- Verifica que la URL del archivo `whitelist.txt` es accesible.
- Comprueba que el archivo tiene el formato correcto.
- Pulsa **"Sincronizar ahora"** en el panel de administración.
- Revisa el historial de sincronización para ver errores.

### No veo las solicitudes de desbloqueo

- Asegúrate de estar autenticado como administrador.
- Las solicitudes se refrescan automáticamente cada 10 segundos.
- Verifica que el service worker está activo en `chrome://extensions`.

---

*EduMonitor — Control Parental Educativo para Chromebooks*
