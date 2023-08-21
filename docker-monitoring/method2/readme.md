### Prerequisite:-

* Basic overview of Prometheus & Grafana
* AWS account
* A server with Docker installed on it and some containers also



## Step 1:- Create a server

Create Ubuntu 20.04 server in AWS. In this server we will install Prometheus & Grafana

For Prometheus & Grafana server we need to open 9090 & 3000 port & port 22 for SSH

## Step 2:- Install Prometheus

Download and install Prometheus on the server from this link.

In order to start the Prometheus we need to run ./prometheus in the Prometheus directory

We can view UI of Prometheus at <Public IP:9090>

## Step 3:- Install Grafana

Download and install Grafana on the same server on which we have installed Prometheus from this link

In order to start the Grafana we need to run ./bin/grafana-server command from the Grafana directory

When you open UI of Grafana for the first time it will ask for Username and Password. The default Username & Password is admin. You can change the password afterwards

We can view UI of Grafana at <Public IP: 3000>

## Step 4:- Modify the daemon.json file

In order for Prometheus to gather the metrics of the docker we need to add below stanza in the /etc/docker/daemon.json file in the docker server.

```
{
  "metrics-addr" : "127.0.0.1:9323",
  "experimental" : true
}
```
9323 is the port from where Prometheus will gather the metrics

After adding above stanza we need to restart the docker using below command
```
systemctl restart docker
```

We can view the different metrics at the <Public IP:9393/metrics>

If the above setting is not working then we can also add below stanza in the daemon.json file.
```
{
  "metrics-addr" : "0.0.0.1:9323",
  "experimental" : true
}
```

## Step 5:- Modify the configuration file of Prometheus

Prometheus manages the configuration file named prometheus.yml in which we can define various configuration like alerts, scrape_configs etc.
In order for Prometheus to gather the metrics of the Docker node we need to define below code in prometheus.yml under the scrape_configs stanza
```
- job_name: "Docker Job"
  static_configs:
    - targets: ["<Public IP of Docker Node:9323"]
```

After adding the code we can check our node as a target in the Prometheus. It will take some time for Prometheus to gather all the metrics and display the state of node to Up.

## Step 6:- Add data source in Grafana

Now in Grafana we need to add Prometheus as the data source
After that, in the URL section of we need to enter the IP address of the Prometheus server and 9090 port


## Step 7:- Create a Dashboard

In Grafana we can create various kinds of dashboards as per our need.
So, for this tutorial we will create a dashboard with 3 panels.
First panel will display the number of running containers, Second panel will display the numbers of paused containers and the last panel will display the number of stopped containers.
Click on create -> Add Panel
In the Metrics browser section add below query. The below query will fetch the number of stopped containers.
```
engine_daemon_container_states_containers{state="stopped"}
```

You can change the visualization to stat
Click on Add Panel to add second panel
In the Metrics browser section add below query. The below query will fetch the number of paused containers

```
engine_daemon_container_states_containers{state="paused"}
```

Click on Add Panel to add second panel
In the Metrics browser section add below query. The below query will fetch the number of running containers
```
engine_daemon_container_states_containers{state="running"}
```
