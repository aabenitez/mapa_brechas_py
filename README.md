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

Resultados:

- **Nueva Germania – Cruce 65 y Cruce 65 – Tacuatí:** Cruce 65 queda sobre el camino Nueva Germania – Tacuatí (PY11), a unos 14,6 km de Tacuatí. El primer tramo pasa de dudoso a aproximado (48,7 km frente a 40 km).
- **La Paloma – Santa Clara:** los 8,92 km arrancan en el camino a Puerto Adela (D086), no en el casco urbano.
- **San Vicente – Itanará:** el lote BID cubre 20,46 km, pero no se publicó qué parte del camino. Sigue como dudoso.
- **Con km oficiales nuevos:** Yuty – 3 de Mayo (26 km), Gral. Morínigo – Caazapá (37,93 km) y Cruce Kimex – Carlos Antonio López (57,22 km, en ejecución con BID).
- **Ubicados por primera vez:** Guajayví – Río Curuguaty'i (32 km), Nueva Esperanza – Santa Lucía (39,51 km) y Cruce Alpa – Lima (15,93 km, en ejecución, en Caazapá y no en Caaguazú).
- **Siguen sin ubicar:** Colonias Unidas, Retiro Alegre, PR-L1196 y CAF fase 3. Tampoco se halló Quiindy (San Pedro).
- **Choré – Kokuere – San Pablo:** 37,7 km según el MOPC; el trazado da 37,2 km y coincide.
- **Caazapá – Tajy – Colonia Cosme – Vizcaíno Kue:** se trazó con los puntos de Google Maps (25,1 km; el trazado da 26,5 km). En el Mapa Vial MOPC 2026 de Caazapá figura como camino vecinal. La ley 7527/2025 lo extiende hasta Fulgencio Yegros, pero ese tramo no se dibuja porque la red del INE no tiene un camino directo.

## Regenerar

```
py -I scripts/rutear.py
py -I scripts/simplificar_dptos.py
py -I scripts/generar_mapa.py
```

Los archivos de `data/` son GeoJSON estándar (EPSG:4326). Sirven igual en Leaflet, MapLibre GL JS, QGIS o kepler.gl.
