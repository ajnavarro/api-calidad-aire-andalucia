# API Calidad del Aire en Andalucía

## Introducción

Este repositorio contiene todos los datos puestos a disposición por la Junta de Andalucía sobre la Calidad del Aire en formato JSON.

No se encuentran disponibles absolutamente todos los días desde 2004 por diversas razones:

- En algunos casos la página con los datos no es generada. [Ejemplo](http://www.juntadeandalucia.es/medioambiente/atmosfera/informes_siva/sep19/gr190926.htm). 
- Muchas veces el reporte existe, pero algunas estaciones de calidad del aire no reportan todos o algunos datos. [Ejemplo](http://www.juntadeandalucia.es/medioambiente/atmosfera/informes_siva/oct19/gr191006.htm).
- A veces el formato o la información en el reporte cambia o es errónea. [Ejemplo](http://www.juntadeandalucia.es/medioambiente/atmosfera/informes_siva/ago06/al060801.htm).

Si detectas que es posible procesar más casos de uso, puedes realizar la petición en el apartado de [issues](https://github.com/ajnavarro/api-calidad-aire-andalucia/issues)

## Como usar la API

Por una parte, puedes descargar este repositorio e ir haciendo `git pull` para obtener los nuevos datos cada día.

Por otra parte, los datos son servidos como si de una API REST se tratase para hacer más fácil su uso.
La URL sería la siguiente:

https://ajnavarro.com/api-calidad-aire-andalucia/v0/[PROVINCIA]/[AÑO]/[MES]/[DIA].json

Ejemplo:

https://ajnavarro.com/api-calidad-aire-andalucia/v0/granada/2008/1/2.json

## Formato de los datos

A continuación describimos la estructura de los datos:

### Tipos de estado

Para saber que significan exactamente los diferentes tipos de estado, puedes leerlo [aquí](http://www.juntadeandalucia.es/medioambiente/site/portalweb/menuitem.7e1cf46ddf59bb227a9ebe205510e1ca/?vgnextoid=7e612e07c3dc4010VgnVCM1000000624e50aRCRD&vgnextchannel=910f230af77e4310VgnVCM1000001325e50aRCRD).

| Valor | Significado | Descripción |
|-------|-------------|-------------|
| 0     | Vacío       | El campo viene sin ningún tipo de valor. Suele significar que la estación no es capaz de hacer ese tipo de mediciones. |
| 1     | Sin datos   | No existen datos para ese día específico. |
| 2     | Bueno             | Calidad del aire marcada como "buena". |
| 3     | Aceptable             | Calidad del aire marcada como "aceptable". |
| 4     | Mala             | Calidad del aire marcada como "mala". |
| 5     | Muy Mala             | Calidad del aire marcada como "muy mala". |

### Tipos de evolución

El campo `evol` nos indica como ha cambiado la calidad del aire de un día para el otro:

| Valor | Significado | Descripción |
|-------|-------------|-------------|
| 0     | Indefinida  | La evolución no se indica. Suele ser cause de que el día anterior la estación o los datos fallasen. |
| 1     | Mejor       | La calidad del aire ha ido a mejor. |
| 2     | Peor        | La calidad del aire ha ido a peor. |
| 3     | Igual       | La calidad del aire sigue igual. |

### Estructura de los datos

```go
type AirQuality struct {
	URL      string `json:"u"` // URL de donde se han obtenido los datos. Ejemplo: http://www.juntadeandalucia.es/medioambiente/atmosfera/informes_siva/dic19/gr191201.htm
	Time     int64  `json:"time"` // Timestamp indicando el día que se generaron los datos.
	Stations []*struct { // lista de estaciones de medición de calidad del aire.
		Town      string `json:"t"` // Ciudad donde se encuentra la estación.
		Station   string `json:"s"` // Nombre de la estación.
		SO2       Status `json:"so2"` // Estado de la cantidad de SO2 en el aire.
		CO        Status `json:"co"` // Estado de la cantidad de CO en el aire.
		NO2       Status `json:"no2"` // Estado de la cantidad de NO2 en el aire.
		Particles Status `json:"part"` // Estado de la cantidad de particulas en el aire.
		O3        Status `json:"o3"` // Estado de la cantidad de Ozono en el aire.
		Global    Status `json:"g"` // Estado del aire de forma global, teniendo en cuenta el total de contaminantes.

		PreviousSituation Status    `json:"prev"` // Estado previo de la calidad del aire.
		ActualSituation   Status    `json:"act"` // Estado actual de la calidad del aire.
		Evolution         Evolution `json:"evol"` // Evolución de la calidad del aire de un día para otro.

		GranularStation *struct { // Este apartado contiene los datos numéricos obtenidos por los sensores en ese día.
			Province string `json:"p"` // Provincia.
			Town     string `json:"t"` // Ciudad.
			Station  string `json:"s"` // Nombre de la estación.
			Address  string `json:"addr"` // Dirección donde se encuentra la estación.
			Data     []*struct { // Datos obtenidos en el día especificado en fracciones de 10 minutos.
				Time      int64 `json:"time"` // Timestamp de la medición.
				SO2       int   `json:"so2"` // Cantidad de S02 en ug/m3.
				Particles int   `json:"part"` // Cantidad de partículas en ug/m3.
				NO2       int   `json:"no2"` // Cantidad de NO2 en ug/m3.
				CO        int   `json:"co"` // Cantidad de CO en ug/m3.
				O3        int   `json:"o3"`// Cantidad de O3 en ug/m3.
			} `json:"gsd"`
		} `json:"gs"`
	} `json:"stations"`
}

```

## TODO list
 
 - Automatizar proceso de generación de nuevos archivos JSON cada día.
 - Liberar código del parser.
 - Implementar más formatos como CSV.
 - Añadir datos agrupados por semana, mes, año.
