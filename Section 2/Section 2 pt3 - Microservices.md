# Microservices with Docker

We will be working with two microservices:
- Currency Conversion Service
    - converts from an amount of one currency to an amount in another
    - uses the Currency Exchange Service to find the exchange rate
- Currency Exchange Service
    - returns the exchange rate between two currencies

## Linking two microservice containers

Listing local networks on docker:
```sh
docker network ls
```

-  Output:

    ```
    NETWORK ID     NAME      DRIVER    SCOPE
    528eb3a56b36   bridge    bridge    local
    deba50c1f5bf   host      host      local
    5aef7cdbdfee   none      null      local
    ```

Let's inspect the bridge network:

```sh
docker network inspect bridge
```

We find that both containers are part of this network:
```json
"Containers": {
            "91b313159ed8d783c3017f44bc26c0e1c427a4a25a5aa14ce85eb2a2eacb7b52": {
                "Name": "currency-exchange",
                "EndpointID": "18ba27ded64496299c385119d8b465ce01cd574f7af9355a0b2583ba1c8c751b",
                "MacAddress": "02:42:ac:11:00:02",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            },
            "c68a625c8738d44f5de718010a56bce1983b0a4c856de2b70faba67a9382c9f9": {
                "Name": "currency-conversion",
                "EndpointID": "5b2c1bcb32479e87c9d8ecd06e50b44d27641bbc1c1dbd7c759abdb1a0615ae6",
                "MacAddress": "02:42:ac:11:00:03",
                "IPv4Address": "172.17.0.3/16",
                "IPv6Address": ""
            }
        },
```

**However**, they cannot communicate together by default.

A quick **(but not ideal!)** way of getting the communication working is:

1. Stop the currency-conversion container
2. Add a link to currency-exchange:
  ```sh
  docker run -d -p 8100:8100 --name currency-conversion --link currency-exchange in28min/currency-conversion:0.0.1-RELEASE
  ```
3. Add the url created by the link as the new host url:
  ```sh
  docker run -d -p 8100:8100 --env CURRENCY_EXCHANGE_SERVICE_HOST=http://currency-exchange --name currency-conversion --link currency-exchange in28min/currency-conversion:0.0.1-RELEASE
  ```

> It is a good idea to name your containers -- to reference them more easily when handling links between them such as this.

## The recommended way of connecting two microservices

- networking on Host instead of Bridge is another option for connecting two containers directly, however... 
- it is not supported on Docker Desktop (Windows, Mac...), only on host machines with Linux
  
With this option unavailable, we can **create a new network** for the two microservice containers:
```sh
docker network create currency-network
```

Afterwards, we only need to specify this new network for both containers:
```
docker run -d -p 8000:8000 --name currency-exchange --network=currency-network in28min/currency-exchange:0.0.1-RELEASE
```

```
docker run -d -p 8100:8100 --env CURRENCY_EXCHANGE_SERVICE_HOST=http://currency-exchange --name currency-conversion --network=currency-network in28min/currency-conversion:0.0.1-RELEASE
```
- the currency conversion container keeps the environment variable CURRENCY_EXCHANGE_SERVICE_HOST, which determines the host to which it should send http requests

# Docker Compose

We saw in the previous steps that launching these containers has become rather difficult. Docker Compose solves this problem by letting us set up a static file that sets up the network and both containers at once.

- Docker Compose is part of Docker Desktop, but not part of the CLI-only release for Linux.

We can create a **docker-compose.yaml** file out of the two container commands and the network command above:
```yaml
version: '3.7'
services:
# docker run -d -p 8000:8000 
# --name currency-exchange --network=currency-network 
# in28min/currency-exchange:0.0.1-RELEASE
  currency-exchange:
    image: in28min/currency-exchange:0.0.1-RELEASE
    ports:
      - "8000:8000"
    restart: always
    networks:
      - currency-compose-network

# docker run -d -p 8100:8100
# --env CURRENCY_EXCHANGE_SERVICE_HOST=http://currency-exchange 
# --name currency-conversion --network=currency-network 
# in28min/currency-conversion:0.0.1-RELEASE
  currency-conversion:
    image: in28min/currency-conversion:0.0.1-RELEASE
    ports:
      - "8100:8100"
    restart: always
    environment:
      CURRENCY_EXCHANGE_SERVICE_HOST: http://currency-exchange
    depends_on:
      - currency-exchange
    networks:
      - currency-compose-network
  
# Networks to be created to facilitate communication between containers
networks:
  currency-compose-network:
```

- The services are our containers and can be microservices, but also databases and other services we may want to use
- This means that this service will only start if currency-exchange launches successfully:
  ```yaml
  depends_on:
      - currency-exchange
  ```
- The name of the network will be prepended with the name of the folder in which the **docker-compose.yaml** is located
- in this case: **microservices_currency-compose-network**

Checking docker-compose:
```sh
docker-compose --version
```
- ```
  Docker Compose version v2.15.1
  ```

Running the docker-compose:
```sh
docker-compose up (-d)
```

Stop and remove the containers and network:
```sh
docker-compose down
```

## Extra commands

See events happening in docker-compose, similarly to "docker system events":
```sh
docker-compose events
```

See the docker-compose.yaml file,  or get warned of any errors present therein:
```sh
docker-compose config
```

Other commands familiar from regular container use:

```
docker-compose ps
docker-compose top
docker-compose pause
docker-compose unpause
docker-compose stop
docker-compose kill
```
