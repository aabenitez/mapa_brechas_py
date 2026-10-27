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

Para abrir el mapa con las capas opcionales ya activas, agregar `#capas=ilum,nse` al final de la dirección.

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

## Regenerar

```
py -I scripts/rutear.py
py -I scripts/simplificar_dptos.py
py -I scripts/generar_mapa.py
```

Los archivos de `data/` son GeoJSON estándar (EPSG:4326). Sirven igual en Leaflet, MapLibre GL JS, QGIS o kepler.gl.
