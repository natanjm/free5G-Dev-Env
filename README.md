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
Then reboot to aply the changes

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

### UPF
Installing dependencies:
```
cd ~
git clone https://github.com/free5gc/gtp5g.git
cd gtp5g
git checkout v0.3.2
make
sudo make install
```
Building the UPF:
```
cd ~
cd go/src/free5gc/NFs/upf
mkdir build
cd build
cmake ..
make -j`nproc`
```
Setup environment
```
# (Must) IPv4 forwarding
sudo sysctl -a | grep forward        # check sys rule
sudo sysctl -w net.ipv4.ip_forward=1

# (Recommend) Forwarding chain in iptables can forward packet
sudo iptables -A FORWARD -j ACCEPT

# (Recommend) Close ubuntu firewall
sudo systemctl stop ufw
sudo systemctl disable ufw

# (Optional) Using NAT for UE to access data network
sudo iptables -t nat -A POSTROUTING -o <YOUR_NETWORK_INTERFACE_LABEL> -j MASQUERADE
```

### Web Console
Installing dependencies:
```
sudo apt remove cmdtest
sudo apt remove yarn
curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
sudo apt-get update
sudo apt-get install -y nodejs yarn
```
Building the Web Console:
```
cd ~
cd go/src/webconsole/frontend
yarn install
yarn build
rm -rf ../public
cp -R build ../public
```

### RAN Tester
Cloning the project:
```
cd ~
cd go/src
git clone https://github.com/my5G/my5G-RANTester.git
```
Installing dependencies:
```
cd my5G-RANTester
go mod download
```
Build the binary:
```
cd cmd 
go build app.go
```

### Debugging/Testing the 5G Core
The network functions must to be executed in the right order. Check it below:
```
NRF -> UDR -> UDM -> AUSF -> NSSF -> AMF -> PCF -> UPF -> SMF
```
After you can run the web server and the frontend.

#### NFs
After opening the project in the Goland IDE, you can access the network functions folders in NFs. Inside the network function folder there is a main file with the same name as the folder, this way you can debug it to test the network function.

![executar_nf](https://user-images.githubusercontent.com/36445263/137417909-64c9d0b5-9354-4cb5-bcd8-c39034c171b1.PNG)

You can do this for every network function, just not for UPF and N3IWF. The N3IWF function will not be used in this tutorial.

#### UPF
```
cd ~
cd go/src/free5gc/NFs/upf/build
sudo -E ./bin/free5gc-upfd
```

#### Web Server
```
cd ~
cd go/src/free5gc/webconsole
go run server.go
```

#### Frontend
```
cd ~
go/src/free5gc/webconsole/frontend
REACT_APP_HTTP_API_URL=http://127.0.0.1:5000/api PORT=3000 yarn start
```
Access url: http://localhost:3000/#/subscriber
User: admin
Password: free5gc

After you can add a new subscriber and save these values:
Submit without changing any field.

![Capturar](https://user-images.githubusercontent.com/36445263/137420977-997c9671-01d1-4aa1-a10c-6039a23178b0.PNG)

These data should be inserted in the ran tester configuration file.

#### Using the RAN Tester
Open RAN Tester project, access config/config.yml and change the msin, key and opc values with the submited subscriber values. Like this:

![config](https://user-images.githubusercontent.com/36445263/137422134-3e834120-fd31-4b07-8268-ef425500c2c2.png)



