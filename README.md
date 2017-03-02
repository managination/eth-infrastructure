# eth-infrastructure
configuration of geth, parity, ipfs and nginx to work in a productin environment

## Etherium-go (geth)

To launch geth 
```sh
nohup geth --testnet --rpcport=8548 --datadir "/home/username/ethereum" --fast --port=30306 --rpc --rpccorsdomain "*" --rpcaddr "0.0.0.0" > /dev/null 2>/home/username/gethLog &
```

## Parity

To launch parity 

```sh
nohup parity --chain=ropsten --rpcport=8551 --datadir "/home/username/parity" --port=30309 --dapps-port=8082 --geth > /dev/null 2>/home/username/parityLog &
```

## Nginx

### Reverse-proxy 
Edit the virtual host *etc/nginx/sites-enabled/default* and in the end of the server block add:

```sh
#geth on test-net
location /BDcHsW5a6RvCHQUJyGFgEAYVGBFP8Hy9v55MP72g/ {
	proxy_pass https://YourWebSite:8548;
	proxy_set_header X-Forwarded-Host $server_name;
	proxy_set_header X-Real-IP $remote_addr;
}
```
Don't forget to open the ports
On ufw:

```sh
sudo ufw allow 8548
```

### Enabling Https 



## IPFS
