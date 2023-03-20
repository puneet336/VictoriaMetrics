# How to migrate influxDB 1.x data to Victoria Metrics
## Download and install VMUtils

```
wget https://github.com/VictoriaMetrics/VictoriaMetrics/releases/download/v1.89.1/vmutils-linux-amd64-v1.89.1.tar.gz
tar -xf vmutils-linux-amd64-v1.89.1.tar.gz
sudo tar -xf vmutils-linux-amd64-v1.89.1.tar.gz -C  /usr/local/bin/vm/
```
You should see following executables `vmagent-prod  vmalert-prod  vmauth-prod  vmbackup-prod  vmctl-prod  vmrestore-prod`

now trigger migration as - 
```
time /usr/local/bin/vm/vmctl-prod influx --influx-addr "http://den-grafana-01:8086" --influx-filter-time-start "2023-01-18T08:48:32Z" --influx-filter-time-end "2023-01-18T09:48:32Z" --vm-addr "http://127.0.0.1:8428" --influx-database monitoring
```

You should see following output - 

![image](https://user-images.githubusercontent.com/5935825/226252719-09948336-aa01-4854-bede-0539bd9bbcfd.png)



