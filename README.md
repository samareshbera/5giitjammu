# 5giitjammu
5G network - using UERANSIM and Open5GS

a) UE (User Equipment)  
b) gNB (gNodeB)  

Firstly, ensure that you have all the updated packages installed in Ubuntu.

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


# CPF and UPF

c) CPF (Control Plane Functions)  
d) UPF (User Plane Functions)  

# MongoDB installation  

`sudo apt update`  
`sudo apt install gnupg`  
`curl -fsSL https://pgp.mongodb.com/server-6.0.asc | sudo gpg -o /usr/share/keyrings/mongodb-server-6.0.gpg --dearmor`  

# Package installation  

`sudo apt update`  
`sudo apt install -y mongodb`  
`sudo systemctl start mongodb`  
`sudo systemctl enable mongodb`  

# Setting TUN device  
## (creating the tun device with the interface name ogstun)  

`sudo ip tuntap add name ogstun mode tun`  
`sudo ip addr add 10.45.0.1/16 dev ogstun`  
`sudo ip addr add 2001:db8:cafe::1/48 dev ogstun`  
`sudo ip link set ogstun up`  

# Open5Gs installation  
## (Installation of dependencies for building the source code.)  

`sudo apt install python3-pip python3-setuptools python3-wheel ninja-build build-essential flex bison git cmake libsctp-dev libgnutls28-dev libgcrypt-dev libssl-dev libidn11-dev libmongoc-dev libbson-dev libyaml-dev libnghttp2-dev libmicrohttpd-dev libcurl4-gnutls-dev libnghttp2-dev libtins-dev libtalloc-dev meson`  

## Git clone  

`git clone https://github.com/open5gs/open5gs`  

## Compile with meson  

`cd open5gs`  
`meson build --prefix=`pwd`/install`  
`ninja -C build`  

# Checking compilation  
## (For 5G core) 

`./build/tests/registration/registration`  

## Running test all programs 

`cd build`  
`meson test -v`  

## Now, need to perform Installation Process 

`ninja install`  
`cd ../`  

# Configure Open5Gs 
(5G core)

## Changes in configuration files of Open5GS 5GC C-Plane

## Modify "/etc/open5gs/amf.yaml" to set the NGAP IP address, PLMN ID, TAC and NSSAI.

    ngap:
     - addr: 127.0.0.5
     - addr: 192.168.100.8

    guami:
       - plmn_id:
            mcc: 999
            mnc: 70
            mcc: 901
            mnc: 70

    tai:
       - plmn_id:
            mcc: 999
            mnc: 70
            mcc: 901
            mnc: 70

    plmn_support:
       - plmn_id:
            mcc: 999
            mnc: 70
            mcc: 901
            mnc: 70

    ## open5gs/install/etc/open5gs/smf.yaml

    pfcp:
        - addr: 127.0.0.4
        - addr: ::1
        - addr: 192.168.100.

    gtpu:
        - addr: 127.0.0.4
        - addr: ::1
        - addr: 192.168.0.111

    subnet:
       - addr: 10.45.0.1/16
        - addr: 2001:db8:cafe::1/48
          dnn: internet
        - addr: 10.46.0.1/16

    pfcp:
        - addr: 127.0.0.7
        - addr: 192.168.0.112
          dnn: [internet]

    ## open5gs/install/etc/open5gs/upf.yaml

    upf:
     pfcp:
        - addr: 127.0.0.7
        - addr: 192.168.100.10

    gtpu:
        - addr: 127.0.0.7
        - addr: 192.168.100.10

    subnet:
       - addr: 10.45.0.1/16
        - addr: 2001:db8:cafe::1/48
          dnn: internet
          dev: ogstun

# Restart

`sudo systemctl restart open5gs-amfd`  
`sudo systemctl restart open5gs-smfd`  
`sudo systemctl restart open5gs-upfd`  



# Building WebUI for Open5Gs
## (Node.js is required)

`sudo apt install curl`  
`curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -`  
`sudo apt install nodejs`  

## Install the dependencies to run WebUI

`cd webui`  
`npm ci`  

## WebUI runs as an npm script

`npm run dev`  

# Hostname and Port 

`HOSTNAME=192.168.100.8 npm run dev`  
`PORT=3000 npm run dev`  

Now register subscriber information
Connect to http://192.168.100.8:3000 and login with admin account.
USERNAME: admin
PASSWORD: 1423

To add subscriber information-
Go to Subscriber Menu.
Click '+' button to add a new subscriber.
Fill the IMSI.
Click SAVE Button

# Adding a route for the UE to have WAN connectivity

`sudo sysctl -w net.ipv4.ip_forward=1`  
`sudo iptables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE`  
`sudo systemctl stop ufw`  
`sudo iptables -I FORWARD 1 -j ACCEPT`


