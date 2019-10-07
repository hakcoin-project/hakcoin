# Docker container
## How to build
Inside the hakcoin root directory, execute
```
docker build -t hakcoin -f docker/Dockerfile .
```

## How to use
To run the daemon, use
```
docker run -it hakcoin
```

In order to use the hakcoin wallet cli, you can use
```
docker run -it hakcoin hakcoin-wallet-cli
```

## Docker compose example
This example runs 2 daemons for HA and a hakcoin wallet rpc with the specified keys
```
version: '3.7'

networks:
  net:
    driver: overlay
    driver_opts:
      encrypted: ''
    attachable: true
    
volumes:
  blockchain:
    name: 'blockchain-{{.Task.Slot}}'
    
services:
  hakcoin_daemon:
    image: hakcoin
    command: ["hakcoind", "--p2p-bind-ip=0.0.0.0", "--p2p-bind-port=38780", "--rpc-bind-ip=0.0.0.0", "--rpc-bind-port=38781", "--non-interactive", "--confirm-external-bind", "--restricted-rpc"]
    volumes:
      - blockchain:/root/.hakcoin
    networks:
      - net
    ports:
     - target: 38780
       published: 38780
       protocol: tcp
       mode: ingress
     - target: 38781
       published: 38781
       protocol: tcp
       mode: ingress
    deploy:
      replicas: 2
      update_config:
        parallelism: 1
        delay: 10s
  wallet:
    image: hakcoin
    command: ["hakcoin-wallet-rpc", "--wallet-file", "funding", "--password", "", "--daemon-host", "hakcoin_daemon_hakcoin_daemon", "--rpc-bind-port", "11182", "--disable-rpc-login", "--rpc-bind-ip=0.0.0.0", "--confirm-external-bind"]
    volumes:
      - ./wallet.keys:/funding.keys
      - ./wallet.cache:/funding.cache
    networks:
      - net
```