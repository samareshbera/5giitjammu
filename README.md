# 5giitjammu
5G network - using UERANSIM and Open5GS

# 1. Machine Setup

    a) UE (User Equipment)  
    b) gNB (gNodeB)  

    Firstly, ensure that you have all the updated packages installed.

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
    

    ## Building
    Use the following command for building.

    `cd ~/UERANSIM`  
    `make` 

    After the compilation is successful, the output binaries are saved to `~/UERANSIM/build` folder.

    Run `nr-gnb` and `nr-ue` to start using UE and gNB.

