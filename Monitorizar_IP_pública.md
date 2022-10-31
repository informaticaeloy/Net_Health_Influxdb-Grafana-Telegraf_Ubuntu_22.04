## Monitorizar IP Pública con InfluxDB, Grafana y Telegraf

### 1. Creamos una cuenta FREE en IPINFO.IO

```shell
[https://ipinfo.io]https://ipinfo.io
```

Nos darán un tóken:

<kbd>![image](https://user-images.githubusercontent.com/20743678/198982285-6aa45366-0c55-4b2d-b834-b14a87d2f570.png)</kbd>

Este tóken podemos utilizarlo directamente desde una shell y nos devuleve un JSON con la info de nuestra IP Pública:

```shell
curl ipinfo.io?token=43dafakefakeb43a
```

<kbd>![image](https://user-images.githubusercontent.com/20743678/198984404-44bffcff-f9a6-461d-a70f-62ca3256ddcb.png)</kbd>

#### Warning - :skull: Haz un Snapshot :eyes:
