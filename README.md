# Spark_distributed_computation_analysis
Repository will show an example of distributed computation with usage of Spark. Servers infrastructure implementd via Azure service.

First step is to log into Azure CLI.
```
az login
```
After sucesfull authentication next step is to create Azure resource group

```
az group create -l francecentral -n spark
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

### After sucessfull creation of 3x servers: download Spark latest version:
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
