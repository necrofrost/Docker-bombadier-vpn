# Docker-bombadier-vpn

First of all you need a Docker enabled system for this. A desktop/server would suffice

I've used Gluetun to create a VPN connection to Private Internet Access. Adjust the docker-compose file to your situation. See https://github.com/qdm12/gluetun/wiki

docker-compose.yml

```
version: "3"
services:
  gluetun:
    image: qmcgaw/gluetun
    container_name: gluetun
    cap_add:
      - NET_ADMIN
    ports:
      - 8888:8888/tcp # HTTP proxy
      - 8388:8388/tcp # Shadowsocks
      - 8388:8388/udp # Shadowsocks      
    volumes:
      - ~/data/gluetun:/gluetun #set this to a valid directory on your system
    restart: unless-stopped    
    environment:
      # See https://github.com/qdm12/gluetun/wiki
      - VPNSP=private internet access  #this is my VPN provider, choose your own
      - OPENVPN_USER=$user  #fill in user account from VPN provider
      - OPENVPN_PASSWORD=$password  #fill in password from VPN provider
      - REGION=$region #fill in region in which VPN needs to connect
      - OPENVPN_IPV6=off
      - HTTPPROXY=on
      - HTTPPROXY_PORT=8888
      - HTTPPROXY_LOG=off
      - PORT_FORWARDING=on
      - TZ=europe/amsterdam
  ```

After this it's time to start the VPN docker instance. Run the following command from the folder in which you have the docker-compose.yml

> docker-compose up -d

This will deamonize the container. To see if it's running use the following command:

> docker ps | grep gluetun

Now it's crucial to test the container if the VPN is working. First find the external IP (internet) address of the system you are working on. Either go to https://www.whatismyip.com/  or on linux use: 
> curl icanhazip.com

You can also test the IP information through docker with running this image:

> docker run --rm  alpine sh -c "apk add wget && wget -qO- https://ipinfo.io"

This should output all information regarding your internet connection. 

Now, when your gluetun container is running, use the same command as above and use the gluetun network as the transport layer

> docker run --rm --network=container:gluetun alpine sh -c "apk add wget && wget -qO- https://ipinfo.io"

You should now get a different IP address as the command without --network=container:gluetun

If this all works fine it's time to use bombadier. The command is as follows:

> docker run --network=container:gluetun -ti --rm alpine/bombardier -c $connections -d $time -l $address

Adjust the following:
  
connections --> set the amount of connections to be used. Please don't use too many as it will drain the host of resources. start with 10000
  
time --> amount of time to run the connections
  
address --> the url that you want to connect to
  
