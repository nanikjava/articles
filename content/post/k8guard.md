---
date: "2019-12-01"
title: "Deploying k8guard (Work in progress)"
author : "Nanik Tolaram (nanikjava@gmail.com)" 

---

<h1>Deploying k8guard</h1>

* Make sure **minikube** and **kubectl** is in PATH
* Use the following command to start  (v1.11.10  is the lowest version possible that will work with minikube master branch)

    > minikube start --memory 4096 --kubernetes-version v1.11.10 
     
    > eval $(minikube docker-env)

    > make build-deploy-minikube
    
* Deploy using the following commands

	> kubectl apply -f ./minikube/report/k8guard-report-secrets.yaml.EXAMPLE 
    
	> kubectl apply -f ./minikube/action/k8guard-action-secrets.yaml.EXAMPLE

<h1>k8guard-action</h1>

{{< highlight text >}}
k8guard-action-configmap.yaml
         |
         |
         |
    k8guard-action-deployment.yam
             |
         |
         |
          k8guard-action-secrets.yaml.EXAMPLE
{{< /highlight >}}

* File **k8guard-action-deployment.yaml** contains deployment information for the k8guard-action app. It uses docker image called **local/k8guard-action** which is build doing the above mentioned step.
* The _env:_ section declared the env variables that will be populated in the EXPORT section of the bash. The value is obtained from the **k8guard-action-configmap.yaml**
* The **k8guard-action-configmap.yaml** contains the config map for the different variables
* The following environment variables are found inside the k8guard-action container. These values are populate via the .env file

