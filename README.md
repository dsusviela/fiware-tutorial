# intro

Esto es parte de los [tutoriales de fiware](https://github.com/FIWARE/tutorials.Time-Series-Data), con solo una explicacion basica de lo que esta pasando y de como levantar los servicios.

# arqui

Basicamente el tutorial implementa la siguiente arquitectura

```
Grafana
  ^
  |
  v
Crate
  ^
  |
QuantumLeap
  ^
  |
  v
Orion
  ^
  |
  v
IoT agent
  ^
  |
Simualdor en la web
```

# levantando los serivicios

Para correr una demo de los servicios, se deben seguir los siguientes pasos:

1. Correr el script "services" con el siguiente comando `./services start`.
2. Suscribir a Orion, el servicio de quantumleap, se puede hacer mediante el sigueinte request:

POST:
```
http://localhost:1026/v2/subscriptions/
```
HEADER
```
{
  Content-Type: application/json,
  fiware-service: openiot,
  fiware-servicepath: /
}
```
BODY
```
{
  "description": "Notify QuantumLeap on luminosity changes on any Lamp",
  "subject": {
    "entities": [
      {
        "idPattern": "Lamp.*"
      }
    ],
    "condition": {
      "attrs": [
        "luminosity",
        "location"
      ]
    }
  },
  "notification": {
    "http": {
      "url": "http://quantumleap:8668/v2/notify"
    },
    "attrs": [
      "luminosity", "location"
    ],
    "metadata": ["dateCreated", "dateModified"]
  },
  "throttling": 1
}
```

Listo.

# visualizando los datos

Primero hay que generar datos, para eso ir a `http://localhost:3000` y prender alguna lamparita. Para visualizar los datos se usa Grafana.

1. Ir a `localhost:3002/login` user: admin, pass: admin
2. Ir a los ajustes -> datasource y seleccionar postgre. `http://localhost:3000/datasources`
3. Llenar con los siguientes datos:
```
Name CrateDB
Host crate-db:5432
Database mtopeniot
User crate
SSL Mode disable
```
4. En el lateral derecho crear un dashboard. `http://localhost:3003/dashboard/new?orgId=1`
5. Clickear en agregar query
6. Llenar con los siguientes datos:
```
Queries to CrateDB (the previously created Data Source)
FROM etlamp
Time column time_index
Metric column entity_id
Select value column:luminosity
```
7. Si esta todo ok, presionar la tecla esc
8. En la esquina superior derecha seleccionar `Add panel` y seleccionar agregar mapa
9. En map layout poner los siguientes datos:
```
Center: custom
Latitude: 52.5031
Longitude: 13.4447
Initial Zoom: 12
```
10. Tocar el boton a la derecha de queries y llenar los sigueintes datos:
```
Format as: Table
FROM etlamp
Time column time_index
Metric column entity_id
Select value
    column:luminosity alias:value
    column:location alias:geojson
    column:entity_type alias:type
```
11. Volver a la seleccion de visualizacion a la izquierda y en la seccion de capas rellnar con los sigueitnes datos:
```
Map Layers:
    Lamp:
        Icon: lightbulb-o
        ClusterType: average
        ColorType: fix
        Column for value: value
        Maker color: red

```
