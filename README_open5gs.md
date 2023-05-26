# 5giitjammu
5G network - using UERANSIM and Open5GS


2. Machine Setup
    a) CPF (Control Plane Functions)
    b) UPF (User Plane Functions)

    MongoDB installation  

    `sudo apt update`  
    `sudo apt install gnupg`  
    `curl -fsSL https://pgp.mongodb.com/server-6.0.asc | sudo gpg -o /usr/share/keyrings/mongodb-server-6.0.gpg --dearmor`  

    Package installation  

    `sudo apt update`  
    `sudo apt install -y mongodb`  
    `sudo systemctl start mongodb`  
    `sudo systemctl enable mongodb`  

    Setting TUN device  
    (creating the tun device with the interface name ogstun)  

    `sudo ip tuntap add name ogstun mode tun`  
    `sudo ip addr add 10.45.0.1/16 dev ogstun`  
    `sudo ip addr add 2001:db8:cafe::1/48 dev ogstun`  
    `sudo ip link set ogstun up`  

    Open5gs installation  
    (Installation of dependencies for building the source code.)  

    `sudo apt install python3-pip python3-setuptools python3-wheel ninja-build build-essential flex bison git cmake libsctp-dev libgnutls28-dev libgcrypt-dev libssl-dev libidn11-dev libmongoc-dev libbson-dev libyaml-dev libnghttp2-dev libmicrohttpd-dev libcurl4-gnutls-dev libnghttp2-dev libtins-dev libtalloc-dev meson`  

    Git clone  
    
    `git clone https://github.com/open5gs/open5gs`  

    Compile with meson  

    `cd open5gs`  
    `meson build --prefix=`pwd`/install`  
    `ninja -C build`  

    Checking compilation  
    (For 5g core) 

    `./build/tests/registration/registration`  

    Running test all programs 

    `cd build`      
    `meson test -v`  

    Now, need to perform Installation Process 
     




  
    
    