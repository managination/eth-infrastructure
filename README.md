# eth-infrastructure
configuration of geth, parity, ipfs and nginx to work in a productin environment

## Etherium-go (geth)

To launch geth as a service with autorestart

In `/etc/systemd/system` create a file named **gethRopsten.service** with: 
```sh
[Unit]
Description=Geth on ropsten

[Service]
Type=simple
ExecStart=/usr/bin/geth --testnet --rpcport=8548 --datadir "/home/yourUsername/ethereum" --fast --port=30306 --rpc --rpccorsdomain "*" --rpcaddr "0.0.0.0" > /dev/null 2>/home/yourUsername/gethLog
Restart=always


[Install]
WantedBy=multi-user.target
```
Enable the service 
```sh
sudo systemctl enable gethropsten
```

Start the service
```sh
sudo systemctl start gethropsten
```

## Parity

To launch parity 

```sh
nohup parity --chain=ropsten --rpcport=8551 --datadir "/home/username/parity" --port=30309 --dapps-port=8082 --geth > /dev/null 2>/home/username/parityLog &
```

## Nginx

### Reverse-proxy 
Edit the virtual host `etc/nginx/sites-enabled/default` and in the end of the server block add:

```sh
#geth on test-net
location /BDcHsW5a6RvCHQUJyGFgEAYVGBFP8Hy9v55MP72g/ {
	proxy_pass https://localhost:8548;
	proxy_set_header X-Forwarded-Host $server_name;
	proxy_set_header X-Real-IP $remote_addr;
}
```

### Enabling self-signed https 

create a certificate
```sh
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/nginx-selfsigned.key -out /etc/ssl/certs/nginx-selfsigned.crt
```

Edit the virtual host `etc/nginx/sites-enabled/default` and add the line 
```sh
# SSL part
	listen 443 ssl default_server;
	listen [::]:443 ssl ipv6only=on;

# SSL configuration
	ssl_certificate /etc/ssl/certs/nginx-selfsigned.crt;
	ssl_certificate_key /etc/ssl/private/nginx-selfsigned.key;
```
now your `etc/nginx/sites-enabled/default` should look like 
```sh
server {

#http part
#       listen 	80 default_server;
#       listen 	[::]:80 default_server;

# SSL part
        listen 	443 ssl default_server;
        listen 	[::]:443 ssl ipv6only=on;

# SSL configuration
        ssl_certificate /etc/ssl/certs/nginx-selfsigned.crt;
        ssl_certificate_key /etc/ssl/private/nginx-selfsigned.key;


#geth on test-net proxy
        location /maDarg9nTJK9VgbGC56kRr440Wu6nglaR7NBYUDk/ {
                proxy_pass http://localhost:8545;
                proxy_set_header X-Forwarded-Host $server_name;
                proxy_set_header X-Real-IP $remote_addr;
                }
#geth on ropsten proxy  
        location /BDcHsW5a6RvCHQUJyGFgEAYVGBFP8Hy9v55MP72g/ {
                proxy_pass http://localhost:8548;
                proxy_set_header X-Forwarded-Host $server_name;
                proxy_set_header X-Real-IP $remote_addr;
                }
#parity Proxy           
        location /u6bC4509vE6IPTNZAzo0VlyrHgaIhvMfX4sNpw3Z/ {
                proxy_pass http://localhost:8551;
                proxy_set_header X-Forwarded-Host $server_name;
                proxy_set_header X-Real-IP $remote_addr;
                }
}


```
Don't forget to open the ports on ufw
```sh
sudo ufw allow 443
```

## IPFS
the easiest way to deal with IPFS  is to create a service.

* create a user for IPF `sudo useradd -m ipfs-user`
* launch the command `sudo runuser -l  ipfs-user -c 'ipfs init'` to create the nessesary files.

in `/etc/systemd/system` create a file named **ipfs.service** and add the lines:
```
#!/bin/sh
[Unit]
Description=IPFS daemon
#After=network.target

[Service]
Type=simple
ExecStart=/usr/local/bin/ipfs daemon
Restart=on-failure
User=ipfs-user

[Install]
WantedBy=multi-user.target
```
Enable the start at boot `sudo systemctl enable ipfs`
Finally start the service ipfs `sudo systemctl start ipfs` 
