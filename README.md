# PAT-A · Plataforma de Apoyo Técnico de La Araucanía

Conjunto de herramientas web para formuladores de proyectos del **Sistema Nacional de Inversiones (SNI) de Chile**, orientado a la Región de La Araucanía.

**Autor:** Cristóbal Oyarzún Parra  
**Arquitectura:** Frontend estático (HTML + JavaScript vanilla) + Supabase como backend  
**Sin servidor propio:** toda la lógica corre en el navegador

---

## Herramientas disponibles

| Archivo | Herramienta | Descripción |
|---|---|---|
| `index.html` | Inicio | Hub de acceso a todas las herramientas |
| `censo2024_consulta.html` | Población, Vivienda y Hogares | Consulta CENSO 2024 + RSH con mapa interactivo |
| `asistente_reevaluaciones.html` | Asistente de Reevaluaciones | Determina si corresponde solicitar reevaluación SNI |
| `moneda_bip.html` | Convertidor de Moneda BIP | Actualiza montos entre años según IPC INE |
| `reajuste_obras.html` | Reajuste de Obras | Reajuste de contratos de obra pública |
| `rsh_importador.html` | Importador RSH | Importa CSV de BIDAT → Supabase |
| `rsh_convertidor.html` | Convertidor RSH | Convierte CSV BIDAT a formato Supabase |
| `uv_importar_2.html` | Importador UV | Importa GeoJSON de Unidades Vecinales → Supabase |

---

## Base de datos — Supabase

**Proyecto:** `dsrvxgpdxzjcpqtopfrk.supabase.co`  
**URL REST:** `https://dsrvxgpdxzjcpqtopfrk.supabase.co/rest/v1`  
**API Key (anon/pública):** `sb_publishable_jsz5WVNHq33JNXqQqKY9UQ_b53dB0kk`

---

### Vistas del CENSO 2024

Cada nivel territorial tiene una vista de totales y otra desagregada por área (`URBANO` / `RURAL`).

| Vista | Nivel | Filtro principal |
|---|---|---|
| `vw_region` | Región | `cod_region` |
| `vw_region_area` | Región × área | `cod_region`, `area` |
| `vw_provincia` | Provincia | `cod_region`, `provincia` |
| `vw_provincia_area` | Provincia × área | `cod_region`, `provincia`, `area` |
| `vw_comuna` | Comuna | `cut_comuna` |
| `vw_comuna_area` | Comuna × área | `cut_comuna`, `area` |
| `vw_distrito` | Distrito censal | `id_distrito` |
| `vw_area` | Distrito × área | `id_distrito`, `area` |
| `vw_zona` | Zona urbana | `id_zona` |
| `vw_localidad` | Localidad rural | `id_distrito`, `id_localidad`, `area` |
| `vw_entidad` | Entidad rural | `id_entidad` |

#### Columnas de métricas (todas las vistas)

| Campo | Descripción |
|---|---|
| `n_per` | Total de personas |
| `n_hombres` | Hombres |
| `n_mujeres` | Mujeres |
| `n_edad_0_5` | Personas de 0 a 5 años |
| `n_edad_6_13` | Personas de 6 a 13 años |
| `n_edad_14_17` | Personas de 14 a 17 años |
| `n_edad_18_24` | Personas de 18 a 24 años |
| `n_edad_25_44` | Personas de 25 a 44 años |
| `n_edad_45_59` | Personas de 45 a 59 años |
| `n_edad_60_mas` | Personas de 60 años o más |
| `n_pueblos_orig` | Personas pertenecientes a pueblos originarios |
| `n_hog` | Hogares |
| `n_vp` | Viviendas particulares |
| `area` | `URBANO` / `RURAL` (solo en vistas `_area`) |

---

### Tablas RSH — Registro Social de Hogares

#### `rsh_personas` y `rsh_hogares`

Datos a nivel de Unidad Vecinal (UV), importados desde BIDAT (Ministerio de Desarrollo Social y Familia).

