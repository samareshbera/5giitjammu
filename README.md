
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

Run Wireshark in the Open5GS machine.  

Restart the `amfd` and `upfd` services.  

It is seen that the PFCP session is established.  

*INSERT THE PFCP IMAGE HERE.....  

Now, gNB(10.0.2.11) sends a NGSetupRequest to Open5gs(10.0.2.13). (3)  

![Screenshot 2023-06-06 231954](https://github.com/samareshbera/5giitjammu/assets/134690717/d02666b6-babf-4082-b223-600084ade942)  

Open5gs(10.0.2.13) sends an acknowledgement to gNB(10.0.2.11) after successfully receiving the request. (3)  

![Screenshot 2023-06-06 232707](https://github.com/samareshbera/5giitjammu/assets/134690717/8086ea65-6c43-4c55-a3b6-d77bb74167c3)  

Open5gs(10.0.2.13) will send NGSetupResponse to gNB(10.0.2.11). (4)  

![Screenshot 2023-06-06 233442](https://github.com/samareshbera/5giitjammu/assets/134690717/d2411c60-a9af-4ed4-a3da-4f62a63db7c1)  

After receiving response, gNB(10.0.2.11) will send an Acknowledgement to Open5gs(10.0.2.13). (4)  

![Screenshot 2023-06-06 233728](https://github.com/samareshbera/5giitjammu/assets/134690717/71c1c792-0436-4045-9f0e-f3837d845b26)  


## 5G Message Flow

![Screenshot 2023-06-06 215644](https://github.com/samareshbera/5giitjammu/assets/134690717/4e543105-331e-499f-85fa-c7451fe66657)

To check for Internet access, try pinging `google.com` from the UE.

`ping -I uesimtun0 google.com`  
`PING google.com (142.250.x.x) from 10.45.0.2 uesimtun0: 56(84) bytes of data.`  

*insert ss...   


