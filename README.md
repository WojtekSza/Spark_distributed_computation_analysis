# Spark_distributed_computation_analysis
Repository will show an example of distributed computation with usage of Spark. Servers infrastructure implemented via Azure service (Ubuntu servers).

First step is to log into Azure CLI.
```
az login
```
After sucesfull authentication next step is to create Azure resource group

```
az group create -l francecentral -n Spark
```

Next step is to create Virtual Network for 3x servers with dedicated subnets IPs.

```
az network vnet create -g Spark -n Sparkvnet --address-prefix 10.0.0.0/16 --subnet-name SRV1 --subnet-prefix 10.0.1.0/24
```
```
az network vnet subnet create --address-prefix 10.0.2.0/24 --name SRV2 --resource-group Spark --vnet-name Sparkvnet
```
```
az network vnet subnet create --address-prefix 10.0.3.0/24 --name SRV3 --resource-group Spark --vnet-name Sparkvnet
```

Virtual machines creation after setting virtual network: <br>
Server 1:  <br>
```
az vm create --name SRV1 --resource-group Spark --admin-password $adminpassword --admin-username adminuser --image UbuntuLTS --location francecentral --subnet SRV1 --subnet-address-prefix 10.0.1.0/24 --vnet-name Sparkvnet
```
Server2: <br>
```
az vm create --name SRV2 --resource-group Spark --admin-password $adminpassword --admin-username adminuser --image UbuntuLTS --location francecentral --subnet SRV2 --subnet-address-prefix 10.0.2.0/24 --vnet-name Sparkvnet
```
Server3: <br>
```
az vm create --name SRV3 --resource-group Spark --admin-password $adminpassword --admin-username adminuser --image UbuntuLTS --location francecentral --subnet SRV3 --subnet-address-prefix 10.0.3.0/24 --vnet-name Sparkvnet
```

### After sucessfull creation of 3x servers: download Spark latest version for each server and run all configuration steps:
```
https://dlcdn.apache.org/spark/spark-3.3.0/spark-3.3.0-bin-hadoop3.tgz
```
For each server download Spark:
```
wget https://dlcdn.apache.org/spark/spark-3.3.0/spark-3.3.0-bin-hadoop3.tgz
```
And then unpack
```
tar zxvf spark-3.3.0-bin-hadoop3.tgz
```
Thne unpacked spark move to /opt/ catalog to add Spark variables into $Path
```
sudo mv spark-3.3.0-bin-hadoop3 /opt/
```
```
echo "export SPARK_HOME=/opt/spark-3.3.0-bin-hadoop3" > profile.txt
echo "export PATH=$PATH:$SPARK_HOME/bin:$SPARK_HOME/sbin" > profile.txt
```
To load created .profile file to update PATH:
```
source profile.txt
```
Before runing Spark load latest updates:
```
sudo apt-get update
```
```
sudo apt-get upgrade
```
Then install programs to proper functioning of Spark:
```
sudo apt install default-jdk scala git -y
```

## If all configuration went sucessfull then type pyspark to test Spark:
![pyspark](https://github.com/WojtekSza/Spark_distributed_computation_analysis/blob/main/spark_distributed/spark.jpg)

## Spark distributed servers configurations:
Let`s set SRV1 as a master node:
```
start-master.sh -h 10.0.1.4 -p 7077
```
On the additional Ubuntu Desktop we can launch web UI to see Spark configuration:
```
sudo apt-get install openssh-client
```
```
ssh -L 8080:localhost:8080 adminuser@20.111.54.253
```
Note: 20.111.54.253 it`s public IP. If Ubuntu Destop is connected to virtual network then 10.0.1.4
```
http://localhost:8080/
```
![pyspark2](https://github.com/WojtekSza/Spark_distributed_computation_analysis/blob/main/spark_distributed/spark2.jpg)

We will launch worker node as well on the Server 1 and conect it to the master node:
```
start-worker.sh -h 10.0.1.4 -p 7078 spark://10.0.1.4:7077
```
![pyspark3](https://github.com/WojtekSza/Spark_distributed_computation_analysis/blob/main/spark_distributed/spark3.jpg)

### Testing - PI estimation 
Before additing remainig Servers 2 & 3 lets check computation time on the single server 1 <br>
Lets run example program in pyspark which will be connected to master node:
```
pyspark --master spark://10.0.1.4:7077
```
```
def inside(p):
    x, y = random.random(), random.random()
    return x*x + y*y < 1
```
```
import random
NUM_SAMPLES=100000000
count = sc.parallelize(range(0, NUM_SAMPLES)).filter(inside).count()
print("Pi is roughly %f" % (4.0 * count / NUM_SAMPLES))
```
### Results:
It took 51s to comlete PI estimation for 100000000 and runing on 1x server.
![pyspark3](https://github.com/WojtekSza/Spark_distributed_computation_analysis/blob/main/spark_distributed/spark4.jpg)
Now let`s add additional workers Server 2&3 and check if computation time will decrease:
```
start-worker.sh -h 10.0.2.4 -p 7079 spark://10.0.1.4:7077
```
```
start-worker.sh -h 10.0.3.4 -p 7080 spark://10.0.1.4:7077
```
Cluster configration of 3x servers is ready for comupation:
![pyspark5](https://github.com/WojtekSza/Spark_distributed_computation_analysis/blob/main/spark_distributed/spark5.jpg)
Let`s run the same PI estimation and see how computation time have changed:
```
def inside(p):
    x, y = random.random(), random.random()
    return x*x + y*y < 1
```
```
import random
NUM_SAMPLES=100000000
count = sc.parallelize(range(0, NUM_SAMPLES)).filter(inside).count()
print("Pi is roughly %f" % (4.0 * count / NUM_SAMPLES))
```
![pyspark5](https://github.com/WojtekSza/Spark_distributed_computation_analysis/blob/main/spark_distributed/spark6.jpg)
# Final conclusion
## Spark distributed computing reduced computation time from 51s to 20s while launched on 3x servers!!!

Last step to remove all created Azure resources:
```
az group delete --name Spark
```
