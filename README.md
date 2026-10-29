# Mapa de tramos de caminos vecinales

Abrir `mapa_tramos.html` con doble clic. Necesita internet para el mapa base y las capas del INE.

## Qué datos ofrece cada geoportal

**INE, portal geoestadístico (GeoNode + GeoServer).** Es el que sirve.

| Capa | Nombre en GeoServer | Contenido |
|---|---|---|
| Red vial 2022 | `geonode:V_as_Paraguay_INE_20221` | 391.538 segmentos. Atributos: departamento, distrito, área urbana/rural, nombre, tipo de vía (Camino Rural, Ruta Nacional…) y calzada (Tierra, Asfalto, Ripio, Empedrado…). |
| Vías principales | `geonode:Vias_Principales_2022` | Rutas nacionales y departamentales. |
| Localidades (centroides) | `geonode:barloc_centroide_id2` | 8.002 barrios y localidades. |
| Departamentos | `geonode:Departamentos_Paraguay_INE_2022` | Límites. |
| Puestos de peaje | `geonode:Puestos_de_Peajes` | 19 puntos. El ícono es `way.svg` del GeoServer y se guarda en `data/iconos/`. |
| Iluminación nocturna | `geonode:iluminacion2` | Ráster con 8 clases de luminosidad. Solo se puede ver por WMS. |
| Nivel socioeconómico | `geonode:_<dpto>___urbano` / `___rural` | Polígonos por barrio o localidad, Censo 2022. Campo `nivsoecdes`: Alto, Medio alto, Medio, Medio bajo, Bajo. Es la misma capa del zip de Guairá. |

La iluminación nocturna arranca activa. Para abrir el mapa con el nivel socioeconómico también activo, agregar `#capas=nse` al final de la dirección.

- Descarga en GeoJSON (WFS):
  `https://geonode.ine.gov.py/geoserver/ows?service=WFS&version=2.0.0&request=GetFeature&typeNames=CAPA&outputFormat=application/json&srsName=EPSG:4326`
- Filtrar con `&CQL_FILTER=DesRodaVia='Tierra' AND AREA<>'1'`.
- Recortar con `&bbox=minLon,minLat,maxLon,maxLat,EPSG:4326` (versión 1.1.0).

Según esta capa, Paraguay tiene unos 141.000 km de caminos rurales de tierra.

**MOPC, sgeMap (MapServer).** Permite descargar en GeoJSON, pero solo la geometría, sin atributos: no hay nombre, tipo de calzada ni estado.

- Endpoint: `https://www.mopc.gov.py/cgi-bin/mapserv?map=/var/www/html/sgeMap/mopc00.map&service=WFS&version=1.0.0&request=GetFeature&typeName=tramos_lineas&outputFormat=geojson`
- Capas: `tramos_lineas` (106 tramos, sin nombre) y `Vias` (416.622 segmentos).
- `Vias` va en `mopcVIAS.map`. En GeoJSON el servidor corta la descarga con un error, así que conviene pedirla en GML.

## Cómo se dibujaron los tramos

`scripts/rutear.py` traza cada tramo como la ruta más corta sobre la red vial del INE entre sus localidades extremas. Los extremos se definen en `scripts/tramos_def.py` y los km obtenidos se comparan con los oficiales:

- **coincide:** diferencia de hasta 15 %.
- **aproximado:** diferencia de hasta 40 %.
- **dudoso:** diferencia mayor. Se dibuja con línea punteada.

En los tramos que empiezan en una ruta («Ruta 10 – …»), el punto de inicio se ajustó para igualar los km oficiales.

