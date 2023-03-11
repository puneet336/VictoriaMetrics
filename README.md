# Install and configure VictoriaMetrics on CentOS 7


## Setup victoriametrics service account 

For running VictoriaMetrics as non-root user, we need to create a victoriametrics user and group. To create a new system user and group, you can use these two commands:
```
$ sudo groupadd -r victoriametrics
$ sudo useradd -g victoriametrics -d /var/lib/victoriametrics -s /sbin/nologin --system victoriametrics
```

## Download  and install VictoriaMetrics

VictoriaMetrics installation files are packaged as precompiled binaries and are available [here](https://github.com/VictoriaMetrics/VictoriaMetrics/releases). In this guide, we'll be installing version v1.88.1  of VictoriaMetrics. To start download, execute the command:
```
$ curl -L https://github.com/VictoriaMetrics/VictoriaMetrics/releases/download/v1.88.1/victoria-metrics-linux-amd64-v1.88.1.tar.gz --output victoria-metrics-linux-amd64-v1.88.1.tar.gz
```
The victoria-metrics-linux-amd64-v1.88.1.tar.gz contains only victoria-metrics-prod . Extract the victoria-metrics-prod executable to /usr/local/bin.
```
$ sudo tar -xf victoria-metrics-linux-amd64*.tar.gz -C /usr/local/bin/
```


Modify the ownership of executable:
```
$ sudo chown -v root:root /usr/local/bin/victoria-metrics-prod
```

## Configure VictoriaMetrics
Create a directory for VictoriaMetrics data and set owner:
```
$ sudo mkdir -v /var/lib/victoria-metrics-data
$ sudo chown -v victoriametrics:victoriametrics /var/lib/victoria-metrics-data
```
Expected output - 
>changed ownership of ‘/usr/local/bin/victoria-metrics-prod’ from 1000:1000 to root:root

Create new systemd services.
```
$ sudo vi /etc/systemd/system/victoriametrics.service
```
Add the following lines to the file:
```
[Unit]
Description=High-performance, cost-effective and scalable time series database, long-term remote storage for Prometheus
After=network.target


[Service]
Type=simple
User=victoriametrics
Group=victoriametrics
StartLimitBurst=5
StartLimitInterval=0
Restart=on-failure
RestartSec=1

ExecStart=/usr/local/bin/victoria-metrics-prod \
        -storageDataPath=/var/lib/victoria-metrics-data \
        -httpListenAddr=127.0.0.1:8428 \
        -retentionPeriod=1

ExecStop=/bin/kill -s SIGTERM $MAINPID

LimitNOFILE=65536
LimitNPROC=32000

[Install]
WantedBy=multi-user.target
```
where - 
-storageDataPath - the path of the data directory. VictoriaMetrics stores all the data in this directory.
-httpListenAddr - The IP address and port on which VictoriaMetrics will listen. If you want to connect to VictoriaMetrics not only from localhost (127.0.0.1), then specify 0.0.0.0:8428. In this case, used 127.0.0.1 because vmauth (for authentification) will be used for external connection and it will be forward all queries to VictoriaMetrics.
-retentionPeriod - retention for stored data. Older data is automatically deleted. retentionPeriod=1 means that the data will be stored for 1 month and then deleted.


Enable and start VictoriaMetrics service to run on system boot automatically.
```
$ sudo systemctl enable victoriametrics.service --now
```

## Verify VictoriaMetrics

Check Service status - 
```
$sudo systemctl status victoriametrics.service
```

Expected Output - 
```
victoriametrics.service - High-performance, cost-effective and scalable time series database, long-term remote storage for Prometheus
   Loaded: loaded (/etc/systemd/system/victoriametrics.service; disabled; vendor preset: disabled)
   Active: active (running) since Sat 2023-03-11 09:05:51 MST; 10s ago
  Process: 7990 ExecStop=/bin/kill -s SIGTERM $MAINPID (code=exited, status=0/SUCCESS)
 Main PID: 7993 (victoria-metric)
    Tasks: 13
   Memory: 11.1M
   CGroup: /system.slice/victoriametrics.service
           └─7993 /usr/local/bin/victoria-metrics-prod -storageDataPath=/var/lib/victoria-metrics-data -httpListenAddr=127.0.0.1:8428 -retentionPeriod=1

Mar 11 09:05:51 server2.lab.com victoria-metrics-prod[7993]: 2023-03-11T16:05:51.152Z        info        VictoriaMetrics/lib/mergeset/table.go:319      ...9AB8"...
Mar 11 09:05:51 server2.lab.com victoria-metrics-prod[7993]: 2023-03-11T16:05:51.157Z        info        VictoriaMetrics/lib/mergeset/table.go:359      ...Bytes: 0
Mar 11 09:05:51 server2.lab.com victoria-metrics-prod[7993]: 2023-03-11T16:05:51.158Z        info        VictoriaMetrics/lib/mergeset/table.go:319      ...9AB7"...
Mar 11 09:05:51 server2.lab.com victoria-metrics-prod[7993]: 2023-03-11T16:05:51.164Z        info        VictoriaMetrics/lib/mergeset/table.go:359      ...Bytes: 0
Mar 11 09:05:51 server2.lab.com victoria-metrics-prod[7993]: 2023-03-11T16:05:51.168Z        info        VictoriaMetrics/app/vmstorage/main.go:124      ...Bytes: 0
Mar 11 09:05:51 server2.lab.com victoria-metrics-prod[7993]: 2023-03-11T16:05:51.170Z        info        VictoriaMetrics/app/vmselect/promql/rollup_resu...sult"...
Mar 11 09:05:51 server2.lab.com victoria-metrics-prod[7993]: 2023-03-11T16:05:51.171Z        info        VictoriaMetrics/app/vmselect/promql/rollup_resu...Bytes: 0
Mar 11 09:05:51 server2.lab.com victoria-metrics-prod[7993]: 2023-03-11T16:05:51.172Z        info        VictoriaMetrics/app/victoria-metrics/main.go:70... seconds
Mar 11 09:05:51 server2.lab.com victoria-metrics-prod[7993]: 2023-03-11T16:05:51.172Z        info        VictoriaMetrics/lib/httpserver/httpserver.go:96....1:8428/
Mar 11 09:05:51 server2.lab.com victoria-metrics-prod[7993]: 2023-03-11T16:05:51.172Z        info        VictoriaMetrics/lib/httpserver/httpserver.go:97...g/pprof/
Hint: Some lines were ellipsized, use -l to show in full.
```


Check if you can get access to VictoriaMetrics via API:
```
$ curl http://localhost:8428/api/v1/query -d 'query={job=~".*"}'
```
If everything fine that output must be something like that:
```
{"status":"success","data":{"resultType":"vector","result":[]}}
```
If at this stage there are any errors or problems, then check the status and logs of the service.
```
$ systemctl status victoriametrics.service
$ journalctl -u victoriametrics.service
```



