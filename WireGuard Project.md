# WireGuard Project

## Step 1
I first signed up for a digital ocean account with the link provided by my teacher. I then created the cheapest possible Droplet and set up root with a password.

## Step 2

I typed this into terminal:
```
mkdir -p ~/wireguard/
mkdir -p ~/wireguard/config/
nano ~/wireguard/docker-compose.yml
```

I put this block of code into the docker-compose:
```
version: '3.8'
services:
  wireguard:
    container_name: wireguard
    image: linuxserver/wireguard
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/Chicago
      - SERVERURL=143.198.146.171
      - SERVERPORT=51820
      - PEERS=pc1,pc2,phone1
      - PEERDNS=auto
      - INTERNAL_SUBNET=10.0.0.0
    ports:
      - 51820:51820/udp
    volumes:
      - type: bind
        source: ./config/


```

The only parts you would need to change is the TZ (timezone) and the SERVERURL which is the url from the Droplet. 

## Step 3

Finally, I ran the docker-compose:
```
cd ~/wireguard/
docker-compose up -d
```

I then downloaded Wireguard on my phone and laptop to test the VPN. I was able to get the mobile VPN working but the laptop wouldn't load a website. Then phone was also running very slow so I don't know if there was a error along the way but I followed the provided instruction guide on the slides.