Cuando un extremo no figura en el INE (Cruce 65, el río Curuguaty'i), se ubicó sobre la red con los km oficiales. En esos tramos la calidad dice «extremo ubicado con los km oficiales», porque el largo ya no sirve para validar el trazado.

## Verificación de tramos (octubre 2026)

Se revisaron los tramos dudosos, los aproximados, los que no tenían km oficial y los que no estaban ubicados. Fuentes:

- MOPC: Balance Anual de Gestión Pública 2021 y notas de prensa en mopc.gov.py.
- Cartera de Proyectos del MOPC, publicada en informacionpublica.paraguay.gov.py.
- Patrulla Caminera, Agencia IP, ABC Color, Última Hora y La Nación.
- Llamado BID LPI 29/2018 (PR-L1092).
- Mapas MOPC de San Pedro, Canindeyú, Caazapá, Caaguazú e Itapúa: Inventario Vial (corte dic-2025) y Obras en Ejecución 2026. Se georreferenciaron con las localidades y se superpusieron los tramos.

Resultados:

- **Nueva Germania – Cruce 65 y Cruce 65 – Tacuatí:** Cruce 65 queda sobre el camino Nueva Germania – Tacuatí (PY11), a unos 14,6 km de Tacuatí. El primer tramo pasa de dudoso a aproximado (48,7 km frente a 40 km).
- **La Paloma – Santa Clara:** los 8,92 km arrancan en el camino a Puerto Adela (D086), no en el casco urbano.
- **San Vicente – Itanará:** los 20,46 km del lote BID (Ecomipa, inaugurado en febrero de 2023) van por la D005 desde San Vicente Pancholo hacia Villa Ygatimí, no hasta Itanará. Lo indican la red del INE, que tiene asfaltada la D005 en ese tramo, la imagen satelital y la nota de inauguración, que ubica un puente del lote en el tramo San Vicente – Villa Ygatimí. Se dibuja hasta donde termina el asfalto. El punto final oficial no se publicó. El «Itanará» del nombre sería la localidad Arroyo Itamará, sobre la D005 (Inventario MOPC 2025); desde Villa Ygatimí hay otro lote en ejecución en 2026 hacia Arroyo Itanará (20,84 km).
- **Con km oficiales nuevos:** Yuty – 3 de Mayo (26 km), Gral. Morínigo – Caazapá (37,93 km) y Cruce Kimex – Carlos Antonio López (57,22 km, en ejecución con BID).
- **Ubicados por primera vez:** Guajayví – Río Curuguaty'i (32 km), Nueva Esperanza – Santa Lucía (39,51 km) y Cruce Alpa – Lima (15,93 km, en ejecución, en Caazapá y no en Caaguazú).
- **Lima – Quiindy:** Quiindy se ubicó con el Inventario MOPC 2025, unos 4 km antes de Naranjito. Lima – Quiindy da 25,0 km (oficial 24,10) y coincide.
- **Nueva Esperanza – Santa Lucía:** Santa Lucía es una localidad de Itakyry (Alto Paraná). El tramo va por Laurel y la D087 (34,4 km frente a 39,51 km); antes estaba dibujado por error hacia Ybyrarovana.
- **Gral. Morínigo – Caazapá:** se redibujó como en el mapa MOPC de obras 2026: desde la PY18, unos 7 km al sureste del casco de Morínigo, hasta Caazapá (38,9 km frente a 37,93 km).
- **Cruce Alpa – Lima:** según el mismo mapa, Cruce Alpa está en el límite con Itapúa y la obra sigue la D046 por Col. Lima; se dibuja hasta completar los 15,93 km oficiales.
- **Abaí – Gral. Morínigo:** obra en ejecución que figura en el mapa MOPC (14,9 km, Edivi). Se agregó al mapa con la ruta más corta del INE (16,5 km).
- **Caaguazú:** PY13 – Tarumai – San Joaquín y Santa Rosa del Mbutuy – San Joaquín siguen caminos pavimentados del Inventario MOPC 2025; el primero se extendió hasta el empalme con la ruta nacional (17,0 km frente a 19,04 km). Se agregaron dos obras en ejecución del mapa MOPC 2026: Santo Domingo – Escuela Agrícola – Sidepar – PY13 (26,2 km frente a 27,13 km) y la Ruta de la Banana por la D043 hasta Tembiaporã (30,3 km, sin km oficial). Colonias Unidas tampoco figura en esos mapas.
- **Itapúa:** según el mapa MOPC de obras 2026, Cruce Kimex – Carlos Antonio López es la pavimentación de la PY18 desde el empalme con la PY06, en el Cruce Kimex de San Rafael del Paraná; antes se dibujaba desde otra localidad del INE con el mismo nombre, 6 km al noreste (56,2 km frente a 57,22 km). Se agregó la obra en ejecución Naranjito – Nueva Aurora – Col. Santa Inés (27,5 km); Col. Santa Inés no figura en el INE y se dibuja hasta Nueva Aurora (19,9 km).
- **Siguen sin ubicar:** Colonias Unidas, Retiro Alegre, PR-L1196 y CAF fase 3.
- **Choré – Kokuere – San Pablo:** 37,7 km según el MOPC; el trazado da 37,2 km y coincide.
- **Caazapá – Tajy – Colonia Cosme – Vizcaíno Kue:** se trazó con los puntos de Google Maps (25,1 km; el trazado da 26,5 km). En el Mapa Vial MOPC 2026 de Caazapá figura como camino vecinal. La ley 7527/2025 lo extiende hasta Fulgencio Yegros, pero ese tramo no se dibuja porque la red del INE no tiene un camino directo.

## Regenerar

```
py -I scripts/rutear.py
py -I scripts/simplificar_dptos.py
py -I scripts/generar_mapa.py
```

Los archivos de `data/` son GeoJSON estándar (EPSG:4326). Sirven igual en Leaflet, MapLibre GL JS, QGIS o kepler.gl.
