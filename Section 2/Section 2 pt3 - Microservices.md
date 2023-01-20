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

## The recommended way of connecting two microservices

