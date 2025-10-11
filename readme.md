# Newtech Solutions - Suite de Gestión Técnica

Una aplicación web *offline-first* diseñada para la gestión integral de operaciones técnicas. La suite permite generar informes, administrar un inventario de repuestos y llevar un historial detallado de mantenimientos preventivos, todo operando sobre un único archivo de base de datos local (`DataBase.js`).

## Resumen

Esta suite de herramientas está construida con Vanilla JavaScript y aprovecha la **File System Access API** para ofrecer una experiencia de escritorio directamente en el navegador. Permite a los técnicos de campo y al personal de mantenimiento interactuar con una base de datos centralizada (`DataBase.js`) para leer y escribir datos sin necesidad de un backend o conexión a internet, garantizando que toda la información crítica esté siempre disponible y actualizada en un único archivo portable.

El proyecto se divide en tres módulos principales:

1.  **Generador de Informes Técnicos**: Para documentar mantenimientos correctivos e incidentes.
2.  **Gestor de Inventario**: Para el control de existencias de repuestos y consumibles.
3.  **Historial de Mantenimientos Preventivos**: Para registrar y consultar tareas de mantenimiento rutinarias según un checklist predefinido.

---

## Características Principales

### 📋 Generador de Informes Técnicos
- **Formulario Dinámico**: Rellena datos clave como máquina, técnico, fechas, horas y resumen de actividades.
- **Integración con Inventario**: Busca y descuenta repuestos del stock de `DataBase.js` al generar un informe.
- **Exportación a PDF**: Genera un documento PDF profesional con un diseño limpio, listo para ser impreso o enviado.
- **Ajuste de Formato**: Permite cambiar el tamaño de la fuente del informe para ajustar la cantidad de contenido por página.
- **Búsqueda y Visualización**: Busca informes históricos y previsualízalos antes de descargarlos.

### 📦 Gestor de Inventario
- **CRUD Completo**: Añade, edita y elimina ítems del inventario de forma intuitiva.
- **Búsqueda y Filtros Avanzados**: Encuentra ítems rápidamente por nombre, código, ubicación o cantidad. Filtra por "caja" o visualiza solo los ítems con bajo stock.
- **Paginación Eficiente**: Maneja grandes inventarios sin sacrificar el rendimiento.
- **Actualización en Tiempo Real**: Los cambios se guardan directamente en `DataBase.js`, reflejándose de inmediato en el generador de informes.
- **Historial de Cambios**: Registra cada modificación para una auditoría sencilla (funcionalidad en desarrollo).

### 🛠️ Historial de Mantenimientos Preventivos
- **Checklists Predefinidos**: El sistema sugiere tareas específicas basadas en la máquina y el tipo de mantenimiento (semanal, mensual, trimestral).
- **Registro Sencillo**: Crea nuevos registros de mantenimiento seleccionando tareas de una lista y añadiendo observaciones.
- **Filtros por Máquina**: Visualiza el historial completo de mantenimientos para un equipo específico con un solo clic.
- **Consulta Rápida**: Accede a un modal con el checklist completo de todas las máquinas para referencia rápida.
- **Base de Conocimiento Centralizada**: Consolida todos los registros de mantenimiento preventivo en la misma base de datos que los informes y el inventario.

---

## ¿Cómo Funciona?

La aplicación utiliza la **File System Access API** de los navegadores modernos (Chrome, Edge) para solicitar permiso de acceso a un archivo local (`DataBase.js`). Una vez vinculado, la aplicación puede leer su contenido y, lo más importante, **sobrescribirlo directamente en el disco** cada vez que se realiza un cambio.

Esto crea un flujo de trabajo "sin guardado", donde todas las modificaciones (actualizar stock, añadir un informe, registrar un mantenimiento) se persisten automáticamente en el archivo. Para mejorar la experiencia, el manejador del archivo se guarda en **IndexedDB**, por lo que solo necesitas vincular la base de datos una vez por sesión.

## Estructura de Datos (`DataBase.js`)

La aplicación espera que `DataBase.js` exporte un objeto `MasterDB` con la siguiente estructura:

```javascript
const MasterDB = {
  // Módulo de Informes Técnicos
  informes: {
    docs: [
      {
        id: "<timestamp>-<random>",
        savedAt: "2025-10-11T12:34:56.789Z",
        data: {
          fecha: "YYYY-MM-DD",
          maquina: "Máquina #1",
          tecnico: "Nombre del Técnico",
          resumen: "Descripción de las actividades...",
          _cambiadosFull: [ /* Array de objetos de repuestos descontados */ ]
        }
      }
    ]
  },
  // Módulo de Inventario
  inventario: {
    lastUpdated: "2025-10-11T12:00:00.000Z",
    data: {
      "A": [ /* Array de ítems en la caja A */ ],
      "B": [ /* ... */ ]
    }
  },
  // Módulo de Mantenimientos Preventivos
  mantenimientos: {
    docs: [
        {
            id: "<timestamp>-<random>",
            savedAt: "2025-10-11T20:08:50.415Z",
            data: {
                tecnico: "Nombre del Técnico",
                fecha: "2025-10-11",
                maquina: "M-10924",
                tipo: "Semanal",
                tareas: ["Limpiar carcasa y ranuras de ventilación"],
                observaciones: "Sin novedad."
            }
        }
    ]
  }
};

export default MasterDB;
```
> **Importante**: Es crucial mantener esta estructura para evitar errores de lectura/escritura. Se recomienda realizar copias de seguridad de `DataBase.js` periódicamente.

## Requisitos y Uso

### Requisitos
- Un navegador moderno compatible con la **File System Access API** (Google Chrome, Microsoft Edge son recomendados).
- Servir los archivos desde un servidor web local. No funcionará abriendo los archivos `index.html` directamente desde el disco (`file://`).

### Instalación y Ejecución
1.  Clona el repositorio:
    ```bash
    git clone https://github.com/strix07/generador_de-informes.git
    cd generador_de-informes
    ```
2.  Inicia un servidor web estático en la raíz del proyecto. Una forma sencilla es usando `http-server` de Node.js:
    ```bash
    # Instala http-server si no lo tienes
    npm install -g http-server

    # Inicia el servidor
    http-server -c-1 .
    ```3.  Abre la URL proporcionada (generalmente `http://127.0.0.1:8080`) en tu navegador.
4.  Desde la interfaz principal, haz clic en **Vincular Base de Datos (DataBase.js)** y selecciona tu archivo `DataBase.js`.
5.  Una vez vinculado, la aplicación cargará los datos y estará lista para usarse.


## Contacto
Adrián Mayora – adrianmayora5@gmail.com