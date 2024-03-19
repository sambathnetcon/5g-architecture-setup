# 5G - ARCHITECTURE SETUP

# AETHER – CONTROL NODE: 

Haswell CPU (or newer), with at least 4 CPU cores and 16GB RAM.
    • Clean install of Ubuntu 20.04 or 22.04 Server ISO(or later) kernel.
For example, something like an Intel N100 is likely more than enough to get started.
Pre-requestie:

```
     sudo apt update
     sudo apt upgrade 
     sudo apt install net-tools 
     sudo apt install iptables-persistent
     sudo apt install netplan.io
```
 
# Check whether the firewall is disable :

```
     sudo systemctl status ufw.service 
     sudo systemctl stop ufw.service 
     sudo systemctl disable ufw.service 
 ```
 
# Check whether the network service is enabled:

``` 
     systemctl status systemd-networkd.service
```

# Removing python3 and Installing python 3.8 v-env:

``` 
     sudo apt autoremove python3.10
     python3 –version
```

# Installing Pipx and Python 3.8 - venv :

```
     sudo apt install pipx
     sudo apt update
     sudo apt install ubuntu-advantage-tools
     sudo apt install software-properties-common
     sudo add-apt-repository ppa:deadsnakes/ppa
     sudo apt install python3.8-venv
```

Note that Aether assumes Ubuntu Server (as opposed to Ubuntu Desktop), the main implication being that it uses systemd-networkd rather than Network Manager to manage network settings. It is possible to work around this requirement, but be aware that doing so may impact the Ansible playbook for installing SD-Core.
OnRamp depends on Ansible, which you can install on your server as follows:

```
$ pipx install --include-deps ansible
$ pipx ensurepath
$ sudo apt-get install sshpass
$ sudo apt install ansible
```

Once installed, displaying the Ansible version number should result in output similar to the following:

```
$ ansible --version
$ ansible-galaxy collection install kubernetes.core
```

Note that a fresh install of Ubuntu may be missing other packages that you need (e.g., git, curl, make), but you will be prompted to install them as you step through the Quick Start sequence.

# Download Aether OnRamp
Once ready, clone the Aether OnRamp repo on this target deployment server:

```
$ git clone --recursive https://github.com/opennetworkinglab/aether-onramp.git
$ cd aether-onramp
```

# To Install K8S Cluster:

```
$make aether-k8s-install
```

# Installing Open5gs – Core Network:

https://artifacthub.io/packages/helm/adaptivenetlab/open5gs – Open5gs -> helms chart packages
Add Helm repo and install in the K8S cluster 

```
$ helm repo add adaptivenetlab https://adaptivenetlab.github.io/charts/
$ helm install my-open5gs adaptivenetlab/open5gs --version 1.0.3 -n open5gs
```

# Add sim details in the Database of open5gs UI:

• Change the Web-ui service into NodePort to access outside the K8S cluster (By editing Type : NodePort instead of ClusterIP in SVC)

• Add the required details in Open5gs Ui to stored the sim details in the Open5gs mongo-database .

• Required details like IMSI ,Ki and OPC ,which is added to the database.

# GNODEB – CABLE-FREE RAN:

# Connect Core network to Gnb(cable free) :

After installing open5gs on Aether-Rancher , check whether the pod ,service are successfully deployed on the K8S cluster.
On the K8S Cluster ,service of AMF will be in NodePort for Internal to External traffic.
Copy the Service NodePort IP of the AMF and saved it for to use in Gnb Configuration File(enb.cfg).
      
  #    5G NR GNB :
• Check whether the Gnb is connected in the same network with Core network and reachable to all the micro service running the K8S cluster by adding default route to the Gnb machine .
1. Verify Network Connectivity: Ensure that the Gnb machine is connected to the same network as the Core network. You can do this by checking the network configuration on the Gnb machine and comparing it with the network configuration of the Core network. Ensure that both networks have compatible IP address ranges and subnet masks.

 2. Check Default Gateway: Verify that the Gnb machine has a default gateway configured. The default gateway is the IP address of the router or gateway device that connects the Gnb machine to the wider network, including the Core network. You can check the network configuration on the Gnb machine to confirm the presence of a default gateway.(Example :- sudo ip r add 0.0.0.0/0 via <Core static ip>) 
      
3. Validate Reachability: To ensure that the Gnb machine is reachable by all the microservices running on the K8S cluster, you can perform the following steps:
      
a. From a microservice within the K8S cluster, attempt to ping the IP address of the Gnb machine. If the ping is successful, it indicates that the Gnb machine is reachable from within the K8S cluster.
      
b. If the Gnb machine has a firewall enabled, ensure that it allows incoming connections from the IP addresses or subnets used by the K8S cluster. Check the firewall configuration on the Gnb machine and make any necessary adjustments to allow incoming connections.
      
