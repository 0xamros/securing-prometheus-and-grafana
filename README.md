

## 1.  Register a Domain

   - Go to freedomain.one and register a domain name for your server. For example, `myprometheusdomain.one`.
   - Take public Ip of your machine or ec2 as "A" DNS RECORD

    
# Prometheus 
  ## 2. Prometheus Server installation  
-   First,  update your package list to ensure you access the most recent versions available
```
 sudo apt update
 ```

-   Next, install the Prometheus package:
```    
 sudo apt install prometheus prometheus-node-exporter prometheus-pushgateway prometheus-alertmanager
```
-   Enable Prometheus to start on boot and start the service
    
```
 sudo systemctl enable --now prometheus
```
  

-   You can also access Prometheus in a browser using the server’s IP address:
    

	http://<PROMETHEUS_SERVER_IP>:9090


## 3. Generate a Let's Encrypt SSL Certificate

   - After registering the domain, you should have an option to generate a Let's Encrypt SSL certificate for your domain.
   - Follow the prompts and instructions provided by freedomain.one to generate the SSL certificate.
   - At the end of the process, you should have access to the SSL certificate files, including the certificate file (e.g., `myprometheusdomain.one.cer`) and the private key file (e.g., `myprometheusdomain.one.key`).

## 4. **Configure Prometheus Server**

   - Copy the SSL certificate files (`myprometheusdomain.one.crt` and `myprometheusdomain.one.key`) to the Prometheus server's configuration directory.


   - Create the file  `web.yml` configuration file  at  `/etc/prometheus/` to enable TLS and specify the paths to the certificate and key files :

```yaml
     # Prometheus web configuration
     tls_server_config:
       cert_file: /path/to/myprometheusdomain.one.cer
       key_file: /path/to/myprometheusdomain.one.key
```

```
	# run prometheus with web file
     prometheus --web.config.file="/etc/prometheus/web.yml"&
```

## 5. **Configure Prometheus Node Exporter**

   - Copy the SSL certificate files (`myprometheusdomain.one.cer` and `myprometheusdomain.one.key`) to the Prometheus Node Exporter's configuration directory.
   
 ## 6. Setting up basic authentication

1. Install apache2-utils
```
sudo apt-get update && sudo apt install apache2-utils -y
```

2. Generate a hashed password
```
htpasswd -nBC 12 "" | tr -d ':\n'
```
 - create another `web.yml` configuration file with the same keys as prometheus
 ```yaml
     # Prometheus web configuration
     tls_server_config:
       cert_file: /path/to/myprometheusdomain.one.cer
       key_file: /path/to/myprometheusdomain.one.key
     basic_auth_users:  
	   YOUR_USERNAME:  <YOUR_hashed_PASSWORD>
```

## 7. **Update Prometheus Scrape Configuration**

   - In the Prometheus configuration file (`prometheus.yml`), update the scrape configuration for the Node Exporter job to use the TLS certificate and basic_auth:

     ```yaml
     scrape_configs:
       - job_name: 'node-exporter'
		 scheme: https
         tls_config:
           cert_file: /path/to/myprometheusdomain.one.cer
		 basic_auth:
	       username: "amr"
           password: "your password here(not hashed)"
         static_configs:
           - targets: ['node-exporter.myprometheusdomain.one:9100']
     ```

   - Replace `node-exporter.myprometheusdomain.one:9100` with the actual hostname or IP address and port of your Prometheus Node Exporter instance.

## 8. **Restart Prometheus and Node Exporter**

   - After making the necessary configuration changes, restart both the Prometheus server and the Prometheus Node Exporter.

or run node exporter directly 

		```
		prometheus-node-exporter --web.config.file="/root/web.yml" &
		```


# grafana 
## 9. Installation 
 ```
 sudo apt-get update
 sudo apt-get install -y apt-transport-https software-properties-common wget  
 wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
 sudo apt-get install ca-certificates
 sudo add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"   
sudo apt-get install grafana=9.2.3
```
-   Enable and start the grafana-server service:
```
sudo systemctl enable --now grafana-server
```
 
-   Make sure the service is in the Active (running) state:
 ```
 sudo systemctl status grafana-server
 ```
  

-   You can also verify that Grafana is working by accessing it in a web browser at:
    

http://<Grafana_Server_Public IP>:3000

## 10. Edit configuration file

```
vi /etc/grafana/grafana.ini
```
add these line and replace keys with previously generated keys from prometheus
```
root_url = https://grafana.yourdomain.co
protocol = https
cert_key = /etc/grafana/grafana.key
cert_file = /etc/grafana/grafana.crt
```
Once done, save and exit the file. Proceed by restarting the Grafana service:

Bash

```
sudo service grafana-server restart
```
