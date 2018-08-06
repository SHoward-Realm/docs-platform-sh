# Advanced Monitoring

## Background

This guide goes through setting up a monitoring and alerting system that integrates into the Realm Object Server deployed as either a single instance or a distributed cluster. There are  sample test apps that can be used to report the health of the whole system and integrate the results into an alerting system.

Within this guide, we will create and use a number of javascript files.  [They can be found completed in this repository](https://github.com/realm/realm-server-side-samples/tree/master/19-Advanced-Monitoring-Walkthrough/Prometheus-Stats-Sink).  

## **Install Prometheus on Ubuntu 16.04**

To start, let's open up a terminal window or ssh session and then run through the following:

```bash
#Update and install nginx
ubuntu@monitoring-server:~$ sudo apt-get update
ubuntu@monitoring-server:~$ sudo apt install nginx -y
ubuntu@monitoring-server:~$ sudo systemctl start nginx
ubuntu@monitoring-server:~$ sudo systemctl enable nginx

#Create system users
ubuntu@monitoring-server:~$ sudo useradd --no-create-home --shell /bin/false prometheus
ubuntu@monitoring-server:~$ sudo useradd --no-create-home --shell /bin/false node_exporter

#Create directories and assign ownership
ubuntu@monitoring-server:~$ sudo mkdir /etc/prometheus
ubuntu@monitoring-server:~$ sudo mkdir /var/lib/prometheus
ubuntu@monitoring-server:~$ sudo chown prometheus:prometheus /etc/prometheus
ubuntu@monitoring-server:~$ sudo chown prometheus:prometheus /var/lib/prometheus

#Download Prometheus
ubuntu@monitoring-server:~$ curl -LO https://github.com/prometheus/prometheus/releases/download/v2.2.1/prometheus-2.2.1.linux-amd64.tar.gz
ubuntu@monitoring-server:~$ tar xvf prometheus-2.2.1.linux-amd64.tar.gz

#Move the binaries to the bin folder
ubuntu@monitoring-server:~$ sudo cp prometheus-2.2.1.linux-amd64/prometheus /usr/local/bin
ubuntu@monitoring-server:~$ sudo cp prometheus-2.2.1.linux-amd64/promtool /usr/local/bin
ubuntu@monitoring-server:~$ sudo chown prometheus:prometheus /usr/local/bin/prometheus
ubuntu@monitoring-server:~$ sudo chown prometheus:prometheus /usr/local/bin/promtool

#Now move the folder libraries
ubuntu@monitoring-server:~$ sudo cp -r prometheus-2.2.1.linux-amd64/consoles /etc/prometheus
ubuntu@monitoring-server:~$ sudo cp -r prometheus-2.2.1.linux-amd64/console_libraries /etc/prometheus
ubuntu@monitoring-server:~$ sudo chown -R prometheus:prometheus /etc/prometheus/consoles

#Create a new config file
ubuntu@monitoring-server:~$ sudo vi /etc/prometheus/prometheus.yml

```

The contents of your config file should read something like: 

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    scrape_interval: 5s
    static_configs:
      - targets: ['localhost:9090']
```

Then let's head back to the terminal window to create a unit file: 

```aspnet
#Now lets create the unit file
ubuntu@monitoring-server:~$ sudo vi /etc/systemd/system/prometheus.service

```

The contents of your unit file should read something like: 

```text
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
    --config.file /etc/prometheus/prometheus.yml \
    --storage.tsdb.path /var/lib/prometheus/ \
    --web.console.templates=/etc/prometheus/consoles \
    --web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target

```

Back to our terminal again: 

```text
#Now lets start and enable the daemon
ubuntu@monitoring-server:~$ sudo systemctl daemon-reload
ubuntu@monitoring-server:~$ sudo systemctl start prometheus
ubuntu@monitoring-server:~$ sudo systemctl enable prometheus

```

To confirm success, you can now go to http://&lt;YOUR\_SERVER\_IP&gt;:9090 in a web browser to see the Prometheus UI

## **Install and Configure Exporters**

Now let’s set-up a node exporter - this will collect resource metrics from the server. If running an [enterprise cluster](../), you will add this to a sync worker node.

```bash
ubuntu@swg-01-east:~$ wget https://github.com/prometheus/node_exporter/releases/download/v0.16.0/node_exporter-0.16.0.linux-amd64.tar.gz
ubuntu@swg-01-east:~$ tar -xvf node_exporter-0.16.0.linux-amd64.tar.gz

#Copy to the bin folder and change owner
ubuntu@swg-01-east:~$ sudo cp node_exporter-0.16.0.linux-amd64/node_exporter /usr/local/bin
ubuntu@swg-01-east:~$ sudo useradd --no-create-home --shell /bin/false node_exporter
ubuntu@swg-01-east:~$ sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter

#Create the unit file
ubuntu@swg-01-east:~$ sudo vi /etc/systemd/system/node_exporter.service

```

The contents of your unit file:

{% code-tabs %}
{% code-tabs-item title="node\_exporter.service" %}
```text
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Now back to the terminal: 

```bash
#Reload and Enable
ubuntu@swg-01-east:~$ sudo systemctl daemon-reload
ubuntu@swg-01-east:~$ sudo systemctl start node_exporter
ubuntu@swg-01-east:~$ sudo systemctl enable node_exporter

```

Now go back to your prometheus monitoring server and adjust the config

```text
ubuntu@monitoring-server:~$ sudo vi /etc/prometheus/prometheus.yml

```

And add this to the end of the file

{% code-tabs %}
{% code-tabs-item title="prometheus.yml" %}
```text
- job_name: 'node_exporter'
    scrape_interval: 5s
    static_configs:
      - targets: ['<IP_OF_SYNC_WORKER>:9100']
```
{% endcode-tabs-item %}
{% endcode-tabs %}

And restart the Prometheus service from the terminal window: 

```bash
ubuntu@monitoring-server:~$ sudo systemctl restart prometheus

```

Now go back to the Prometheus web-ui and you should be able to add metrics into graphs

![Viewing metrics via the Prometheus Web UI](https://lh4.googleusercontent.com/fhxdf_gKM5LwJGle470EaP98vkNwyPgFI8a12bVREYX2Oddvmxzpib6sPFPxkUnAxUUjiDcJ_5_61O3JsoR6sA9CXfbRgHaVbn9llA-Jo_AWioYp6czWyhQOPVokS0848siaXUKJ)

## **Add a Prometheus Monitoring Endpoint to Scrape for Realm**

Adding the above `node_exporter` will grab metrics about what is going on the server but does give realm specific metrics. To do this we will add a `StatsSink` to each realm process in our distributed cluster.  
SSH to a server running a realm process and open the JS code in an editor

```text
ubuntu@swg-01-east:~/syncWorker1$ vi syncWorker.js

```

Then add the following lines to your code. Prometheus.js goes at the top of the JS code near your require statements and then add the new class constructors and configuration key-values in your StartConfig including the new `MetricsService()`  
  
[**You can use these example files for reference.**  ](https://github.com/realm/realm-server-side-samples/tree/master/19-Advanced-Monitoring-Walkthrough/Prometheus-Stats-Sink)\*\*\*\*

{% hint style="warning" %}
`address: getIPv4Address('eth0'),`  
Is only needed for sync-workers 
{% endhint %}

  
Be sure to npm install any packages and do not double declare the ‘os’ npm package if you are already using it in your script. [For instance a sync-worker could come out looking like this](https://github.com/realm/realm-server-side-samples/blob/master/19-Advanced-Monitoring-Walkthrough/Services/syncService.js).

Next restart the realm process and you should see this in the logs:

```text
detail: Starting service prometheus-metrics

```

This will use Consul to register a new prometheus-metrics service for each realm node we configure this for - now go back to your Prometheus server and add this to your prometheus.yml

{% code-tabs %}
{% code-tabs-item title="promtheus.yml" %}
```yaml
  - job_name: 'consul-metrics'
    scrape_interval: 5s
    consul_sd_configs:
      - server: '<ADDRESS_OF_CONSUL>:8500'
        services: ['prometheus-metrics']

```
{% endcode-tabs-item %}
{% endcode-tabs %}

After you restart prometheus you should start seeing realm specific metrics like this:

![Realm metrics within the Prometheus Web UI](https://lh5.googleusercontent.com/zrSGHBO9ljqWLorFTjRBCmNEUltYumuUn8kAA4DpuRojN5Ly_Y8Z-f1iSTBPkeRU6MeUBtvcu_xNZ1Vrji-IGHiEAxeudq0Gwe44-98432kMw9SHYXx2hj0iBKpUEGZp6tNAKoIl)

## Add Health Endpoints

Realm recommends configuring a custom health endpoint on the ingress node for a distributed realm cluster. [The custom health endpoint would look like this](https://github.com/realm/realm-server-side-samples/blob/master/19-Advanced-Monitoring-Walkthrough/Services/customHealthService.js).

[And it will be integrated into the ROS start like so.  ](https://github.com/realm/realm-server-side-samples/blob/master/19-Advanced-Monitoring-Walkthrough/Services/proxyService.js)

By adding the `new CustomHealthService(),` class.

You can customize this health endpoint code to return whatever is relevant to your monitoring system such  as a different HTTP error code or more information in the JSON response. We would recommend installing the CustomHealthService on any ROS service that is not the replicated Sync Service as this checks to make sure there is a healthy master sync-worker and responds with the JSON:

```text
{"version":"3.6.12","status":"UP"}
```

On the sync-workers we would recommend using the built-in health service that is part of the server startConfig with class `new ros.HealthService()`

When querying the /health endpoint you will get a 200 back with the version of realm-object-server installed like this if the ROS server has started:

```text
{"version":"3.6.6"}
```

## Install AlertManager

Now that we have some basic reporting and health checks setup we need to get alerts based on thresholds being exceeded or triggers on our HTTP checks. SSH back to your monitoring server

```text
#Create the user
ubuntu@monitoring-server:~$ sudo adduser --no-create-home --disabled-login --shell /bin/false --gecos "Alertmanager User" alertmanager

#Create the directories and config files
ubuntu@monitoring-server:~$ sudo mkdir /etc/alertmanager
ubuntu@monitoring-server:~$ sudo mkdir /etc/alertmanager/template
ubuntu@monitoring-server:~$ sudo mkdir -p /var/lib/alertmanager/data
ubuntu@monitoring-server:~$ sudo touch /etc/alertmanager/alertmanager.yml

#Now assign ownership to the alertmanager user
ubuntu@monitoring-server:~$ sudo chown -R alertmanager:alertmanager /etc/alertmanager
ubuntu@monitoring-server:~$ sudo chown -R alertmanager:alertmanager /var/lib/alertmanager

#Download and untar alertmanager
ubuntu@monitoring-server:~$ wget https://github.com/prometheus/alertmanager/releases/download/v0.15.0-rc.2/alertmanager-0.15.0-rc.2.linux-amd64.tar.gz
ubuntu@monitoring-server:~$ tar -xvzf alertmanager-0.15.0-rc.2.linux-amd64.tar.gz

#Copy the data to your bin and change ownership
ubuntu@monitoring-server:~$ sudo cp alertmanager-0.15.0-rc.2.linux-amd64/alertmanager /usr/local/bin
ubuntu@monitoring-server:~$ sudo cp alertmanager-0.15.0-rc.2.linux-amd64/amtool /usr/local/bin
ubuntu@monitoring-server:~$ sudo chown alertmanager:alertmanager /usr/local/bin/alertmanager
ubuntu@monitoring-server:~$ sudo chown alertmanager:alertmanager /usr/local/bin/amtool

#Create your configuration file. 

ubuntu@monitoring-server:~$ sudo vi /etc/alertmanager/alertmanager.yml

```

### Configure the Service

Your `alertmanager.yml` should look something like: 

{% code-tabs %}
{% code-tabs-item title="alertmanager.yml" %}
```yaml
route:
  group_by: [Alertname]
  # Send all notifications to me.
  receiver: realm-alert

receivers:
- name: realm-alert
  email_configs:
  - to: "<SEND_TO_EMAIL>"
    from: "<SEND_FROM_EMAIL>"
    smarthost: smtp.gmail.com:587
    auth_username: "<SEND_FROM_EMAIL_ACCOUNT>"
    auth_identity: "<SEND_FROM_EMAIL_ACCOUNT>"
    auth_password: "<SEND_FROM_EMAIL_PASSWORD>"

```
{% endcode-tabs-item %}
{% endcode-tabs %}

{% hint style="info" %}
There are a variety of built-in ways to alert with AlertManager. We use regular email here but there is also PagerDuty and Slack plugins. If you are using Gmail for development you can use [this guide to generate an app token](https://support.google.com/accounts/answer/185833?hl=en) 
{% endhint %}

### Starting the Service

Now let's add the AlertManager unit file and start the service

```bash
#Create the unit file
ubuntu@monitoring-server:~$ sudo vi /etc/systemd/system/alertmanager.service
```

Contents of the file: 

{% code-tabs %}
{% code-tabs-item title="alertmanager.service" %}
```yaml
[Unit]
Description=Prometheus Alertmanager Service
Wants=network-online.target
After=network.target

[Service]
User=alertmanager
Group=alertmanager
Type=simple
ExecStart=/usr/local/bin/alertmanager \
    --config.file /etc/alertmanager/alertmanager.yml \
    --storage.path /var/lib/alertmanager/data
Restart=always

[Install]
WantedBy=multi-user.target

```
{% endcode-tabs-item %}
{% endcode-tabs %}

Head back to the terminal: 

```bash
#Reload the daemons and enable the service
ubuntu@monitoring-server:~$ sudo systemctl daemon-reload
ubuntu@monitoring-server:~$ sudo systemctl enable alertmanager
ubuntu@monitoring-server:~$ sudo systemctl start alertmanager

```

Finally, let's add this to prometheus via our `/etc/prometheus/prometheus.yml`

```text
alerting:
  alertmanagers:
  - static_configs:
    - targets:
      - localhost:9093

```

{% hint style="warning" %}
You'll need to restart Prometheus service to pick up the added AlertManager config
{% endhint %}

## Add Health Checks

\*\*\*\*[**Here is a simple node app that will query the health endpoints that we just installed on each component service**](https://github.com/realm/realm-server-side-samples/blob/master/19-Advanced-Monitoring-Walkthrough/Health-Checks/realmHealthCheck.js)\*\*\*\*

This node application is HTTP GET request that will make sure the HTTP endpoint returns a 200. You should configure as many of these as there are Realm nodes in your distributed cluster. The health checks to the Custom Health Endpoint will tell you if there is a healthy Realm cluster in the backend \(a sync-worker with a healthy master node\). The health checks to the sync-workers will tell if you if the process is up and started. It should be configured to run as a regular cron job at an interval you specify. If the promise is rejected it should integrate into your alerting system to sound an alarm such as an email to an operations team.  


A cronjob can be set-up by using the `crontab` command, like `crontab -e` and choosing your favorite editor and then pasting in the path to your designated script into the cron file. For instance the above script could look like this on our monitoring server:

```text
 * * * * /home/ubuntu/.nvm/versions/node/v8.11.3/bin/node /home/ubuntu/healthScripts/realmHealthCheck.js
```

  
This will tell the server to execute our health check script every 5 minutes using a specific path to a node version. If you are not familiar with node you may run into path issues in which case it may be better to run the health check script in a node process monitor like pm2 and then integrate into a monitoring system like Prometheus. [We will need to edit the script a bit to integrate the `prom-client` npm package like so.](https://github.com/realm/realm-server-side-samples/blob/master/19-Advanced-Monitoring-Walkthrough/Health-Checks/promHealthProbe.js)

```bash
#First lets install the needed npm packages
ubuntu@monitoring-server:~/healthScripts$ npm install superagent
ubuntu@monitoring-server:~/healthScripts$ npm install prom-client

#Then we can start the script with a process monitor like pm2
ubuntu@monitoring-server:~/healthScripts$ pm2 start promHealthProbe.js --name=PromHealthProbe

```

  
Now we will want to point our Prometheus to scrape the newly created server which is exposing the metrics we are now collecting for the health check. Add this to your `prometheus.yml`

{% code-tabs %}
{% code-tabs-item title="prometheus.yml" %}
```yaml
  - job_name: 'superagent-healthcheck'
    scrape_interval: 5s
    static_configs:
      - targets: ['localhost:9085']

```
{% endcode-tabs-item %}
{% endcode-tabs %}

And then restart the prometheus service - you will now see the new metric showing up in Prometheus.

```bash
#Now we will want to set-up an alerting rule for this metric 
ubuntu@monitoring-server:/etc/prometheus$ sudo vi alert.rules

```

And add this: 

{% code-tabs %}
{% code-tabs-item title="alert.rules" %}
```text
groups:
- name: example
  rules:
  
    # Alert for any instance that is unreachable
  - alert: rest_healthcheck
    expr: health_reachability == 0
    labels:
      severity: page
    annotations:
      summary: "Instance {{ $labels.instance }} down"
      description: "{{ $labels.instance }} of job {{ $labels.job }} has reported an HTTP failure code"

```
{% endcode-tabs-item %}
{% endcode-tabs %}

Now in our `prometheus.yml` file add this:

{% code-tabs %}
{% code-tabs-item title="prometheus.yml" %}
```text
  rule_files:
  - '/etc/prometheus/alert.rules'

```
{% endcode-tabs-item %}
{% endcode-tabs %}

Then restart the prometheus service and if you reload the prometheus UI and append \`/alerts\` to the URL you should see the Alert loaded like below:

![Confirmation that Health Check Alert has loaded](https://lh3.googleusercontent.com/cl_2RK8ns4Lzwm_VerV1gTrceElkuchwIACwmdHIOv-ui2GbbfYdZcSo6LDAs9M_pNvtQ2-P-bUxxLpt8rIMWYWYWLfskpGzMHQupDHIQR2VuTS_VONvaRFvEg--aUZjc3sSsFOV)

### Testing our Setup

In the previous section we have set up an alert that will fire if an our HTTP endpoint returns a REST error code but when things go wrong this can fire many times and that is what Alertmanager can do - route alerts appropriately and stem the tide so that you are not spamming operators. It is a good best practice to have general defaults in your `/etc/alertmanger/alertmanger.yml` like these:

{% code-tabs %}
{% code-tabs-item title="/etc/alertmanger/alertmanger.yml" %}
```yaml
global:
  resolve_timeout: 5m
route:
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 12h
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Now if we shut down the ROS component that the health check was querying we should see this in the Alerts tab

![Active alert from our monitoring dashboard](https://lh6.googleusercontent.com/1N4HOJR1bsqK6eSvtgqx7rcDT3hbIDTE0U-Z-UqUJVZpAtzNaDN2ZAiehB6sVkLS4w7M5GBug1YPx4GuNZSM8JJCmI-7Su7_wiKHasp85KlgwKzTn2Fm3NChAGbdEmqTr6gNNwZH)

And you should get an email in your inbox  


![Email alert letting us know that ROS is unhealthy](https://lh5.googleusercontent.com/oMKAy9egIwuZOjkZbAXPEo6gZ4FTg0ysBwagbWU2pweJGS4scu2Qj47DNaIdXs0vnUVgNi-EBZwEvX-fBxYh6v3Ki5mZj89kpJ25H08IU03U-5Cm-ERRLWv5OMPDGEeyQfRmj-Uf)

## Add Liveness Probe

In the previous section we added health checks for each component server of the Realm Object Server distributed cluster. This is particularly useful to identify which service is experiencing an issue, for instance, when one of your Auth services has a stuck process. But in most cases you will also want to have an end-to-end test that tests the distributed cluster as a whole - essentially acting as a fake mobile client and testing the Realm subsystems. This is what we will implement with a Liveness Probe.

\*\*\*\*[**Add a Simple Node client that will login and Open a Realm in your system**](https://github.com/realm/realm-server-side-samples/blob/master/19-Advanced-Monitoring-Walkthrough/Health-Checks/realmProbe.js)\*\*\*\*

This node application is an end-to-end test that acts just like a mobile client that would be connecting in your production system. It should be configured to run as a regular cron job at an interval you specify. If the promise is rejected it should integrate into your alerting system and sound an alarm, such as an email to an operations team.

\*\*\*\*[**For integration into Prometheus we could use this script.** ](https://github.com/realm/realm-server-side-samples/blob/master/19-Advanced-Monitoring-Walkthrough/Health-Checks/promRealmProbe.js)\*\*\*\*

{% hint style="info" %}
Just like with the health checks in the previous section we recommend running these scripts either as a cronjob or in pm2 and then integrating with your alerting system when the system rejects the promise.
{% endhint %}

## Install Grafana

It is common practice to install another frontend on top of Prometheus for more advanced visuals, Grafana is a common choice.

```bash
#Add the repo
ubuntu@monitoring-server:~$ curl https://packagecloud.io/gpg.key | sudo apt-key add -
ubuntu@monitoring-server:~$ sudo add-apt-repository "deb https://packagecloud.io/grafana/stable/debian/ stretch main"
ubuntu@monitoring-server:~$ sudo apt-get update

#Install Grafana and Start it
ubuntu@monitoring-server:~$ sudo apt-get install grafana
ubuntu@monitoring-server:~$ sudo systemctl start grafana-server
ubuntu@monitoring-server:~$ sudo systemctl enable grafana-server

```

Once it is started you can go to port 3000 of the monitoring server IP in a web browser and login with admin/admin for the username/password  


Then select “Add New Data Source” and select Prometheus. You should be able to keep the defaults and click Save

![Adding Prometheus as a Data Source in Grafana](https://lh6.googleusercontent.com/QOFU1rRYFWuHDlkRUZLGLQ9wwo_YJN_JJjN0UTr1j1qmuFzq2SPvYtG7DLzhFhqddZG-rtHvXJot_3YmWt9U3_beltJWoWktsDIG5uShuJmT0XCrqtNPfFJi9sj2MKDzFxd1uMwY)

You can then view this information via a Grafana Dashboard:

![Simple Granana Dashboard for Monitoring ROS](https://lh6.googleusercontent.com/R6OHPbihY9BYRG3IjW3g9mK0I2HARuS9ChMSgRpgU0vxE4W29kLfpVURG1JOSk3WwnHIWURA29lccIclrLSw_xEsFtIpetFWTmCRjVjQVuC4coy1Cj21UgISdZ4D_H4VQCPYDTbF)

## Additional Monitoring Topics

Monitoring is an expansive topic, and this guide could drag on with endless rules and probes to test all sorts of different scenarios. You should monitor, test, and alert on metrics and use cases that are relevant to your app and what you want to optimize for. You may be using Realm to back a webapp and so want to measure response time or perhaps Realm is backing a middleware tier and you want to autoscale the service when the requests per second reach a certain threshold. Only you will know what is appropriate for your particular app.

### Alerting on free disk space

We do recommend always alerting on disk size however. Without free space the realm system will not be able to sync new data and may not even be able to accept new connections. Here is a sample rule set that sends a variety of increasing severity alerts as the volume continue to fill up

```text
    - name: alerts
      rules:
      - alert: TenantVolumeHalfFull
        expr: (sum by(tenant,syncLabel)(ros_sync_disk_usage_bytes) / sum by(tenant,syncLabel)(ros_sync_disk_size_bytes)) * 100 > 50
        labels:
          severity: warning
          priority: "P3"
        annotations:
          summary: Tenant volume is half full
          description: 'Tenant volume {{"{{"}} $labels.tenant {{"}}"}} is {{"{{"}} $value | printf "%.2f" {{"}}"}}% Full'
          remediation: TenantVolumeUsageHigh
      - alert: TenantVolumeAlmostFull
        expr: (sum by (tenant) (ros_sync_disk_free_bytes) / (1024 * 1024)) < 200
        for: 5m
        labels:
          severity: warning
          priority: "P2"
        annotations:
          summary: Tenant volume is almost full (< 200Mi)
          description: 'Tenant volume is almost full: {{"{{"}} $labels.tenant {{"}}"}}: {{"{{"}} $value | printf "%.2f" {{"}}"}}Mi free'
          remediation: TenantVolumeUsageHigh
      - alert: TenantVolumeFull
        expr: (sum by (tenant) (ros_sync_disk_free_bytes) / (1024 * 1024)) < 1
        for: 2m
        labels:
          severity: critical
          priority: "P1"
        annotations:
          summary: Tenant volume is full
          description: 'Tenant volume is full: {{"{{"}} $labels.tenant {{"}}"}}: {{"{{"}} $value | printf "%.2f" {{"}}"}}Mi free'
          remediation: TenantVolumeUsageHigh

```

It is also a good practice to pull statistics from your process monitoring tool which you should be using to run ROS. We recommend using pm2 and there are a variety of plugins that will pull statistics from pm2 and feed them into Prometheus. Alerting on restarts and low uptime is a must in a production system. Here are a few free packages to use:

* [PM2 Prometheus Explorer ](https://github.com/burningtree/pm2-prometheus-exporter)
* [PM2 performance monitoring using Statsd and Graphite](https://github.com/Tjatse/pm2-ant) 

  


  


  
  
  