| Campo | Tipo | Descripción |
|---|---|---|
| `cut_comuna` | integer | Código único territorial de la comuna |
| `cod_uv` | integer | Código de Unidad Vecinal (BIDAT) |
| `periodo` | text | Período de corte (`YYYYMM`) |
| `total` | integer | Total RSH en la UV |
| `total_comuna` | integer | Total RSH en la comuna |
| `total_region` | integer | Total RSH en la región |
| `tramo_0_40` | integer | Personas/hogares en tramo CSE 0%–40% |
| `tramo_41_50` | integer | Personas/hogares en tramo CSE 41%–50% |
| `tramo_51_60` | integer | Personas/hogares en tramo CSE 51%–60% |
| `tramo_61_70` | integer | Personas/hogares en tramo CSE 61%–70% |
| `tramo_71_80` | integer | Personas/hogares en tramo CSE 71%–80% |
| `tramo_81_90` | integer | Personas/hogares en tramo CSE 81%–90% |
| `tramo_91_100` | integer | Personas/hogares en tramo CSE 91%–100% |

#### `rsh_agregados`

Datos consolidados a nivel regional, provincial y comunal.

| Campo | Tipo | Descripción |
|---|---|---|
| `nivel` | text | `region`, `provincia` o `comuna` |
| `codigo` | text | Código del nivel (`cod_region`, nombre de provincia o `cut_comuna`) |
| `tipo` | text | `personas` o `hogares` |
| `total` | integer | Total RSH |
| `tramo_0_40` … `tramo_91_100` | integer | Por tramo CSE |
| `periodo` | text | Período de corte (`YYYYMM`) |

#### `rsh_config`

Tabla de configuración interna (clave-valor).

| Campo | Descripción |
|---|---|
| `clave` | Nombre del parámetro (ej. `admin_hash`) |
| `valor` | Valor almacenado (ej. hash SHA-256 de contraseña) |

---

### Tabla `uv_chile` — Unidades Vecinales

Geometría y metadatos de aproximadamente 6.800 Unidades Vecinales de todo Chile.

| Campo | Descripción |
|---|---|
| `cod_uv_bidat` | Código UV usado por BIDAT (llave de cruce con RSH) |
| `cut_comuna` | Código único territorial de la comuna |
| `geometry` | Polígono geográfico (PostGIS) |

#### Funciones RPC

| Función | Parámetros | Retorno |
|---|---|---|
| `uv_from_point` | `lat`, `lng` | UV que contiene el punto |
| `get_uv_geojson` | `cut_comuna`, `cod_uvs[]` | GeoJSON de las UVs solicitadas |

---

## APIs externas

### ArcGIS FeatureServer — Límites territoriales CENSO 2024

**Base:** `https://services5.arcgis.com/hUyD8u3TeZLKPe4T/ArcGIS/rest/services/Censo2024_v2/FeatureServer`

| Layer | Nivel territorial |
|---|---|
| 3 | Regiones |
| 4 | Provincias |
| 11 | Comunas |
| 10 | Distritos censales |
| 2 | Zonas censales / Entidades |
| 7 | Localidades rurales |
| 9 | Entidades rurales |

Se usa para renderizar los polígonos de cada área en el mapa Leaflet.

### Leaflet.js 1.9.4

Librería de mapas interactivos. Tiles base: CartoDB Voyager (OpenStreetMap).

---

## Datos embebidos en el frontend

### IPC BIP (`moneda_bip.html`, `asistente_reevaluaciones.html`)

Objeto `IPC_BIP` con índices de precios de **98 años** (1928–2027), hardcodeados en JavaScript.

```js
const IPC_BIP = {
  1928: 4.84878e-07,
  // ...
  2024: 163.432,
  2025: 170.297,
  2026: 175.576,
  2027: 175.576   // proyectado
};
```

**Fórmula de conversión:**

```
monto_convertido = monto_original × (IPC_BIP[año_destino] / IPC_BIP[año_origen])
```

La tabla es administrable desde la misma herramienta con contraseña (hash almacenado en `rsh_config`).

---

### Causales de Reevaluación (`asistente_reevaluaciones.html`)

12 causales definidas en el **punto 3.2 de las NIP 2026** (Módulo Ex Dure):