c. Verify Routing: Check the routing tables on the Gnb machine to ensure that it has a route to reach the IP addresses used by the K8S cluster. This includes the IP addresses assigned to the microservices running on the cluster. If the Gnb machine does not have a route to the K8S cluster's IP addresses, you may need to add a specific route or adjust the default route configuration on the Gnb machine.
• Switch to the root user to edit the configuration files of root/lte.../config/enb.cfg by adding the AMF IP address and Change the gtpu address of Gnb IP address (static ip where we set) ,save and exit.
• Check whether the Gnb and Core are connected by Cable free UI ,where Gnap is connected to the service IP of AMF or Log file generate in gnb0.log file(path nano /tmp/gnb0.log). If Gnb is connected ,then check the (UE device which the sim is attached) Cable free  outdoor CPE has connected to the Gnb in Cable free UI.
    
• Check whether the CPE/5G MODEM(UE) devices has attached SIM .UPF (Core network – NFV) will assign an IP address for CPE(UE) by tunnel created in UPF. 

1 . The CPE/5G MODEM can have an IP address assigned to it by the UPF (User Plane Function) in the core network, which is a component of the 5G network architecture based on NFV (Network Function Virtualization).

2 . When a CPE/5G MODEM connects to the network, it establishes a communication link with the UPF via a tunnel. This tunnel is created between the CPE/5G MODEM and the UPF to facilitate the transmission of data packets.

3.Once the tunnel is established, the CPE/5G MODEM can communicate with the core network and other external networks using the assigned IP address. The UPF acts as a gateway for the CPE/5G MODEM, forwarding the data packets between the CPE/5G MODEM and the appropriate network destinations.

# USER END DEVICE : {CABLE FREE CPE / 5G QUECTEL MODEM – MINI PC – STANDALONE NODE (K8S -RANCHER)}:

# Cable free – CPE :

Once, the CPE received internet through SIM via Wireless Connection (UPF Tunnel – 10.45.0.X), CPE default gateway 192.168.10.10, Connect the Mini PC as LAN port on the CPE Device with Static IP of 192.168.10.40.

# 5G Quectel Modem :

https://ubuntu.com/core/docs/networkmanager/configure-cellular-connections – follow these steps to enable cellular connection in ubuntu 22.04 server,

```
$ sudo apt install network-manager
$ snap install modem-manager
$ sudo mmcli -L
$ sudo mmcli -m 0
$ nmcli c add type gsm ifname wwx0ef0fc69011f con-name NETCON apn  internet 
$ nmcli r wwx0ef0fc69011f on
```

# Mini PC (Aether – Rancher{Standalone}):

Similar steps followed in Aether – Control Node to bulit K8S cluster by connected through Cable free CPE or 5G QUECTEL,

# Pre-requestie:

```
     sudo apt update
     sudo apt upgrade 
     sudo apt install net-tools 
     sudo apt install iptables-persistent
     sudo apt install netplan.io
 ```

# Check whether the firewall is disable :

```
     sudo systemctl status ufw.service 
     sudo systemctl stop ufw.service 
     sudo systemctl disable ufw.service 
  ```

# Check whether the network service is enabled:

 ```
     systemctl status systemd-networkd.service
```

# Removing python3 and Installing python 3.8 v-env:

 ```
     sudo apt autoremove python3.10
     python3 –version
```

# Installing Pipx and Python 3.8 - venv :

```
     sudo apt install pipx
     sudo apt update
     sudo apt install ubuntu-advantage-tools
     sudo apt install software-properties-common
     sudo add-apt-repository ppa:deadsnakes/ppa
     sudo apt install python3.8-venv
```

Note that Aether assumes Ubuntu Server (as opposed to Ubuntu Desktop), the main implication being that it uses systemd-networkd rather than Network Manager to manage network settings. It is possible to work around this requirement, but be aware that doing so may impact the Ansible playbook for installing SD-Core.
OnRamp depends on Ansible, which you can install on your server as follows:

```
$ pipx install --include-deps ansible
$ pipx ensurepath
$ sudo apt-get install sshpass
$ sudo apt install ansible
```

Once installed, displaying the Ansible version number should result in output similar to the following:

```
$ ansible --version
$ ansible-galaxy collection install kubernetes.core
```

Note that a fresh install of Ubuntu may be missing other packages that you need (e.g., git, curl, make), but you will be prompted to install them as you step through the Quick Start sequence.

# Download Aether OnRamp
Once ready, clone the Aether OnRamp repo on this target deployment server:

```
$ git clone --recursive https://github.com/opennetworkinglab/aether-onramp.git
$ cd aether-onramp
```

# To Install K8S Cluster:

```
$make aether-k8s-install
```




