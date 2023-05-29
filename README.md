# 5giitjammu
5G network - using UERANSIM and Open5GS

![Screenshot 2023-05-29 153657](https://github.com/samareshbera/5giitjammu/assets/134690717/65844f36-9a46-439d-82af-ccd6f9afb25d)


a) UE (User Equipment)  
b) gNB (gNodeB) 

Firstly, we have to create the NAT Network for all the virutal machines, name it as a 5G network with IP address- 10.0.2.0/24

Ensure that you have all the updated packages installed in Ubuntu.

`sudo apt update`  
`sudo apt upgrade -y`  

Install the latest version of UERAMSIM on the UE and gNB. The same can be done by cloning the repository.

`git clone https://github.com/aligungr/UERANSIM`  

Other packages that are required can be installed by these commands:

`sudo apt install make`  
`sudo apt install gcc`  
`sudo apt install g++`  
`sudo apt install libsctp-dev lksctp-tools`  
`sudo apt install iproute2`  
`sudo snap install cmake --classic` 

File configuration  

    (UERANSIM/config/open5gs-ue.yaml)

    supi: 'imsi-901700000000001'  

    mcc: '901'  
    mnc: '70'  

    gnbSearchList:  
      - 10.0.2.13  

    (UERANSIM/config/open5gs-gnb.yaml)

    mcc: '901'  
    mnc: '70'  

    linkIp: 10.0.2.11  # gNB's local IP address for Radio Link Simulation (Usually same with local IP)  
    ngapIp: 10.0.2.11  # gNB's local IP address for N2 Interface (Usually same with local IP)  
    gtpIp:  10.0.2.11  # gNB's local IP address for N3 Interface (Usually same with local IP)  

    amfConfigs:  
      - address: 10.0.2.13  
        port: 38412  



# Installing Open5GS for CPF and UPF

The Open5GS is an open source implementation of 5G mobile core network.  

Unlike previous cellular networks 5G Core network architecture designed with Network Function Virtualization and Software Defined Networking.

In this scenario, the following IP addresses are assigned.

UE = 10.0.2.15
gNB = 10.0.2.11
Open5GS = 10.2.0.13

# Machine Setup

## Install Open5GS

Firstly, ensure that you have the latest packages installed.

`sudo apt update`  
`sudo apt upgrade`  

Open5GS can be installed with the following commands on Ubuntu 20.04 or higher.

`sudo apt install software-properties-common`  
`sudo add-apt-repository ppa:open5gs/latest`  
`sudo apt update`  
`sudo apt install open5gs`  


## Configure Open5GS

A few changes need to be made to the config files to deploy the network and make it run.

`sudo nano /etc/open5gs/amf.yaml`  

    ngap:
     - addr: 10.0.2.5

    guami:
       - plmn_id:
            mcc: 901
            mnc: 70

    tai:
       - plmn_id:
            mcc: 901
            mnc: 70

    plmn_support:
       - plmn_id:
            mcc: 901
            mnc: 70


`sudo nano /etc/open5gs/upf.yaml`  

    gtpu:
        - addr: 10.2.0.5


After making the changes, restart Open5GS.

`sudo systemctl restart open5gs-amfd`  
`sudo systemctl restart open5gs-upfd`  


Run the following commands in different terminals windows to see the logs.

`sudo tail -f /var/log/open5gs/amf.log`  
`sudo tail -f /var/log/open5gs/upf.log`  


## NAT Port Forwarding

`sudo sysctl -w net.ipv4.ip_forward=1`  
`sudo iptables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE`  
`sudo systemctl stop ufw`  
`sudo iptables -I FORWARD 1 -j ACCEPT`  


## Building the WebUI of Open5GS

Node.js is required to build the WebUI of Open5GS.

`sudo apt update`
`sudo apt install curl`  
`curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -`  
`sudo apt install nodejs` 
`curl -fsSL https://open5gs.org/open5gs/assets/webui/install | sudo -E bash -`  
`git clone https://github.com/open5gs/open5gs.git`  


Install the dependencies to run WebUI

`cd webui`  
`npm ci`  

The WebUI runs as an npm script.

`npm run dev`  

Server listening can be changed by setting the environment variable HOSTNAME or PORT as below.

`HOSTNAME=10.0.2.13 npm run dev`  
`PORT=3000 npm run dev`


Fire up a web browser and login to http://10.0.2.13:3000

Username: admin
Password: 1423

Add subscriber info.

IMSI: 901700000000001

Click SAVE Button

## gNB setup
(start gnb with open5gc-gnb.yaml config file)

`./build/nr-gnb -c config/open5gs-gnb.yaml`  

## UE setup
(start gnb with open5gc-ue.yaml config file)

`sudo ./build/nr-ue -c config/open5gs-ue.yaml`  

## TEST 5G NEtwork
(ping command bind direcly to uesimtun0)
`ping -I uesimtun0 google.com`  


