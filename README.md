# Informe Técnico • Newtech Solutions — README

> **Generador de Informe Técnico** — Interfaz web (single-file) para crear, previsualizar y exportar informes técnicos de mantenimiento en PDF. Incluye guardado local en `localStorage`, exportación a archivo `.js` y (opcional) sincronización con un archivo usando la **File System Access API**. Usa `html2pdf.bundle.min.js` para la generación del PDF.

---

## Tabla de contenidos
1. [Descripción general](#descripción-general)  
2. [Características principales](#características-principales)  
3. [Estructura del proyecto](#estructura-del-proyecto)  
4. [Dependencias](#dependencias)  
5. [Instalación y ejecución](#instalación-y-ejecución)  
6. [Cómo usar la app (flujo de usuario)](#cómo-usar-la-app-flujo-de-usuario)  
7. [Explicación del código (secciones clave)](#explicación-del-código-secciones-clave)  
8. [Almacenamiento y sincronización de DB](#almacenamiento-y-sincronización-de-db)  
9. [Generación de PDF (detalles)](#generación-de-pdf-detalles)  
10. [Validaciones y límites](#validaciones-y-límites)  
11. [Compatibilidad y notas de navegador](#compatibilidad-y-notas-de-navegador)  
12. [Mejoras sugeridas / TODO](#mejoras-sugeridas--todo)  
13. [Contribuciones y licencia](#contribuciones-y-licencia)

---

## Descripción general
Esta app es una página HTML/JS/CSS que permite rellenar un formulario de mantenimiento, previsualizar el informe en formato carta (A4/Letter) y descargarlo como PDF. Además guarda los informes en `localStorage` y permite exportar la "base de datos" en un archivo JavaScript (`newtech_reports_db_YYYY-MM-DD.js`). Opcionalmente puede **vincular** un archivo en disco (usando File System Access API) para escribir ahí la DB exportada automáticamente.

Ideal para técnicos que quieren generar informes estandarizados, rápidos y offline.

---

## Características principales
- Formulario con campos obligatorios:
  - Fecha, hora inicio/fin, máquina, técnico, resumen, repuestos cambiados y pendientes.
- Previsualización WYSIWYG en pantalla (miniatura) con formato listo para imprimir.
- Generación de PDF de alta calidad con `html2pdf` (usa `html2canvas` internamente).
- Contadores y límites por campos (líneas/ítems/caracteres).
- Selector de tamaño de letra para ajustar cuántas líneas/ítems entran en la página.
- Guardado local de informes en `localStorage`.
- Detección de **datos idénticos**: si guardas los mismos datos, sobrescribe la entrada existente en vez de duplicarla.
- Exportación manual a `.js` (archivo que define `window.NEWTECH_DB`).
- Opción de "vincular" un archivo en disco (File System Access API) para escribir la DB directamente.
- Sanitización básica del texto para evitar inyección HTML en la previsualización.

---

## Estructura del proyecto
Archivo principal (todo en uno):
- `index.html` (el HTML que enviaste) — contiene:
  - `<style>`: estilos y variables CSS (tema oscuro UI, estilos de página blanca para PDF).
  - Formulario y vista previa (markup).
  - `<script>`: objeto `App` con toda la lógica JS.

Importante: el proyecto requiere que `html2pdf.bundle.min.js` esté disponible (referenciado desde la carpeta raíz en el `<head>`).

---

## Dependencias
- `html2pdf.bundle.min.js` — para convertir el DOM en PDF. (Incluye html2canvas y jsPDF internamente).
- Opcional pero recomendado: navegador basado en Chromium (Chrome, Edge, Opera) para soporte File System Access API.

---

## Instalación y ejecución
1. Clona o copia el archivo `index.html` y pon `html2pdf.bundle.min.js` en la misma carpeta.
2. Para evitar restricciones CORS y permitir el uso de File System Access API en entorno de desarrollo, **sirve** el archivo con un servidor HTTP simple (no `file://`):
   ```bash
   # Python 3
   python -m http.server 8000
   # luego abre: http://localhost:8000
   ```
3. Abre la página en Chrome/Edge/Opera (recomendado). Algunas funcionalidades (vincular archivo) requieren File System Access API.

---

## Cómo usar la app (flujo de usuario)
1. Rellenas la fecha, hora de inicio/fin, máquina, técnico y el resto de campos.
2. Mientras escribes, la previsualización se actualiza en tiempo real.
3. Cambia el tamaño de letra con el botón **Tamaño de letra** si necesitas más/menos contenido en la plantilla.
4. Para guardar los datos localmente: pulsa **Guardar Datos**. La app:
   - Valida campos obligatorios.
   - Crea o sobrescribe una entrada en la DB guardada en `localStorage`.
   - Si hay un archivo vinculado, también escribe la DB en ese archivo.
5. Para descargar el PDF: pulsa **Descargar PDF**. La app:
   - Valida campos.
   - Forza la vista de página al 100% y llama a `html2pdf` para generar y descargar el PDF con nombre `Informe_Mantenimiento_YYYY-MM-DD_<maquina>.pdf`.
6. Para vincular un archivo en disco (opcional): pulsa **Vincular DB** y elige dónde quieres guardar el `.js`. A partir de entonces, las exportaciones automáticas usan ese handle.

---

## Explicación del código (secciones clave)
El código principal está dentro del objeto `App`. Aquí se resumen sus partes más importantes:

### `elements`
Referencias a DOM: inputs del formulario, elementos de salida (preview), botones, contadores y la página a imprimir.

### `limits` y `fontClasses`
Define límites y cómo cambian según tamaño de fuente. Por ejemplo:
- `resumenMaxLines` y `listMaxItems` cambian según la clase `.font-size-X` aplicada a `.page`.

### `init()`
- `setDefaultDateTime()`: inicializa fecha y horas.
- Añade clase de fuente por defecto (`font-size-3`).
- Recupera el handle de archivo (si está guardado en IndexedDB) y actualiza el estatus DB.
- Vincula eventos y renderiza la preview inicial.

### `bindEvents()`
Registra listeners:
- `input` del formulario → `renderPreview()`.
- Limitadores para listas y resumen (impiden exceder líneas permitidas).
- Botones: imprimir, guardar, limpiar, tamaño de letra, vincular DB, exportar DB.

### `renderPreview()` / `updateCounters()`
- Toma datos del formulario, formatea la fecha, sanitiza datos y los inserta en la vista previa.
- `formatList()` convierte texto con saltos de línea a arrays cortos y truncados.
- Actualiza contadores (líneas, ítems, caracteres permitidos).

### Validación: `validate()`
- Verifica que campos obligatorios (fecha, horas, máquina, técnico, resumen) existan. Si faltan, alerta al usuario.

### IndexedDB helpers: `idbPut()` / `idbGet()`
- Guarda y recupera un **handle** de archivo (File System Access API) en IndexedDB para persistir la referencia al archivo vinculado. (No confundir con la DB de informes, que está en `localStorage`).

### File System Access: `askUserForDBFileHandle()` / `writeDBToHandle()` / `ensureAndWriteDB()`
- `askUserForDBFileHandle()` abre el diálogo `showSaveFilePicker()` para que el usuario elija un archivo donde guardar la DB.
- `writeDBToHandle()` escribe la representación JS de la base de datos en el archivo.
- `ensureAndWriteDB()` intenta escribir la DB si existe el handle; si falla, limpia el handle.

> **Nota**: `showSaveFilePicker()` sólo está disponible en navegadores compatibles (Chromium-based). El código comprueba disponibilidad y muestra mensajes claros.

### DB interna: `loadDatabase()` / `saveDatabase()` / `exportDatabaseAsJS()`
- La "DB" de informes se guarda en `localStorage` bajo `newtech_reports_db`. Estructura:
  ```json
  { "meta": { "version": 1 }, "docs": [ { "savedAt": "2025-08-24T12:34:56.789Z", "data": { ... } } ] }
  ```
- `exportDatabaseAsJS()` genera y descarga un archivo `.js` con `window.NEWTECH_DB = { ... }` (útil para transportar o versionar offline).

### Guardado inteligente: `saveDataToDB()`
- Valida formulario.
- Crea `currentData`.
- Compara `JSON.stringify(currentData)` con las entradas existentes para decidir si sobrescribir o agregar (evita duplicados idénticos).
- Guarda `savedAt` (timestamp ISO) y `data`.
- Sincroniza con archivo vinculado si existe.

### Generación de PDF: `generatePdf()`
- Comprueba que `html2pdf` esté cargado.
- Valida formulario.
- Construye nombre de archivo seguro (reemplaza caracteres no alfanuméricos).
- Forza la clase `.is-generating-pdf` en el elemento de página para quitar la escala de miniatura y obtener una captura al 100%.
- Usa `html2pdf().set(options).from(element).save()`.
- Opciones: `html2canvas.scale = 2` para buena resolución + `jsPDF` en formato `letter` (se puede cambiar a `a4`).
- Restaura la vista miniatura y estado del botón.

### Utilidades
- `sanitize`: evita que texto crudo se inyecte como HTML en la vista previa.
- `truncateInputsToLimits`: hace recortes para ajustarse a límites tras cambiar tamaño de fuente.
- `clearForm`: limpia el formulario (con confirmación), preserva fecha/hora por defecto.

---

## Almacenamiento y sincronización de DB
- **localStorage**: la DB real de informes está en `localStorage` bajo la clave `newtech_reports_db`. Estructura:
  ```json
  { "meta": { "version": 1 }, "docs": [ { "savedAt": "2025-08-24T12:34:56.789Z", "data": { ... } } ] }
  ```
- **Exportación manual**: genera un `.js` descargable que define `window.NEWTECH_DB`.
- **Vinculación de archivo**: si el navegador soporta File System Access API, puedes vincular un archivo `.js`. La app guarda una referencia (handle) en IndexedDB para reutilizarla y escribir directamente en el archivo cada vez que se guarda la DB.

---

## Generación de PDF (detalles técnicos)
- La app usa `html2pdf.bundle.min.js` (que combina `html2canvas` y `jsPDF`).
- Para evitar que la miniatura reduzca la calidad de captura, se:
  1. Añade `.is-generating-pdf` a la `.page` para restaurar escala 100%.
  2. Espera ~50 ms para que el estilo se aplique.
  3. Llama a `html2pdf` con `html2canvas.scale = 2` para mejorar resolución.
- Nombre de archivo: `Informe_Mantenimiento_<fecha>_<maquinaSafe>.pdf`.
- Si necesitas A4: cambiar `jsPDF.format` a `'a4'`.

---

## Validaciones y límites
- Campos obligatorios: Fecha, hora inicio/fin, máquina, técnico y resumen.
- Máquina: máximo 30 caracteres.
- Resumen: límite de líneas según tamaño de fuente (configurable).
- Repuestos (cambiados/pendientes): máximo de ítems según tamaño de fuente, y límite por ítem (`limits.listMaxChars = 60`).
- Al llegar al límite de ítems o líneas, la app trunca automáticamente el contenido.

---

## Compatibilidad y notas de navegador
- La app funciona en la mayoría de navegadores modernos para edición, previsualización y generación de PDF **cuando `html2pdf.bundle.min.js` está cargado**.
- **File System Access API** (vincular archivo) está disponible principalmente en Chromium (Chrome, Edge, Opera). Safari y Firefox no la soportan — la app detecta la ausencia y muestra `DB: API no soportada`.
- Recomendación: usar Chrome/Edge en entorno de escritorio. Para pruebas locales, sirve la carpeta mediante un servidor local (p. ej. `python -m http.server`).

---

## Limitaciones conocidas
- La exportación a archivo requiere permisos del navegador; si la escritura falla el handle se borra para evitar referencias rotas.
- `html2pdf`/`html2canvas` pueden producir ligeras diferencias visuales entre navegadores; siempre revisar el PDF generado.
- No hay autenticación ni cifrado de la DB local: `localStorage` es legible por cualquier código en el origen.

---

## Mejoras sugeridas / TODO
- Añadir IDs / hashes únicos para cada documento guardado (en vez de comparar solo por `JSON.stringify`) para historial más robusto.
- Implementar vista/lista de documentos guardados con posibilidad de edición/duplicado/eliminación.
- Añadir soporte multi-página (si resumen largo) y paginación automática en el PDF.
- Añadir plantilla de firma del técnico (imagen) y campos personalizados.
- Añadir pruebas unitarias y automatizadas.
- Soporte para A4/EU / carta según idioma/región seleccionado.

---

## Ejemplos rápidos
### Ejecutar localmente
```bash
# en la carpeta donde está index.html y html2pdf.bundle.min.js
python -m http.server 8000
# abrir: http://localhost:8000
```

### Cambiar formato a A4 (en generatePdf)
```js
const options = {
  // ...
  jsPDF: { unit: 'in', format: 'a4', orientation: 'portrait' }
};
```

### Forzar que la DB solo guarde nuevos (no sobrescribir)
Reemplaza la lógica de `saveDataToDB()` que busca por igualdad completa, por una que siempre inserte:
```js
db.docs.push({ savedAt: now, data: currentData });
```

---

## Contribuciones
Si quieres colaborar:
1. Haz fork del repo.
2. Abre un issue describiendo la mejora.
3. Envía un PR con cambios documentados.
- Sigue las convenciones: HTML/CSS/JS limpios; añade comentarios y tests manuales.

