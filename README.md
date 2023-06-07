
5G network - using UERANSIM and Open5GS

![Screenshot 2023-05-29 153657](https://github.com/samareshbera/5giitjammu/assets/134690717/65844f36-9a46-439d-82af-ccd6f9afb25d)


 UE (User Equipment)  
 gNB (gNodeB) 

Firstly, create a NAT Network for all the virutal machines.
Name it as `5GNetwork` with the IP address as `10.0.2.0/24`.

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

## File Configuration  

UERANSIM/config/open5gs-ue.yaml

    supi: 'imsi-901700000000001'  

    mcc: '901'  
    mnc: '70'  

    gnbSearchList:  
      - 10.0.2.13  

UERANSIM/config/open5gs-gnb.yaml

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
`git clone https://github.com/open5gs/open5gs.git`  


Install the dependencies to run WebUI

`cd webui`  
`npm ci`  

The WebUI runs as an npm script.

`npm run dev`  

Server listening can be changed by setting the environment variable HOSTNAME or PORT as below.

`HOSTNAME=10.0.2.13 npm run dev`  
`PORT=3000 npm run dev`

## Adding a UE subscriber

Fire up a web browser and login to http://10.0.2.13:3000 (for `npm run dev`: `127.0.0.1:3000`)

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

## Testing 5G Network from UE
(ping command bind direcly to uesimtun0)  
`ping -I uesimtun0 google.com`  

  
    

# 5G Message Flow Monitoring using Wireshark

Wireshark is used to monitor the packet flows between the machines.  

## Packets Flow

![Screenshot 2023-06-07 103228](https://github.com/samareshbera/5giitjammu/assets/134690717/3b2f1ed6-a662-4acc-ab2f-61fbe4a16a9e)

Restart the `amfd` and `upfd` services.  

It is seen that the PFCP session is established.  

PFCP Association Setup Request is sent from SMF to UPF. (3)  

![5](https://github.com/samareshbera/5giitjammu/assets/96954630/94aa6b2a-9adc-49af-a8c8-891042df8e9a)

UPF replies back to SMF with a PFCP Association Setup Response. (4)  

![6](https://github.com/samareshbera/5giitjammu/assets/96954630/f1a6d189-16cc-4a5a-a48a-fe6fade45a5b)


The gNB (10.0.2.11) sends an NGSetupRequest to Open5GS(10.0.2.13). (2)  

![Screenshot 2023-06-07 104228](https://github.com/samareshbera/5giitjammu/assets/134690717/5ecb4dc1-5f66-40af-9df0-3b90f2cd84ab)  

Open5GS (10.0.2.13) sends an acknowledgement to gNB (10.0.2.11) after successfully receiving the request. (2)  

![Screenshot 2023-06-07 104329](https://github.com/samareshbera/5giitjammu/assets/134690717/30a202ef-aeab-4ca6-a382-1bee4191b569)  

Open5GS (10.0.2.13) sends NGSetupResponse to gNB(10.0.2.11). (5)  

![Screenshot 2023-06-07 104717](https://github.com/samareshbera/5giitjammu/assets/134690717/c403cb6e-8006-4bd2-b0cf-24b874d4cc25)  

After receiving response, gNB(10.0.2.11) sends an acknowledgement to Open5GS (10.0.2.13). (5)  

![Screenshot 2023-06-07 104800](https://github.com/samareshbera/5giitjammu/assets/134690717/c6b96f96-c78a-4dae-88ef-d9ef5994fd45)  

After the UE is started, it (10.0.2.15) sends a PDU session request to gNB (10.0.2.11). (1)  

![1](https://github.com/samareshbera/5giitjammu/assets/96954630/48e5ba43-14ef-47fe-be74-4d83908f8f24)

The gNB (10.0.2.11) accepts the session and responds back to the UE (10.0.2.15). (7)  

![2](https://github.com/samareshbera/5giitjammu/assets/96954630/41e0c774-68a8-4eea-8351-440fa055695b)


GTP-U tunnel (10.45.0.2) is established between the gNB and the UPF. (6)  

![7](https://github.com/samareshbera/5giitjammu/assets/96954630/33cfc35a-3735-4218-8e5c-51cc1b25700b)


To check for Internet access, try pinging `google.com` from the UE.

`ping -I uesimtun0 google.com`   

The UE (10.0.2.15) tries to ping google.com (142.250.193.42) from the 5G network.  

![p1](https://github.com/samareshbera/5giitjammu/assets/96954630/b377dc22-48e0-4a9f-945f-857d3332248b)

As Internet access is available, google.com (142.250.193.42) responds back to the UE (10.0.2.15) with acknowledgement.  

![p2](https://github.com/samareshbera/5giitjammu/assets/96954630/050a4c0c-d0ee-4ad3-8b2f-f4153d57d736)



