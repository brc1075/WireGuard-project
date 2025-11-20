# Project 3 Documentation


## Accessing Digital Ocean
1. Sign up for a DigitalOcean[.]com Account using this link: https://m.do.co/c/d33d59113ab6
	- Press Get started on the $200 credit pop up
	- Sign up for a new account
2. Once you have logged on and are in the dashboard
	- Navigate to the billing tab on the left side menu
	- Add a credit card to the  payment section
		- you will not be charged until after the 60 days, so don't forget to cancel
3. Navigate to the left side menu again and click on the droplets tab
	- Click create droplet
	- Choose a plan
		- Ubuntu 24.04 LS
		- Regular Intel CPU
		- Basic $6/month droplet
	- Create a password (Write it down)
	- Pick a location near you for the region
		- I chose San Francisco
	- Click Create Droplet
4. You should now see a droplet in the droplets section that looks similar to the image below
	- write down the ip address
	
![Droplet Image](https://raw.githubusercontent.com/brc1075/WireGuard-project/main/wireGuard_photos/Lab3_photo1.png)

## Installing Docker

5. Run the command: `ssh root@”your_droplets_ip_address”`
	- Type `yes` if it says “Are you sure you want to continue connecting?”
	- Then it will ask you for the password to your droplet
	
**NOTE:** If it immediately closes right after opening and there are no typos, just run the command again. You should get the password prompt

6. Once in the droplet terminal run the command
	- `sudo apt update`
7. Then install docker using the commands from  [docker install](https://docs.docker.com/engine/install/). Scroll down to the table that shows all the platforms and choose the one that best fits your VM
	- Ubuntu is the best fit for my case
	- This will take you to instructions for installing docker
8. Follow the provided instructions:
	- Run the following command:
		`sudo apt remove $(dpkg --get-selections docker.io docker-compose docker-compose-v2 docker-doc podman-docker containerd runc | cut -f1)`
	- Install using the apt repository by entering the code from the website:
```
	#Add Docker&#39;s official GPG key:
	sudo apt-get update
	sudo apt-get install ca-certificates curl
	sudo install -m 0755 -d /etc/apt/keyrings
	sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o
/etc/apt/keyrings/docker.asc
	sudo chmod a+r /etc/apt/keyrings/docker.asc
	#Add the repository to Apt sources:
	sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
	Types: deb
	URIs: https://download.docker.com/linux/ubuntu 
	Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
	Components: stable 
	Signed-By: /etc/apt/keyrings/docker.asc 
	EOF
	sudo apt update
```

- Install the docker packages with the following command `sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin`

- Verify the installation by running the following command
`sudo docker run hello-world` OR
`docker –version` and `docker compose version`

**NOTE:** I recommend copying the code from the website instead of this file. Adding the code into the markup file lead to some discrepancies.

## Create Folder and Start Container

9. Once Docker and Docker compose have been installed you are ready to start the project 
10. Make a new directory called wireguard and move into it
	- `mkdir wireguard` 
	- `cd wireguard`
11. Create a compose.yml file
	- `nano compose.yml`
12. Copy the code that was on the slides from class into that file:
```
version: '3.8'

services:
  wireguard:
    container_name: wireguard
    image: linuxserver/wireguard
    environment:
     - PUID=1000
     - PGID=1000
     - TZ=Asia/Hong_Kong
     - SERVERURL=1.2.3.4
     - SERVERPORT=51820
     - PEERS=pc1,pc2,phone1
     - PEERDNS=auto
     - INTERNAL_SUBNET=10.0.0.0
    ports:
      - 51820:51820/udp
    volumes:
      - type: bind
        source: ./config/
        target: /config/
      - type: bind
        source: /lib/modules
        target: /lib/modules
    restart: always
    cap_add:
     - NET_ADMIN
     - SYS_MODULE
    sysctls:
      - net.ipv4.conf.all.src_valid_mark=1
```
14. You will need to edit it a little after
	- Change TZ=Asia/Hong_Kong to TZ=America/Chicago
	- Change PEERS to pc,phone
	- Change the serverurl with the ip of your droplet
	- Change the internal sebnet to something that will not conflict with other networks, i chose 10.13.13.0
![compose file](https://raw.githubusercontent.com/brc1075/WireGuard-project/main/wireGuard_photos/Lab3_photo2.png)
![compose file 2](https://raw.githubusercontent.com/brc1075/WireGuard-project/main/wireGuard_photos/Lab3_photo3.png)
15. Start the wireguard container
	- `docker compose up -d`
	- `docker ps`
![compose up example](https://raw.githubusercontent.com/brc1075/WireGuard-project/main/wireGuard_photos/Lab3_photo4.png)
16. After you have it up and runnning you are going to want to move into your config folder
	- `cd /root/wireguard/config/wg_confs`
	- `ls`
		- You should see a file named wg0.conf

## Getting your Config files
15. Open another terminal that is not in your droplet
16. Copy the  wg0.conf file to your home
	- `scp root@159.65.111.98:/root/wireguard/config/wg_confs/wg0.conf .`
	- Enter your password
	- now you can see that file from your home folder using `nano`
17. Now we need to copy the files for pc and phone
	- `scp root@159.65.111.98:/root/wireguard/config/peer_pc/*.conf .`
	- `scp root@159.65.111.98:/root/wireguard/config/peer_phone/*.conf .`
	
18. Get the configs onto phone using a QR code
	 - Install qrencode on your vm
		 - `sudo apt update`
		 - `sudo apt install qrencode`
	- Since you already copied the phone config to the home folder you can now create the QR code
		- `qrencode -t ANSIUTF8 < peer_phone.conf`
19. Open your phone
20. Download the wireguard app
21. Click add tunnel
22. Scan the QR code

## Testing Iphone VPN
23. Go to IPLeak.net to see your ip address
![ipleak before example](https://raw.githubusercontent.com/brc1075/WireGuard-project/main/wireGuard_photos/Lab3_photo5.png)
 
24. Then go to settings and turn on/activate your vpn
![proof vpn was on](https://raw.githubusercontent.com/brc1075/WireGuard-project/main/wireGuard_photos/Lab3_photo6.PNG)
25. Go back to IPLeak.nets and see how your ip address has changed
	- your ip should now be the same as your droplet
![changed phone ip](https://raw.githubusercontent.com/brc1075/WireGuard-project/main/wireGuard_photos/Lab3_photo7.PNG)
## Testing on PC
**Note:** Since I have a mac, for some reason it was not working with the GUIs so I had to google how to set it up in my vm terminal.

26. Open the VM and go to Firefox
	- Go to IPLeak.net and you can see your current ip address
![pc before ip](https://raw.githubusercontent.com/brc1075/WireGuard-project/main/wireGuard_photos/Lab3_photo8.png)
27. Open your terminal back up
28. Install wireguard
	- `sudo apt install wireguard -y`
	- enter password
29. Once installed use `cat peer_pc.conf` to view the config file
![peer_pc file](https://raw.githubusercontent.com/brc1075/WireGuard-project/main/wireGuard_photos/Lab3_photo9.png)
30. Move your peer_pc.conf file from home to wg0.conf and change its permissions
	- `sudo mv ~peer_pc.conf /etc/wireguard/wg0.conf`
	- `sudo chmod 600 /etc/wireguard/wg0.conf`
31. Turn the VPN on
	- `sudo wg-quick up wg0`
![vpn up and running](https://raw.githubusercontent.com/brc1075/WireGuard-project/main/wireGuard_photos/Lab3_photo10.png)
32. Go back to IPLeak.net and see how your ip address has changed
![changed pc ip address](https://raw.githubusercontent.com/brc1075/WireGuard-project/main/wireGuard_photos/Lab3_photo11.png)