| Causal | Referencia NIP | Descripción | Umbral |
|---|---|---|---|
| A | 3.2.1.1 | Modificaciones previas a la licitación | 20% del monto recomendado |
| B | 3.2.1.2 | Modificaciones por proceso de licitación | — |
| C | 3.2.1.3 | Modificaciones de contratos vigentes | 10% del contrato |
| D | 3.2.1.4 | Término anticipado de contratos | 10% del contrato |
| E | 3.2.1.5 | Licitación o ejecución por fases constructivas | — |
| F | 3.2.1.6 | Programación de inversión superior a 120 meses | — |
| G | 3.2.1.7 | Ingreso o modificación de asignación/fuente financiera | — |
| H | 3.2.1.8 | Cambio de magnitud en etapa de Diseño | 20% del presupuesto diseño |
| I | 3.2.1.9 | Verificación de hito de control | — |
| J | 3.2.1.10 | Ajuste a solicitud de financiamiento | — |
| K | 3.2.1.11 | Solicitud expresa del MDSF | — |
| L | 3.2.1.12 | Otra situación no tipificada | — |

---

## Variables de estado globales (`censo2024_consulta.html`)

| Variable | Tipo | Descripción |
|---|---|---|
| `selectedAreas` | `Array` | Áreas territoriales seleccionadas para comparación múltiple |
| `areaDataCache` | `Array` | Datos cacheados de cada área seleccionada |
| `currentNavLevel` | `string \| null` | Nivel activo del navegador (`region`, `provincia`, `comuna`, `distrito`, `zona`, `localidad`, `entidad`) |
| `combinedSums` | `Object` | Sumas acumuladas `{total, urbano, rural}` para múltiples áreas |
| `selectedUVs` | `Array` | UVs seleccionadas en el panel RSH |
| `rshCurrentUV` | `Object \| null` | UV activa en el panel RSH (`{cut_comuna, cod_uv_bidat, ...}`) |
| `currentPanels` | `Array` | Paneles de datos renderizados actualmente |
| `_sbCache` | `Map` | Cache en memoria de respuestas Supabase (por URL) |
| `boundaryLayer` | `Leaflet Layer` | Polígono territorial activo en el mapa |
| `map` | `Leaflet Map` | Instancia del mapa Leaflet |
| `_reqToken` | `number` | Token para cancelar requests anteriores y evitar race conditions |

---

## Flujo de datos típico

```
Usuario selecciona región/provincia/comuna
        ↓
onRegionChange() / onComunaChange() / etc.
        ↓
sb("vw_region", "select=*&cod_region=eq.9")   →  REST GET Supabase
        ↓
renderPanels([{key:"total"}, {key:"urbano"}, {key:"rural"}])
        ↓
updateMap()  →  ArcGIS FeatureServer (polígono SVG sobre Leaflet)
        ↓
renderRSHPanel()  →  sb("rsh_agregados", ...)
```

El helper `sb(view, params)` centraliza todas las llamadas a Supabase con caché en memoria para vistas estáticas (`vw_*`, `rsh_agregados`, `uv_chile`, `rsh_personas`, `rsh_hogares`).

---

## Niveles territoriales y jerarquía

```
Chile
 └── Región (cod_region)
      └── Provincia (provincia)
           └── Comuna (cut_comuna)
                ├── Distrito censal (id_distrito)
                │    ├── Zona urbana (id_zona)
                │    └── Localidad rural (id_localidad)
                │         └── Entidad rural (id_entidad)
                └── Unidad Vecinal — UV (cod_uv) [solo RSH]
```

La herramienta permite **selección múltiple** de áreas en cualquier nivel y suma sus métricas automáticamente.

---

## Exportación de datos

| Herramienta | Formatos disponibles |
|---|---|
| Censo 2024 | Excel (`.xlsx`), PDF |
| Convertidor BIP | CSV (UTF-8 con BOM) |
| Importador RSH | — (solo importa hacia Supabase) |

---

## Normativa de referencia

- **NIP 2026** — Normas, Instrucciones y Procedimientos del SNI, Ministerio de Desarrollo Social y Familia
  - Punto 3.2: Reevaluación de Iniciativas de Inversión (Módulo Ex Dure)
  - Punto 3.2.5: Exclusiones por cambio de naturaleza
- **CENSO de Población y Vivienda 2024** — INE Chile
- **RSH** — Registro Social de Hogares, BIDAT, MDSF
- **IPC** — Índice de Precios al Consumidor, INE Chile (serie 1928–2027)

---

## Descargo de responsabilidad

> La información entregada es de carácter referencial. La validación de los antecedentes para su uso en procesos de inversión pública es responsabilidad del formulador.
