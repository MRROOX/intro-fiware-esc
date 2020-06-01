# intro-fiware-esc

# Tutorial

## Diagrama de Arquitectura
![](docs/images/)

## Diagrama de Secuencia

![](docs/images/)

## Listado de Componentes

| Servicio      | Puertos            |
| ------------- | -------------------|
| IOT-AGENT     |   7896, 4041       |
| ORION         |  1026              |
| QUANTUMLEAF   | 8668               |  
| GRAFANA       |  3000              |  
| MONGODB       | 27017              |  
| CRATEDB       |  5432, 4300, 4200  |

## Recursos 

| URL           | Descripción                          |
| ------------- | -------------------------------------|
| /             |   Dashboard generado en grafana      |
| /ORION         |  1026                                |


# LOCALHOST
## 1. Desplegar servicios definidos en docker-compose.yml

```
docker-compose up -d
```
## 2. Registro de entidad en Orion Context Brocker.
```
POST http://localhost:1026/v2/entities
Header: Content-Type: "application/json"
        fiware-service: openiot
        fiware-servicepath: /

En el body, utilizadmos el contenido de dht22.json
 

```
## 3. Subscripción de cambio de estado de entidad.
### Se crea un subcripcion de la entidad, esto quiere decir que cuando se actualice cualquiera de los datos de la entidad estos seran notificados al servicio de Quantumleap.
### Quantumleap, recive la notificacion del cambio de estado de los datos con el valor de los datos y los perciste en la base de datos CRATE DB.
### Quantumleap cuenta con una api y su documentacion se encuentra en el siguiente link : http://localhost:8668/v2/ui/

```
POST http://localhost:1026/v2/subscriptions/
Header: Content-Type: "application/json"
        fiware-service: openiot
        fiware-servicepath: /

En body, utilizamos el contenido de dht22-subcription.json

```


##  Configurar Servicios:
### CRATE DB:

```
    localhost:4200
    -> (Observar el nombre de la base de datos generada automaicamente)
    mtopeniot



```

### Consulta por entidad registrada en base de datos.

```
select * from mtopeniot.etdht22 limit 100;
```
### Grafana:
### Grafana se configurar para ir obteniendo los datos que son percistido en la base de datos CRATE DB.
### En Grafana se pueden crear tableros par visualizar distintos tipos de datos e incluso grandes volumenes de datos.

### El nombre de la Database: mtopeniot es definido por el Header del registro en Orion Context Broker `fiware-service: openiot`.

```
    localhost:3000

    -> Postgres:
    Name: CrateDB
    Host: crate-db:5432
    Database: mtopeniot
    User: crate
    SSL Mode disable
```
```
New Dashboard

->
FROM etdht22 Time column time_index Metric column entity_id SELECT Column: temperature Column: relativehumidity 
```


## Configuracion IOT-Agent 

### Sirve de intermediario entre Orion Context Brocker y un Dispositivo IOT, para la comunicacion entre el dispositivo IOT se puede utilizar varios tipos de protocolos que son ligeros para la comunicación y el envio de datos bidireccional. Además se puede asignar una Token a cada disposivito para su identificacion e integrar una medida de seguridad.

### Configuracion de SERVICE
```
POST http://localhost:4041/iot/services
Header: Content-type: application/json
        Fiware-service: openiot
        fiware-servicepath: /

En el body se agrea el json iot-agent.service.json

```


### Configuracion de DEVICES
```
POST http://localhost:4041/iot/devices

Header: Content-type: application/json
        Fiware-service: openiot
        Fiware-servicepath: /

En el body de agrega el json iot-agent.device.json
```
#### Se va a registrar una nueva entidad en Orion, esta entidad será del tipo DHT22, entonces podra ser mapeada por Queantum leap y se podran persistir su datos en la base de datos CrateDB y luego ser visualizados mediante Grafana.


## Configuracion Cliente en Raspberry

A modo de cliente o dispositivo IOT se utiliza una raspbery pi 3 modelo b, en donde se encuentra integrado un sensor DHT22 de temperatura y humedad.
```
POST http://localhost:7896/iot/d

Header: Content-Type: text/plain

Params: 
        k=4jggokgpepnvsb2uv4s40d59ov
        i=DHT22003

En el body se agrega:

t|22|h|34

```

El atributos t y h son definidos en iot-agent.devices.json y corresponden a object_id en este paso para la temperatura y humedad.

Mediante el Script en Python que se encuetra en LOCALHOST -> IOT -> FIWARE-DHT22 llamado dht22-fiware+conf.py se utilizan estos parametros y se envia al servicio de IOT-AGENT los datos obtenidos desde el sensor DHT22. Ademas se encuentra un archivo de configuración iot-agent.conf.json en donde se puede configurar la IP o URL del servicio de IOT-AGENT.

# Sistema ALPR y Fiware.
Se integra un servicio de reconocimiento de patentes vehiculares, utilizando Open Alpr y Fiware. 

Se envia un imagen al serviciode Alpr este procesa la imagen y se obtiene como resultado el valor de la patente y su grado de confianza, estos datos son enviado al Orion Broker. Median el servicio de Cygnus se persisten los datos en una base de datos MongoDB.

## Diagrama de Arquitectura + Servicio ALPR

![](https://raw.githubusercontent.com/MRROOX/gpi-fiware/master/TUTORIALES/LOCALHOST/LOCALHOST%2BALPR.png)

## Diagrama de Secuencia + Servicio ALPR
![](https://raw.githubusercontent.com/MRROOX/gpi-fiware/master/TUTORIALES/LOCALHOST/Secuencia%2BALPR.png)

## Configuracion de Cygnus para Persistir en MongoDB.

## 1. Registro de entidad en Orion Context Brocker.
```
POST http://localhost:1026/v2/entities
Header: Content-Type: "application/json"
        fiware-service: openalpr
        fiware-servicepath: /alpr

En el body, utilizadmos el contenido de alpr.json
 
```

## 2. Actualización de Contexto
```
PATCH http://localhost:1026/v2/entities/ALPR0004/attrs
Header: Content-Type: "application/json"
        fiware-service: openalpr
        fiware-servicepath: /alpr

En el body, utilizadmos los atributos que se requiera actualizar.
 
```
## 3. Subcripcion de servicio de Cygnus
```
POST http://localhost:1026/v2/subscriptions/
Header: Content-Type: "application/json"
        fiware-service: openalpr
        fiware-servicepath: /alpr

En el body, utilizadmos el contenido de alpr-cygnus.json
 
```
## Uso de Servicio de ALPR

```
POST http://localhost:8090/v1/identify/plate?country=us
Header: Content-Type: "application/x-www-form-urlencoded"

En el body, se utiliza un form-data, image = <imagen.jpg>


```


# Referencias:


#### https://documenter.getpostman.com/view/513743/RWEcR2DC?version=latest#eb4be0c3-5563-444a-9539-c90ef2df0fda

#### https://github.com/FIWARE/catalogue/blob/master/security/README.md

#### https://fiware-training.readthedocs.io/es_MX/latest/casodeestudio/descripcion/

#### https://hub.docker.com/r/fiware/quantum-leap/dockerfile

#### https://camo.githubusercontent.com/93f44facc15bb6a9edf6ab9968fc1d8b862033fe/68747470733a2f2f6669776172652e6769746875622e696f2f7475746f7269616c732e496f542d4167656e742f696d672f6172636869746563747572652e706e67

#### https://www.fiware.org/wp-content/uploads/2016/12/2_FIWARE-NGSI-Managing-Context-Information-at-large-scale.pdf

#### https://fiware-tutorials.readthedocs.io/en/latest/time-series-data/index.html
