# Newtech Solutions — Generador de Informes & Gestor de Inventario

## Resumen
Aplicación web ligera para generación de informes técnicos (PDF) y gestión de inventario de repuestos. Diseñada para trabajar con un único archivo de datos (`DataBase.js`) que contiene tanto los informes como el inventario, permitiendo edición local, guardado directo en disco y exportación/imprenta de informes.

## Características principales
- **Generador de informes técnicos**: formulario con fecha/hora, máquina, técnico, resumen, repuestos cambiados y pendientes; vista previa imprimible y exportación a PDF.
- **Gestor de inventario**: búsqueda, filtros por caja o stock bajo, paginación, añadir/editar/eliminar ítems.
- **Persistencia en un único archivo (`DataBase.js`)**: la app lee y reescribe el objeto `MasterDB` como módulo JS exportable.
- **Guardado directo en disco**: utiliza la File System Access API para vincular y sobrescribir el archivo `DataBase.js` desde el navegador.
- **Historial de cambios**: registro de operaciones sobre inventario e informes.
- **Manejo de compatibilidad**: mensajes y rutas alternativas cuando el navegador no soporta la File System Access API.

## Estructura de datos esperada (`DataBase.js`)
La aplicación espera un módulo que exporte un objeto `MasterDB` con, al menos, las secciones `informes` e `inventario`. Ejemplo simplificado:

```js
const MasterDB = {
  informes: {
    docs: [
      {
        id: "<iso>-<random>",
        savedAt: "2025-09-13T12:34:56.789Z",
        data: {
          fecha: "YYYY-MM-DD",
          horaInicio: "HH:MM",
          horaFin: "HH:MM",
          maquina: "Máquina #",
          tecnico: "Nombre",
          resumen: "Texto...",
          cambiados: "Listado en texto",
          pendientes: "Listado en texto",
          fontSizeClass: "font-size-3",
          _cambiadosFull: [ /* objetos con item y cantidad */ ]
        }
      }
    ]
  },
  inventario: {
    lastUpdated: "2025-09-13T12:00:00.000Z",
    data: {
      "A": [ /* items */ ],
      "B": [],
      "C": [],
      "D": [],
      "E": [],
      "null": []
    }
  }
};

export default MasterDB;
```

> Nota: Mantener la estructura evita errores de lectura/escritura por la aplicación. Recomendable realizar copias de seguridad antes de cambios masivos.

## Requisitos
- Navegador moderno con soporte para **File System Access API** (ej. Chrome, Edge) para vinculación y guardado directo.
- Servir los archivos como sitio estático (GitHub Pages, `http-server`, `live-server`, etc.).
- `html2pdf.bundle.min.js` incluido para la exportación a PDF (se referencia desde `index.html`).

## Instalación (para desarrolladores)
```bash
# clona el repositorio
git clone https://github.com/strix07/generador_de-informes.git
cd generador_de-informes

# sirve los archivos con un servidor estático
npx http-server -c-1 .
```
Abrir `index.html` en un navegador compatible y usar la opción **Vincular Base de Datos (DataBase.js)** desde la interfaz.

## Flujo de uso (resumido)
1. **Vincular** `DataBase.js` desde la interfaz principal. La app cargará inventario e informes.
2. **Generar informe**: completar formulario → vista previa → `Descargar PDF` o `Guardar Datos` (añade informe y actualiza stock).
3. **Gestión de inventario**: buscar/filtrar, añadir/editar/eliminar ítems; al eliminar un informe, el stock asociado se restaura si procede.

## Consideraciones técnicas
- La app serializa el objeto `MasterDB` y sobrescribe el archivo vinculado mediante la File System Access API.
- El manejador del archivo puede persistirse en IndexedDB para evitar volver a seleccionar el archivo en cada sesión.
- Validaciones: la UI controla el stock disponible antes de restar cantidades y adapta la presentación del informe según la configuración de fuente/tamaño.


## Contribuir
1. Abrir un *issue* describiendo el bug o la mejora.
2. Crear una rama `feature/*` o `fix/*` y enviar un *pull request* con descripción y casos de prueba.
3. Si se modifica la serialización de `MasterDB`, incluir pruebas y una guía de migración.


## Contacto

adrianmayora5@gmail.com

---



