# free5G-Dev-Env
This is a step-by-step how to install and configure the 5G core development environment.<br/>
Tested on Ubuntu 18.04.5 and kernel 5.4.0-42-generic.<br/>
Recommended kernel versions: 5.0.0-23-generic or 5.4.x.

### Kernel version
Checking the kernel version:
```
uname -r
```
To change the kernel version, execute:
```
sudo apt-get install -y linux-image-5.0.0-23-generic
```
or
```
sudo apt-get install -y linux-image-5.4.0-42-generic
```

### Go installation
If another version of Go is installed, remove the existing version and install Go 1.14.4:
```
sudo rm -rf /usr/local/go
wget https://dl.google.com/go/go1.14.4.linux-amd64.tar.gz
sudo tar -C /usr/local -zxvf go1.14.4.linux-amd64.tar.gz
```
If Go is not installed on your system:
```
wget https://dl.google.com/go/go1.14.4.linux-amd64.tar.gz
sudo tar -C /usr/local -zxvf go1.14.4.linux-amd64.tar.gz
mkdir -p ~/go/{bin,pkg,src}
echo 'export GOPATH=$HOME/go' >> ~/.bashrc
echo 'export GOROOT=/usr/local/go' >> ~/.bashrc
echo 'export PATH=$PATH:$GOPATH/bin:$GOROOT/bin' >> ~/.bashrc
echo 'export GO111MODULE=auto' >> ~/.bashrc
source ~/.bashrc
```

### Control-plane Supporting Packages
```
sudo apt -y update
sudo apt -y install mongodb
sudo apt -y install wget
sudo apt -y install git
sudo systemctl start mongodb
```

### User-plane Supporting Packages
```
sudo apt -y update
sudo apt -y install gcc       
sudo apt -y install g++
sudo apt -y install cmake
sudo apt -y install autoconf
sudo apt -y install libtool
sudo apt -y install pkg-config
sudo apt -y install libmnl-dev
sudo apt -y install libyaml-dev
sudo apt -y install net-tools
go get -u github.com/sirupsen/logrus
```

### Linux Host Network Settings
First of all, you need to know the **internet network interface** of your system, then execute **ifconfig** to get it.

![ifconfig](https://user-images.githubusercontent.com/36445263/137234844-6aba81cd-dd19-44d5-a5ed-7d7c3418d577.PNG)

```
sudo sysctl -w net.ipv4.ip_forward=1
sudo iptables -t nat -A POSTROUTING -o <YOUR_NETWORK_INTERFACE_LABEL> -j MASQUERADE
sudo iptables -A FORWARD -p tcp -m tcp --tcp-flags SYN,RST SYN -j TCPMSS --set-mss 1400
sudo systemctl stop ufw
```

### Jetbrains Goland IDE
```
sudo snap install goland --classic
```

### Clone the free5GC repository
```
cd ~
cd go/src
git clone --recursive -b v3.0.6 -j `nproc` https://github.com/free5gc/free5gc.git
```

### UPF dependencies
```
cd ~
git clone https://github.com/free5gc/gtp5g.git
cd gtp5g
git checkout v0.3.2
make
sudo make install
```

### UPF buid
```
cd ~
cd go/src/free5gc/NFs/upf
mkdir build
cd build
cmake ..
make -j`nproc`
```