{{< highlight text >}}
export HOME='/root'
export HOSTNAME='k8guard-action-deployment-67c4db498-j4hvd'
export K8GUARD_ACTION_CASSANDRA_CAPATH=''
export K8GUARD_ACTION_CASSANDRA_HOSTS='k8guard-cassandra-service.default.svc.cluster.local:9042'
export K8GUARD_ACTION_CASSANDRA_KEYSPACE='k8guardkeyspace'
export K8GUARD_ACTION_CASSANDRA_PASSWORD='cassandra'
export K8GUARD_ACTION_CASSANDRA_SSL_HOST_VALIDATION='false'
export K8GUARD_ACTION_CASSANDRA_USERNAME='cassandra'
export K8GUARD_ACTION_DRY_RUN='false'
export K8GUARD_ACTION_DURATION_BETWEEN_CHAT_NOTIFICATIONS='30s'
export K8GUARD_ACTION_DURATION_BETWEEN_NOTIFYING_AGAIN='24h'
export K8GUARD_ACTION_DURATION_VIOLATION_EXPIRES='120h'
export K8GUARD_ACTION_HIPCHAT_BASE_URL=''
export K8GUARD_ACTION_HIPCHAT_ROOM_ID=''
export K8GUARD_ACTION_HIPCHAT_TAG_NAMESPACE_OWNER='true'
export K8GUARD_ACTION_HIPCHAT_TOKEN='REPLACE_ME'
export K8GUARD_ACTION_SAFE_MODE='true'
export K8GUARD_ACTION_SMTP_FALLBACK_SEND_TO='REPLACE@REPLACE_WITH_DOMAIN.COM'
export K8GUARD_ACTION_SMTP_PORT='25'
export K8GUARD_ACTION_SMTP_SEND_FROM='DO_NOT_REPLY@REPLACE_WITH_DOMAIN.COM'
export K8GUARD_ACTION_SMTP_SEND_TO_NAMESAPCE_OWNER='true'
export K8GUARD_ACTION_SMTP_SERVER=''
export K8GUARD_ACTION_VIOLATION_EMAIL_FOOTER=''
export K8GUARD_ACTION_WARNING_COUNT_BEFORE_ACTION='4'
export K8GUARD_CASSANDRA_CREATE_KEYSPACE='true'
export K8GUARD_CASSANDRA_CREATE_TABLES='true'
export K8GUARD_CASSANDRA_SERVICE_PORT='tcp://10.99.115.177:9042'
export K8GUARD_CASSANDRA_SERVICE_PORT_9042_TCP='tcp://10.99.115.177:9042'
export K8GUARD_CASSANDRA_SERVICE_PORT_9042_TCP_ADDR='10.99.115.177'
export K8GUARD_CASSANDRA_SERVICE_PORT_9042_TCP_PORT='9042'
export K8GUARD_CASSANDRA_SERVICE_PORT_9042_TCP_PROTO='tcp'
export K8GUARD_CASSANDRA_SERVICE_SERVICE_HOST='10.99.115.177'
export K8GUARD_CASSANDRA_SERVICE_SERVICE_PORT='9042'
export K8GUARD_CLUSTER_NAME='minikube'
export K8GUARD_DISCOVER_SERVICE_PORT='tcp://10.103.122.8:3000'
export K8GUARD_DISCOVER_SERVICE_PORT_3000_TCP='tcp://10.103.122.8:3000'
export K8GUARD_DISCOVER_SERVICE_PORT_3000_TCP_ADDR='10.103.122.8'
export K8GUARD_DISCOVER_SERVICE_PORT_3000_TCP_PORT='3000'
export K8GUARD_DISCOVER_SERVICE_PORT_3000_TCP_PROTO='tcp'
export K8GUARD_DISCOVER_SERVICE_SERVICE_HOST='10.103.122.8'
export K8GUARD_DISCOVER_SERVICE_SERVICE_PORT='3000'
export K8GUARD_KAFKA_ACTION_TOPIC='k8guard-to-action-k8s-lab'
export K8GUARD_KAFKA_BROKERS='k8guard-kafka-service.default.svc.cluster.local:9092'
export K8GUARD_KAFKA_SERVICE_PORT='tcp://10.106.168.209:2181'
export K8GUARD_KAFKA_SERVICE_PORT_2181_TCP='tcp://10.106.168.209:2181'
export K8GUARD_KAFKA_SERVICE_PORT_2181_TCP_ADDR='10.106.168.209'
export K8GUARD_KAFKA_SERVICE_PORT_2181_TCP_PORT='2181'
export K8GUARD_KAFKA_SERVICE_PORT_2181_TCP_PROTO='tcp'
export K8GUARD_KAFKA_SERVICE_PORT_9092_TCP='tcp://10.106.168.209:9092'
export K8GUARD_KAFKA_SERVICE_PORT_9092_TCP_ADDR='10.106.168.209'
export K8GUARD_KAFKA_SERVICE_PORT_9092_TCP_PORT='9092'
export K8GUARD_KAFKA_SERVICE_PORT_9092_TCP_PROTO='tcp'
export K8GUARD_KAFKA_SERVICE_SERVICE_HOST='10.106.168.209'
export K8GUARD_KAFKA_SERVICE_SERVICE_PORT='2181'
export K8GUARD_KAFKA_SERVICE_SERVICE_PORT_KAFKA='9092'
export K8GUARD_KAFKA_SERVICE_SERVICE_PORT_ZK='2181'
export K8GUARD_LOG_LEVEL='debug'
export K8GUARD_MEMCACHED_SERVICE_PORT='tcp://10.100.96.55:11211'
export K8GUARD_MEMCACHED_SERVICE_PORT_11211_TCP='tcp://10.100.96.55:11211'
export K8GUARD_MEMCACHED_SERVICE_PORT_11211_TCP_ADDR='10.100.96.55'
export K8GUARD_MEMCACHED_SERVICE_PORT_11211_TCP_PORT='11211'
export K8GUARD_MEMCACHED_SERVICE_PORT_11211_TCP_PROTO='tcp'
export K8GUARD_MEMCACHED_SERVICE_SERVICE_HOST='10.100.96.55'
export K8GUARD_MEMCACHED_SERVICE_SERVICE_PORT='11211'
export K8GUARD_REPORT_SERVICE_PORT='tcp://10.105.186.144:3001'
export K8GUARD_REPORT_SERVICE_PORT_3001_TCP='tcp://10.105.186.144:3001'
export K8GUARD_REPORT_SERVICE_PORT_3001_TCP_ADDR='10.105.186.144'
export K8GUARD_REPORT_SERVICE_PORT_3001_TCP_PORT='3001'
export K8GUARD_REPORT_SERVICE_PORT_3001_TCP_PROTO='tcp'
export K8GUARD_REPORT_SERVICE_SERVICE_HOST='10.105.186.144'
export K8GUARD_REPORT_SERVICE_SERVICE_PORT='3001'
export KUBERNETES_PORT='tcp://10.96.0.1:443'
export KUBERNETES_PORT_443_TCP='tcp://10.96.0.1:443'
export KUBERNETES_PORT_443_TCP_ADDR='10.96.0.1'
export KUBERNETES_PORT_443_TCP_PORT='443'
export KUBERNETES_PORT_443_TCP_PROTO='tcp'
export KUBERNETES_SERVICE_HOST='10.96.0.1'
export KUBERNETES_SERVICE_PORT='443'
export KUBERNETES_SERVICE_PORT_HTTPS='443'
export OLDPWD='/var/log'
export PATH='/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin'
export PWD='/var'
export SHLVL='1'
export TERM='xterm'
{{< /highlight >}}
