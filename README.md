# Wiki NOC - MSGW AS400 / JDE Edwards

Wiki funcional autocontenida (HTML + CSS + JS, sin dependencias externas) para documentar los errores **MSGW (Message Waiting)** que genera AS400 / JDE Edwards en el día a día del NOC.

## Contenido

- Errores de archivo / operación (RNX1221, RNX0114, RNX1021, RNX0122, RNX1216, MCH1210)
- Stream file / entidades (MSG0008)
- Errores de fecha / hora
- Errores de array / índice
- Errores de acceso / IFS (CPF4131, CPF4103, CPFA0B1)
- Backups (CPF377F)
- Jobs de chequeo / batch
- Procedimientos especiales (SNDF$AR_P1 / Pinot)

## Características

- **Buscador en vivo** sobre todos los campos (código, título, mensaje, causa, programas, archivos, resolución, casos).
- **Filtros por categoría**.
- **Navegación lateral** con scroll suave y resaltado temporal.
- **Crear / editar / eliminar entradas propias** desde un modal (se guardan en `localStorage` del navegador).
- **Severidad** con código de color (info / warning / critical).
- **Tags** para programas y archivos involucrados.
- **Casos documentados** por entrada (fecha, ticket, job, acción tomada).
- Imprimible, responsive y 100% offline.

## Cómo usarla

1. Descargar `index.html`.
2. Abrirla en cualquier navegador moderno (Chrome, Edge, Firefox).
3. Para agregar una nueva entrada, tocar **"+ Nueva entrada"** (arriba a la derecha) o el botón flotante.
4. Las entradas que cree el operador quedan guardadas en el navegador; si quiere migrarlas a otra máquina, hay que exportar `localStorage` (próximamente).

## Origen de la información

Casos reales documentados en los mails del NOC (Hugo Magariños - Semantix/Atos). Cuando un caso requiere análisis, la acción es **llamar a Ricardo Caldeiro**. Para backups de fin de mes, los contactos son Omar Baldomir / Leonardo Cavalieri.

## Mantenimiento

- Las entradas base viven embebidas en el `index.html` (objeto `BASE_ENTRIES`).
- Las entradas creadas por el operador se persisten en `localStorage` bajo la clave `msgw_wiki_custom_entries_v1`.
- El script generador original está en `/scripts/build_msgw_wiki.py` del entorno de desarrollo.
