# Kubeadm-ready Vagrant

## Setting up

```
vagrant plugin install vagrant-hostmanger
```

### Vagrant Up

```
vagrant up
```

### Vagrant Up Nodes separately

```
vagrant up master
vagrant up node-1
vagrant up node-1
```

### Connecting to the nodes

```
vagrant ssh master
vagrant ssh node-1
```


## Trouble Shooting


1. For people encounterred the follow issue:

```
The machine with the name 'default' was not found configured for this Vagrant
environment.
```

Run the following command to prune the status:

```
vagrant global-status --prune
```


## Other Misc Setup instructions in Mac

```
export KUBECONFIG=/Users/joychak/joy/workspace/repo/vagrant-kubeadm/kube.config
sudo route add 10.96.0.1 gw 192.168.77.2
kubectl get  services --all-namespaces -o wide

docker network create --subnet=172.18.0.0/16 mynet

docker run -d -p 5000:5000 -e REGISTRY_HTTP_ADDR=0.0.0.0:5000 --restart=always --name docker-registry registry:2
docker run -d --add-host="localhost:10.0.2.2" -p 5000:5000  --restart=always --name docker-registry registry:2

cd /Developer/spark-2.3.0-bin-hadoop2.7/
docker build -t spark:latest -f kubernetes/dockerfiles/spark/Dockerfile .
docker tag spark localhost:5000/spark
docker push localhost:5000/spark


docker build -t kdc-admin:latest -f Dockerfile .
docker tag kdc-admin localhost:5000/kdc-admin
docker push localhost:5000/kdc-admin


docker build -t kdc-client:latest -f Dockerfile .
docker tag kdc-client localhost:5000/kdc-client
docker push localhost:5000/kdc-client

kubectl exec -it kdc-client bash
apt-get install dnsutils telnet nmap net-tools
apt-cache search telnet
nslookup kdc-admin-srv
nmap -A 10.97.33.36/32 -p 88
netstat -an

env KRB5_TRACE=/dev/stdout kadmin -p kadmin/admin@TESTKDC.COM -w Kdc123! -q list_principals
kinit kadmin/admin@TESTKDC.COM

kadmin -p kadmin/admin@TESTKDC.COM
add_principal testuser1@TESTKDC.COM
getprinc testuser1@TESTKDC.COM

ktuitl
addent -password -p testuser1@TESTKDC.COM -k -1 -e aes256-cts-hmac-sha1-96
wkt /home/testuser1.keytab

========================================

vagrant ssh master
vagrant ssh node-1
vagrant ssh node-2
vagrant ssh node-3
***** Do the following for each node *****
sudo vi /etc/default/docker
DOCKER_OPTS="--config-file=/etc/docker/daemon.json"

sudo vi /etc/docker/daemon.json
{
  "insecure-registries" : ["10.0.2.2:5000"]
}

sudo service docker restart
sudo docker pull 10.0.2.2:5000/spark

========================================

sudo apt install -y nfs-common => all nodes

Node-5:
sudo vi /etc/exports
Add => /pv *(rw,no_root_squash,no_subtree_check)
sudo exportfs -r
showmount -e => To check

========================================

kubectl create serviceaccount spark
kubectl create clusterrolebinding spark-role --clusterrole=edit --serviceaccount=default:spark --namespace=default


~/joy/workspace/repo/kubeadm-vagrant $ kubectl cluster-info
Kubernetes master is running at https://192.168.26.10:6443
KubeDNS is running at https://192.168.26.10:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy



vagrant scp /Developer/spark-2.3.0-bin-hadoop2.7 w3:/home/vagrant/


spark-submit \
--master k8s://https://192.168.26.10:6443 \
--deploy-mode cluster \
--name spark-pi \
--class org.apache.spark.examples.SparkPi \
--conf spark.driver.cores=1 \
--conf spark.driver.memory=512m \
--conf spark.executor.cores=1 \
--conf spark.executor.instances=1 \
--conf spark.executor.memory=512m \
--conf spark.kubernetes.authenticate.driver.serviceAccountName=spark \
--conf spark.kubernetes.container.image=10.0.2.2:5000/spark \
local:///opt/spark/examples/jars/spark-examples_2.11-2.3.1.jar

spark-submit \
--master k8s://https://192.168.26.10:6443 \
--deploy-mode cluster \
--name spark-pi \
--class org.apache.spark.examples.SparkPi \
--conf spark.driver.cores=1 \
--conf spark.driver.memory=256m \
--conf spark.executor.cores=1 \
--conf spark.executor.instances=1 \
--conf spark.executor.memory=512m \
--conf spark.kubernetes.container.image=10.0.2.2:5000/spark \
local:///opt/spark/examples/jars/spark-examples_2.11-2.3.0.jar

kubectl get pods --all-namespaces --show-all -o wide
kubectl delete --all pods --namespace=default
kubectl describe pod spark-pi-4adf2c21234c301aa4a92b687d6597a9-driver

systemctl restart kubelet

===================================================================

spark-submit \
--master local \
--class com.datalogs.dataset.SampleDatasetCsvProcessor \
/Users/joychak/joy/workspace/repo/data-ingestion/spark-processor-dataset/target/scala-2.11/ingestion-spark-processor-dataset-assembly-1.1.0.jar \
--input-dir /Users/joychak/joy/workspace/repo/data-ingestion/src/test/data/input \
--output-dir /Users/joychak/joy/workspace/repo/data-ingestion/src/test/data/output \
--archive-dir /Users/joychak/joy/workspace/repo/data-ingestion/src/test/data/archive \
--trap-dir /Users/joychak/joy/workspace/repo/data-ingestion/src/test/data/trap \
--state-store-dir /Users/joychak/joy/workspace/repo/data-ingestion/src/test/data/state-store \
--batch-id 20181231080000 \
--duration 525600 \
--prepare-n-days 365 \
--dataSourceName CSV-DATA

mkdir ./src
mkdir ./src/test
mkdir ./src/test/data
mkdir ./src/test/data/input
mkdir ./src/test/data/output
mkdir ./src/test/data/trap
mkdir ./src/test/data/archive
mkdir ./src/test/data/state-store
mkdir ./src/test/data/state-store/data
touch ./src/test/data/state-store/data/part-00000
touch ./src/test/data/state-store/data/_SUCCESS
cp ./test-data/sample-data-csv-file1.csv ./src/test/data/input


mv src/test/data/archive/20181231080000/sample-data-csv-file1.csv src/test/data/input/
rm -r -f src/test/data/archive/20181231080000
rm -r -f src/test/data/output/*
rm src/test/data/output/._*
rm src/test/data/output/_*
rm -r -f src/test/data/state-store/ARCHIVES
rm src/test/data/state-store/data/._SUCCESS.crc
rm src/test/data/state-store/data/.part-00000.crc
rm src/test/data/state-store/data/part*
touch src/test/data/state-store/data/part-00000
rm -r -f src/test/data/trap/20181231080000
rm -r -f src/test/data/trap/*



val sqlContext = new org.apache.spark.sql.SQLContext(sc)
val df = sqlContext.read.parquet("/Users/joychak/joy/workspace/repo/data-ingestion/src/test/data/output/eventDate=2018-04-02/batchId=20181231080000/*")
df.show()

==============================================================


./build-image.sh
docker network create --driver=bridge hadoop
./start-container.sh

vi /etc/hosts => 172.17.0.4 hadoop-slave1

hadoop fs -mkdir /user
hadoop fs -put run-wordcount.sh /user

hadoop fs -ls hdfs://localhost:8020/user/run-wordcount.sh

docker exec -it hadoop-master bash
./start-hadoop.sh
./run-wordcount.sh

docker run -itd --hostname dns.joychak --name dns-proxy-server -p 5380:5380 \
-v /var/run/docker.sock:/var/run/docker.sock \
-v /etc/resolv.conf:/etc/resolv.conf \
defreitas/dns-proxy-server

docker run --hostname dns.mageddo --name dns-proxy-server -p 5380:5380 \
-v /var/run/docker.sock:/var/run/docker.sock \
-v /etc/resolv.conf:/etc/resolv.conf \
defreitas/dns-proxy-server

====================================================================
wget http://apache.claz.org/hadoop/common/hadoop-2.8.4/hadoop-2.8.4.tar.gz
tar xvf hadoop-2.8.4.tar.gz

# added path for Hadoop
export PATH=/home/vagrant/hadoop/hadoop-2.8.4/bin:$PATH
export HADOOP_HOME=/home/vagrant/hadoop/hadoop-2.8.4
export HADOOP_CONF_DIR=/home/vagrant/hadoop/hadoop-2.8.4/conf

apt-get install -y openjdk-8-jdk
====================================================================
```
