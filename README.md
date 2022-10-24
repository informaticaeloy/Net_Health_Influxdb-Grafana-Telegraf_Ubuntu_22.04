# Net_Health_Influxdb-Grafana-Telegraf_Ubuntu_22.04
Sistema de monitorización de red en Ubuntu 22.04 usando Influxdb + Grafana + Telefrag

![image](https://user-images.githubusercontent.com/20743678/197484311-d46c47aa-b413-42c6-85b3-8300e2c9bf30.png)

apt-get update
apt-get upgrade

wget -qO- https://repos.influxdata.com/influxdb.key | sudo apt-key add -
source /etc/lsb-release
echo "deb https://repos.influxdata.com/${DISTRIB_ID,,} ${DISTRIB_CODENAME} stable" | sudo tee /etc/apt/sources.list.d/influxdb.list

sudo apt-get update
sudo apt-get install influxdb2
sudo service influxdb start

systemctl enable influxdb

Editamos c/etc/telegraf/telegraf.conf

![image](https://user-images.githubusercontent.com/20743678/197489809-63245a4f-6666-4ea3-b0cd-5f6d7e171ba8.png)

Creamos fichero nuevo /etc/telegraf/telegraf.d/00-influx.conf

[outputs.influxdb_v2]
   urls = ["http://localhost:8086"]
   ## Token for authentication.
   token = "gve6Zjj_isOKjUg=="
   ## Organization is the name of the organization you wish to write to; must exist.
   organization = "MyOrg"
   ## Destination bucket to write into.
   bucket = "telegraf"

![image](https://user-images.githubusercontent.com/20743678/197490156-9213bb9f-cc17-47a5-828d-4680454dccc4.png)

systemctl restart telegraf

systemctl status telegraf

![image](https://user-images.githubusercontent.com/20743678/197490681-989bbee7-dc45-483c-b7ae-bf630a3f95b9.png)

sudo apt-get install -y apt-transport-https

wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -

echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list

sudo apt-get update

sudo apt-get install grafana

sudo systemctl start grafana-server

sudo systemctl status grafana-server

![image](https://user-images.githubusercontent.com/20743678/197492249-416645c5-6e9f-4888-baca-13779383ba13.png)

sudo nano /etc/grafana/grafana.ini

sudo systemctl restart grafana-server

![image](https://user-images.githubusercontent.com/20743678/197493459-4a07a856-20a3-449f-a828-30dacef8c58c.png)

![image](https://user-images.githubusercontent.com/20743678/197493637-6a881783-a3e9-420d-87ce-8a73163d1289.png)

![image](https://user-images.githubusercontent.com/20743678/197494188-97e79bca-5c78-4c0f-bd74-c97addbf6aa6.png)

![image](https://user-images.githubusercontent.com/20743678/197495246-e65db0d8-b927-4e35-b3f8-8769bc66333f.png)

![image](https://user-images.githubusercontent.com/20743678/197495448-5771147e-3de6-4164-91d6-e1281c18d02e.png)


