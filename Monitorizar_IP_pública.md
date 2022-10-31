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
