# myDU-Linux-setup
How to set up myDU server in a Linux Environment, assuming Ubuntu 24.04


   
Install Docker and add your user to the docker group
```
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```
followed by
```
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
Now, if you are not logging in as root, add your user to the dockergroup
```
sudo usermod -aG docker $USER
```
log out and back in to the shell or reboot if that is easier for you.


   
Create the folder you want myDU settings to be installed into and enter it, for example
```
mkdir mydu
cd mydu
```

At least for initial install and testing I'd suggest disabling the OS firewall, if only to eliminate a possible point of failure. If you want  to keep it enabled or enable it later, when using a LAN IP or on unsecured external (WAN) IP make sure ports 8081, 9210, 9630, 10000, 10111 and 12000 are allowed through for TCP.

Download the compose and script package:
```
docker run --rm -it -v ./:/output novaquark/dual-server-fastinstall:latest
```
   
I'd advise to change the scripts to use "docker compose" instead of "docker-compose" as current docker version no longer uses/needs docker-compose and it may break python3:
```
sed -i 's/docker-compose/docker compose/g' ./scripts/*
```

copy the clean config to a separate file and set up IP. MY_IP would be the LAN IP if you only access the server internally on LAN, else it should be the EXTERNAL (WAN) IP you need the IP TWICE (that is not a typo) or the domain and the IP!!
```
cp ./config/dual.yaml ./config/dual.yaml.ori
python3 scripts/config-set-domain.py config/dual.yaml http://[DOMAIN OR MY_IP] MY_IP
```


   
----------------------------------------------------

-- OPTIONAL - SET UP SSL --

Make sure that the domain and IP are set up for SSL
```
python3 scripts/config-set-domain.py config/dual.yaml https://DOMAIN EXTERNAL_IP
```

Set up your domain, prefix and if needed alternate subdomain names in config/domains.json:
Example domains.json with alternate subdomain for queueing and backoffice:
```
{
  "tld": "MY.DOMAIN",
  "prefix": "du-",
  "services": {
    "queueing":"login",
    "backoffice":"bo"
  }
}
```

Set A or CNAME records in DNS for your subdomains
open ports 80, 443 and 9210 for TCP to the server internal IP

Create certs: 
```
./scripts/ssl.sh --create-certs
```

close port 80 after certs insalled completely

edit docker-compose.yml and comment out the ports under nginx, uncomment the 443 port

Reconfigure stack
```
./scripts/ssl.sh --config-dual MY_IP
./scripts/ssl.sh --config-nginx
```


------------------------------------

Start the stack:
```
./scripts/up.sh
```

Log into backoffice as admin to change the password (default is admin/admin)
```
https://[MYIP OR DOMAIN]:12000
```
