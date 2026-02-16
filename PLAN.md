# Plan: Mapa Conceptual Interactivo del PAE

## Objetivo
Crear una vista principal tipo "mapa conceptual" donde el usuario navega desde lo más amplio (categorías) hasta lo más específico (lineamiento individual), haciendo clic progresivamente. Incluye filtros que actualizan todo dinámicamente.

---

## Jerarquía de Navegación (4 niveles)

```
Nivel 0: Vista panorámica → 10 CATEGORÍAS como burbujas grandes
    ↓ clic en una categoría
Nivel 1: Subcategorías → Las subcategorías de esa categoría (3-6 nodos)
    ↓ clic en una subcategoría
Nivel 2: Lineamientos → Lista de lineamientos de esa subcategoría (cards)
    ↓ clic en un lineamiento
Nivel 3: Detalle → Panel con texto completo, verificadores, conexiones, indicadores
```

---

## Pasos de Implementación

### Paso 1: Estructura HTML del Mapa Conceptual
- Agregar la nueva sección `sec-mapa` como primera sección activa en el HTML
- Agregar el enlace en el sidebar como primer ítem (con icono de mapa)
- Componentes del HTML:
  - **Barra de filtros** (top): 4 dropdowns (Documento, Etapa, Actor, Riesgo)
  - **Breadcrumb** de navegación: PAE > [Categoría] > [Subcategoría] > [Lineamiento]
  - **Área del mapa** (SVG D3.js): donde viven las burbujas
  - **Panel lateral de detalle** (derecho): aparece al seleccionar un lineamiento
- Mover "Resumen General" al segundo lugar del sidebar

### Paso 2: CSS del Mapa Conceptual
- Estilos para la barra de filtros superior
- Estilos del breadcrumb (clicable, con flechas separadoras)
- Estilos del contenedor SVG del mapa (ocupa ~70% del ancho)
- Estilos del panel lateral de detalle (~30% derecho, aparece/desaparece con animación)
- Estilos para las burbujas/nodos (hover glow, transiciones de tamaño)
- Estilos para las cards de lineamientos (nivel 2)
- Animaciones de entrada/salida entre niveles (fade + scale)

### Paso 3: Nivel 0 - Vista de Categorías (D3 Bubble Pack)
- Usar D3.js `d3.pack()` para crear un bubble chart con las 10 categorías
- Cada burbuja:
  - Tamaño proporcional al número de lineamientos (después de filtros)
  - Color único por categoría (paleta de 10 colores)
  - Muestra: nombre de la categoría + count de lineamientos
  - Hover: tooltip con descripción completa
- Los counts se recalculan cuando cambian los filtros
- Al hacer **clic** en una burbuja → transición animada al Nivel 1

### Paso 4: Nivel 1 - Subcategorías (expansión visual)
- Al hacer clic en una categoría:
  - El breadcrumb se actualiza: **PAE > [Nombre Categoría]**
  - La burbuja seleccionada se "abre" mostrando sus subcategorías como burbujas internas
  - Las demás categorías se encogen y se mueven a los bordes (o se ocultan con fade)
  - Cada subcategoría muestra: nombre + count de lineamientos
  - Hover: descripción de la subcategoría
- Botón "Volver" en el breadcrumb para regresar al Nivel 0
- Al hacer **clic** en una subcategoría → transición al Nivel 2

### Paso 5: Nivel 2 - Lineamientos (grid de cards)
- Al hacer clic en una subcategoría:
  - El breadcrumb: **PAE > [Categoría] > [Subcategoría]**
  - Las burbujas se desvanecen y aparece un grid de cards con los lineamientos
  - Cada card muestra:
    - ID del lineamiento
    - Texto (truncado a ~100 chars)
    - Badges: etapa, riesgo (con color), obligatoriedad, actor
    - Documento fuente
  - Las cards respetan los filtros activos
  - Ordenamiento por riesgo (crítico primero)
- Al hacer **clic** en una card → aparece el panel de detalle (Nivel 3)

### Paso 6: Nivel 3 - Panel de Detalle del Lineamiento
- Panel lateral derecho que se desliza desde la derecha
- Contenido del panel:
  - **Encabezado**: ID + badges de riesgo/etapa/obligatoriedad
  - **Texto completo** del lineamiento
  - **Metadata**: documento, sección fuente, actor responsable, actor verificador
  - **Verificadores** asociados (de la hoja VERIFICADORES via lin_id):
    - Descripción, tipo, instrumento, frecuencia, criterio de cumplimiento
  - **Conexiones** (de CONEXIONES donde lin_id_origen o lin_id_destino coincide):
    - Tipo de conexión + lineamiento relacionado (clicable)
  - **Indicadores** relacionados (de INDICADORES via subcat_id compartido):
    - Nombre, fórmula, meta, periodicidad
  - Botón "Cerrar" (X)
- Si haces clic en un lineamiento conectado → el panel se actualiza con ese lineamiento

### Paso 7: Filtros Dinámicos
- 4 filtros en la barra superior:
  - **Documento**: dropdown con los 12 documentos
  - **Etapa PAE**: dropdown (planeación, operación, transversal, seguimiento, evaluación)
  - **Actor Responsable**: dropdown con los 16 actores
  - **Riesgo**: dropdown (crítico, alto, medio, bajo)
- Comportamiento:
  - Al cambiar cualquier filtro → se filtran los LINEAMIENTOS
  - Los counts en las burbujas de categorías/subcategorías se recalculan
  - Las burbujas con 0 lineamientos se muestran en gris/deshabilitadas
  - Si estás en Nivel 2 (cards), las cards se filtran en tiempo real
  - Botón "Limpiar filtros" para resetear todo
- Indicador visual: "Mostrando X de 395 lineamientos"

### Paso 8: Integración con el Dashboard Existente
- El Mapa Conceptual es la sección **por defecto** (active al cargar)
- Las demás secciones (Resumen, Documentos, etc.) siguen accesibles via sidebar
- La función `switchSection()` existente maneja la navegación
- La Red de Conexiones (sección 5) sigue con su grafo D3 separado
- El Mapa Conceptual tiene su propio inicializador `initMapa()`

### Paso 9: Testing y Refinamiento
- Verificar transiciones entre los 4 niveles
- Verificar que los filtros recalculan correctamente
- Verificar responsividad en móvil (panel de detalle se pone full-width)
- Verificar que los breadcrumbs funcionan para navegar hacia atrás

### Paso 10: Commit y Push
- Commit con mensaje descriptivo
- Push al branch `claude/interactive-data-visualization-QvxYZ`

---

## Tecnologías
- **D3.js v7**: bubble pack layout para Niveles 0 y 1
- **CSS transitions/animations**: para las transiciones entre niveles
- **HTML/JS vanilla**: para cards, panel de detalle, filtros
- **Chart.js**: solo para las demás secciones (no para el mapa)

## Paleta de Colores por Categoría
| Categoría | Color |
|-----------|-------|
| NOR (Normativo) | #1a365d |
| DIA (Diagnóstico) | #2b6cb0 |
| CON (Contractual) | #3182ce |
| ALI (Alimentario) | #38b2ac |
| CPL (Compras) | #319795 |
| CAL (Calidad) | #805ad5 |
| FIN (Financiero) | #d69e2e |
| GOB (Gobernanza) | #dd6b20 |
| EVA (Evaluación) | #e53e3e |
| TRA (Transversal) | #2f855a |
