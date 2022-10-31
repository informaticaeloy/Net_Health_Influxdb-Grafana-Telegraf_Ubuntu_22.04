## Monitorizar IP Pública con InfluxDB, Grafana y Telegraf

### 1. Creamos una cuenta FREE en IPINFO.IO

[https://ipinfo.io](https://ipinfo.io)

Nos darán un tóken:

<kbd>![image](https://user-images.githubusercontent.com/20743678/198982285-6aa45366-0c55-4b2d-b834-b14a87d2f570.png)</kbd>

Este tóken podemos utilizarlo directamente desde una shell y nos devuleve un JSON con la info de nuestra IP Pública:

```shell
curl ipinfo.io?token=43dafakefakeb43a
```

<kbd>![image](https://user-images.githubusercontent.com/20743678/198984404-44bffcff-f9a6-461d-a70f-62ca3256ddcb.png)</kbd>

#### Warning - :skull: Haz un Snapshot :eyes:

### 2. Configurar el agente de Telegraf para tragar estos datos

Editamos el fichero /etc/telegraf/telegraf.conf

```shell
sudo nano /etc/telegraf/telegraf.conf
```

Sobre la línea 6650 (en mi caso) tenemos los inputs. En mi ejemplo tengo dos del tipo [[inputs.ping]] que hace ping a la IP 8.8.8.8 y a www.microsoft.com con los que compruebo la "calidad de mi conexión a internet, ya que si el ping falla hacia estas dos dirección es que mi internet está caído (en teoría), y también tengo la métrica de la latencia para conocer el estado de la conexión. 

Ahí creamos un input del tipo [[inputs.http]]. Dentro de él, creamos otro del tipo [[inputs.http.json_v2]] con el que leeremos el JSON devuelto por la consulta anterior y nos quedaremos sólo con el dato de la IP pública, mediante [[inputs.http.json_v2.field]] y path = "ip":

```shell
 [[inputs.ping]]
   name_suffix = "_Host"
   urls = ["8.8.8.8"]
   count = 4
   ping_interval = 1.0
   timeout = 2.0
   [inputs.ping.tags]
      nombre = "Google DNS"

 [[inputs.ping]]
   name_suffix = "_Host"
   urls = ["www.microsoft.com"]
   count = 4
   ping_interval = 1.0
   timeout = 2.0
   [inputs.ping.tags]
      nombre = "Microsoft Site"

[[inputs.http]]
  urls = [ "https://ipinfo.io?token=43dafakefakeb43a" ]
  data_format = "json_v2"
  interval = "60s"
  [[inputs.http.json_v2]]
    measurement_name = "IPINFO_IO"
    [[inputs.http.json_v2.field]]
      path = "ip"
```

<kbd>![image](https://user-images.githubusercontent.com/20743678/198988588-4f8bf9fb-4b1c-422d-9c48-3ab7ef460904.png)</kbd>

En la imagen, el apartado "interval="60s", lo hago por que la cuenta gratuita de ipinfo.io nos da 50K consultas al mes. Hacemos un cálculo para no pasarnos de ese número al mes:

```shell
50000 consultas / 30 días al mes / 24 horas al día / 60 minutos por hora = 1,157 consultas por minuto
```

Si hacemos una consulta cada 60 segundos (1 minuto) no nos pasaremos de las condiciones de nuestra cuenta FREE

<kbd>![image](https://user-images.githubusercontent.com/20743678/198989563-d9333b81-36ea-46d6-b012-3035da12c518.png)</kbd>

#### IMPORTANTE: Se supone que ya tenemos nuestro InfluxDB, Grafana y Telegraf funcionando y comunicándosen entre ellos

### 3. Crear un Dashboard en Grafana

Para la visualización de estos datos en Grafana, he utilizado "Time Series" como tipo de gráfico para el tiempo medio de respuesta y "Discrete" para la monitorización de la IP Pública y el porcentaje de paquetes perdidos.

Si no tienes "Discrete" puedes instalarlo desde la shell con el comando siguiente:

```shell
grafana-cli plugins install natel-discrete-panel
```

<kbd>![image](https://user-images.githubusercontent.com/20743678/198990863-9bacb2f9-3815-4b5e-861d-ef092589e403.png)</kbd>

#### 3.1 Panel para la IP Pública ("Discrete")

Query en grafana para obtener la medicón relativa a la IP Pública:

```shell
from(bucket: "telegraf")
  |> range(start: v.timeRangeStart, stop:v.timeRangeStop)
  |> filter(fn: (r) =>
    r._measurement == "IPINFO_IO" and
    r._field == "ip"
  )
```

<kbd>![image](https://user-images.githubusercontent.com/20743678/198992429-96c40a78-cb16-47fe-8a3d-bd6b051b4aa7.png)</kbd>

1- Visualización de las IPs públicas que vamos teniendo

2- Tipo de visualización ("Discrete")

3- He creado un "Color Mapping", para que el gráfivco tenga estos 3 colores conforme vaya cambiando la IP. En mi caso conozco los posibles valores de las IPS, por loq ue asigno un color a cada una de estas IPs.

4- Código de la Query

#### 3.2 Panel para el tiempo medio de respuesta del Ping ("Time Series")

Query en grafana para el tiempo medio de respuesta del ping

```shell
from(bucket: "telegraf")
  |> range(start: v.timeRangeStart, stop:v.timeRangeStop)
  |> filter(fn: (r) =>
    r._measurement == "ping" and
    r._field == "average_response_ms"
  )
```

<kbd>![image](https://user-images.githubusercontent.com/20743678/198993214-41d508fd-8a77-493a-9493-a371d4982871.png)</kbd>

1- Visualización del tiempo medio de respuesta del ping a la IP 8.8.8.8 y a www.microsoft.com

2- Tipo de visualización "Time series"

3- Código de la Query

#### 3.3 Panel para el porcentaje de paquetes perdidos ("Discrete")

Query en Grafana para el porcentaje de paquetes perdidos:

```shell
from(bucket: "telegraf")
  |> range(start: v.timeRangeStart, stop:v.timeRangeStop)
  |> filter(fn: (r) =>
    r._measurement == "ping" and
    r._field == "percent_packet_loss"
  )
 ```  
 
<kbd>![image](https://user-images.githubusercontent.com/20743678/198995546-66126aef-b15a-4906-8994-b89fd7501015.png)</kbd>


1- Visualización del porcentaje de paquetes perdidos del ping a la IP 8.8.8.8 y a www.microsoft.com

2- Tipo de visualización "Discrete"

3- Código de la Query

#### Warning - :skull: Haz un Snapshot :eyes:

#### Enjoy :wink:
