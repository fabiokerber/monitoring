# Grafana

Pré requisito:

|Tool    |Link|
|-------------|-----------|
|`Vagrant`| https://releases.hashicorp.com/vagrant/2.2.19/vagrant_2.2.19_x86_64.msi
|`VirtualBox`| https://download.virtualbox.org/virtualbox/6.1.30/VirtualBox-6.1.30-148432-Win.exe

# Geral
Dashboard é uma coleção de painéis.<br>
Painel é um mostrador de métricas.<br>
O banco a ser utilizado é o **InfluxDB**. Solução de **Time Series Database (TSDB)**, mais leve para gravações simples de métricas.<br>
O coletor de métricas a ser utilizado será o **Telegraf**. O mesmo tem total compatibilidade com o InfluxDB e claro, deve ser instalado no host à ser monitorado.<br>
Obs:<br>
1. Grafana **(latest)** roda em container no **grafana_srv**.<br>
2. Influx **(v2.1.1)** roda em container no **grafana_srv**.<br>
3. Telegraf **(latest)** é executado no **centos_srv03** e envia as métricas ao Influx no **grafana_srv**.<br>
4. Para efeito de lab, **centos_srv03** possui apenas **um core** de CPU.<br>
5. Container NGINX:8080 **(latest)** é executado no **centos_srv03** apenas para efeitos de monitoramento.<br>
6. Apache:80 **(latest)** é executado no **centos_srv03** apenas para efeitos de monitoramento.<br>

# Grafana + InfluxDB + Telegraf
<kbd>
    <img src="https://github.com/fabiokerber/Grafana/blob/main/img/180220221600.png">
</kbd>
<br />
<br />

# Alterar configurações iniciais e start grafana_srv e centos_srv03
```
> cd install
!! edit .env 
    GRAFANA_IP='192.168.0.245'

!! edit files/telegraf.conf
    [[outputs.influxdb_v2]]
      urls = ["http://192.168.0.245:8086"]

    [[inputs.docker]]
      endpoint = "tcp://[ip]:[port]" - Monitorar Docker em outro host a partir do telegraf local, necessário expor a porta do serviço do docker.

    Conteúdo arquivo override.conf para expor porta Docker:
    [Service]
    ExecStart=
    ExecStart=/usr/bin/dockerd -H fd:// -H tcp://0.0.0.0:2376
    
    $ sudo mkdir -p /etc/systemd/system/docker.service.d/
    $ sudo cp /vagrant/configs/override.conf /etc/systemd/system/docker.service.d/
    $ sudo systemctl daemon-reload
    $ sudo systemctl restart docker.service

    https://github.com/influxdata/telegraf/blob/master/docs/DATA_FORMATS_INPUT.md
    https://github.com/influxdata/telegraf/tree/master/plugins/parsers/grok
    https://github.com/influxdata/telegraf/blob/master/plugins/inputs/logparser/README.md

> vagrant up (irá subir a VM do grafana_srv e centos_srv03)
```

# Instalação inicial InfluxDB
```
http://192.168.0.245:8086/
```
<kbd>
    <img src="https://github.com/fabiokerber/Grafana/blob/main/img/190220222122.png">
</kbd>
<br />
<br />
<kbd>
    <img src="https://github.com/fabiokerber/Grafana/blob/main/img/190220222123.png">
</kbd>
<br />
<br />
<kbd>
    <img src="https://github.com/fabiokerber/Grafana/blob/main/img/190220222124.png">
</kbd>
<br />
<br />
<kbd>
    <img src="https://github.com/fabiokerber/Grafana/blob/main/img/190220222125.png">
</kbd>
<br />
<br />
<kbd>
    <img src="https://github.com/fabiokerber/Grafana/blob/main/img/190220222126.png">
</kbd>
<br />
<br />
unix:///var/run/docker.sock<br>
<kbd>
    <img src="https://github.com/fabiokerber/Grafana/blob/main/img/190220222127.png">
</kbd>
<br />
<br />
Anotar info!<br>
<kbd>
    <img src="https://github.com/fabiokerber/Grafana/blob/main/img/190220222129.png">
</kbd>
<br />
<br />

# Inserir token no serviço telegraf executando em centos_srv03
```
> cd install
> vagrant ssh centos_srv03
    $ sudo vim /etc/telegraf/telegraf.conf
        [[outputs.influxdb_v2]]
          ...
          token = "$INFLUX_TOKEN"

    $ sudo systemctl restart telegraf
```

# Verificar no InfluxDB se estão chegando os dados de monitoramento
```
http://192.168.0.245:8086
```
<kbd>
    <img src="https://github.com/fabiokerber/Grafana/blob/main/img/190220222248.png">
</kbd>
<br />
<br />

# Acessar o Grafana
```
http://192.168.0.245:3000
    admin | (admin)
```

# Configurar Data Source no Grafana
Antes de configurar o Data Source no Grafana, é necessário anotar o **admin's Token** no InfluxDB!<br>
<kbd>
    <img src="https://github.com/fabiokerber/Grafana/blob/main/img/190220222259.png">
</kbd>
<br />
<br />
<kbd>
    <img src="https://github.com/fabiokerber/Grafana/blob/main/img/190220222300.png">
</kbd>
<br />
<br />
Agora acesse o Grafana e configure o Data Source.<br>
<kbd>
    <img src="https://github.com/fabiokerber/Grafana/blob/main/img/190220220900.png">
</kbd>
<br />
<br />
<kbd>
    <img src="https://github.com/fabiokerber/Grafana/blob/main/img/190220222258.png">
</kbd>
<br />
<br />

# Primeira Dashboard
Adicionar painel e Query.<br>
Query **Random Walk**, são números gerados de forma aleatória apenas para exemplo de gráfico mesmo.<br> 
<kbd>
    <img src="https://github.com/fabiokerber/Grafana/blob/main/img/180220221516.png">
</kbd>
<br />
<br />
<kbd>
    <img src="https://github.com/fabiokerber/Grafana/blob/main/img/180220221518.png">
</kbd>
<br />
<br />
<kbd>
    <img src="https://github.com/fabiokerber/Grafana/blob/main/img/180220221523.png">
</kbd>
<br />
<br />


# Novo Dashboard utilizando variáveis
Variáveis são utilizadas para quando for criar novos dashboard, utilizar os mesmos valores padrão.<br>
**Refresh:** *On Time Range Change* - Quando alterar o valor de tempo na visualização, o Grafana atualiza o valor da variável em questão.<br>

```
import "influxdata/influxdb/v1"
v1.tagValues(
    bucket: v.bucket,
    tag: "host",
    predicate: (r) => true,
    start: -1d
)
```

Obs:<br>
1. Clicando fora dos campos, já é exibido um preview do valor coletado.<br>
<kbd>
    <img src="https://github.com/fabiokerber/Grafana/blob/main/img/200220220858.png">
</kbd>
<br />
<br />

# Query Build InfluxDB e Primeiro Painel Grafana CPU
```
from(bucket: "telegraf")
  |> range(start: v.timeRangeStart, stop: v.timeRangeStop)
  |> filter(fn: (r) => r["host"] == "${server}")
  |> filter(fn: (r) => r["_measurement"] == "cpu")
  |> filter(fn: (r) => r["_field"] == "usage_idle" or r["_field"] == "usage_user" or r["_field"] == "usage_system" or r["_field"] == "usage_iowait" or r["_field"] == "usage_irq" or r["_field"] == "usage_nice" or r["_field"] == "usage_softirq" or r["_field"] == "usage_steal" or r["_field"] == "usage_guest" or r["_field"] == "usage_guest_nice")
  |> filter(fn: (r) => r["cpu"] == "cpu-total")
  |> aggregateWindow(every: v.windowPeriod, fn: mean, createEmpty: false)
  |> yield(name: "mean")
```
```
Stress: $ stress-ng -c 0 -l 95
```
<kbd>
    <img src="https://github.com/fabiokerber/Grafana/blob/main/img/200220220753.png">
</kbd>
<br />
<br />
<kbd>
    <img src="https://github.com/fabiokerber/Grafana/blob/main/img/200220220813.png">
</kbd>
<br />
<br />
<kbd>
    <img src="https://github.com/fabiokerber/Grafana/blob/main/img/200220220815.png">
</kbd>
<br />
<br />
<kbd>
    <img src="https://github.com/fabiokerber/Grafana/blob/main/img/200220220827.png">
</kbd>
<br />
<br />

# Painel Memória
```
from(bucket: "telegraf")
  |> range(start: v.timeRangeStart, stop: v.timeRangeStop)
  |> filter(fn: (r) => r["_measurement"] == "mem")
  |> filter(fn: (r) => r["_field"] == "total" or r["_field"] == "cached")
  |> filter(fn: (r) => r["host"] == "${server}")
  |> aggregateWindow(every: v.windowPeriod, fn: mean, createEmpty: false)
  |> yield(name: "last")
```
```
Stress: $ stress-ng --vm 2 --vm-bytes 256M --timeout 240s
```
<kbd>
    <img src="https://github.com/fabiokerber/Grafana/blob/main/img/200220221034.png">
</kbd>
<br />
<br />
<kbd>
    <img src="https://github.com/fabiokerber/Grafana/blob/main/img/200220221043.png">
</kbd>
<br />
<br />
<kbd>
    <img src="https://github.com/fabiokerber/Grafana/blob/main/img/200220221044.png">
</kbd>
<br />
<br />

# Painel Uptime
```
from(bucket: "telegraf")
  |> range(start: v.timeRangeStart, stop: v.timeRangeStop)
  |> filter(fn: (r) => r["_field"] == "uptime")
  |> filter(fn: (r) => r["_measurement"] == "system")
  |> filter(fn: (r) => r["host"] == "centos-srv03")
  |> aggregateWindow(every: v.windowPeriod, fn: mean, createEmpty: false)
  |> yield(name: "last")
```
<kbd>
    <img src="https://github.com/fabiokerber/Grafana/blob/main/img/200220221111.png">
</kbd>
<br />
<br />

# Painel Disco
```
from(bucket: "telegraf")
  |> range(start: v.timeRangeStart, stop: v.timeRangeStop)
  |> filter(fn: (r) => r["_measurement"] == "disk" or r["_measurement"] == "diskio")
  |> filter(fn: (r) => r["_field"] == "total" or r["_field"] == "used")
  |> filter(fn: (r) => r["device"] == "sda1")
  |> filter(fn: (r) => r["fstype"] == "xfs")
  |> filter(fn: (r) => r["host"] == "${server}")
  |> filter(fn: (r) => r["mode"] == "rw")
  |> filter(fn: (r) => r["path"] == "/")
  |> aggregateWindow(every: v.windowPeriod, fn: mean, createEmpty: false)
  |> yield(name: "last")
```
<kbd>
    <img src="https://github.com/fabiokerber/Grafana/blob/main/img/200220221130.png">
</kbd>
<br />
<br />
<kbd>
    <img src="https://github.com/fabiokerber/Grafana/blob/main/img/200220221131.png">
</kbd>
<br />
<br />

# Painel Docker
```
from(bucket: "telegraf")
  |> range(start: v.timeRangeStart, stop: v.timeRangeStop)
  |> filter(fn: (r) => r["_measurement"] == "docker")
  |> filter(fn: (r) => r["_field"] == "n_containers_running")
  |> filter(fn: (r) => r["engine_host"] == "${server}")
  |> filter(fn: (r) => r["host"] == "${server}")
  |> filter(fn: (r) => r["server_version"] == "20.10.12")
  |> aggregateWindow(every: v.windowPeriod, fn: mean, createEmpty: false)
  |> yield(name: "last")
```
<kbd>
    <img src="https://github.com/fabiokerber/Grafana/blob/main/img/200220221132.png">
</kbd>
<br />
<br />

# Resultado Paineis de Monitoração de Infra
<kbd>
    <img src="https://github.com/fabiokerber/Grafana/blob/main/img/200220221133.png">
</kbd>
<br />
<br />

# Painel de monitoramento HTTP 200 (Apache)
Serão monitorados os logs do apache contidos em /var/log/httpd/.<br>
Neste caso não será habilitado um input próprio para o apache, mas sim um módulo do telegraf que analisa/"parseia" logs.<br>
```
from(bucket: "telegraf")
  |> range(start: v.timeRangeStart, stop: v.timeRangeStop)
  |> filter(fn: (r) => r["status"] == "200")
  |> filter(fn: (r) => r["_field"] == "counter")
  |> filter(fn: (r) => r["_measurement"] == "http_query_request_count")
  |> filter(fn: (r) => r["endpoint"] == "/api/v2/query")
  |> filter(fn: (r) => r["org_id"] == "101c17b5d2fa3be8")
  |> aggregateWindow(every: v.windowPeriod, fn: mean, createEmpty: false)
  |> yield(name: "last")
```
<kbd>
    <img src="https://github.com/fabiokerber/Grafana/blob/main/img/200220221233.png">
</kbd>
<br />
<br />