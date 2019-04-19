# kong-ratelimiting-demo

This demo shows the basic concepts of Kong by creating Services, Routes, Consumers and installing Plugins. In this demo, you will be able to set up the Rate Limiting plugin for different consumers.

## Pre-requisites

- Install [Kong Enterprise](https://docs.konghq.com/enterprise/0.34-x/installation/docker/)
- In this demo, Kong Enterprise is installed locally and ports are configured as :


Listeners | HTTP Ports | HTTPS Ports
- |:-: | -:
Proxy | 8000 | 8443
Admin APIs| 8001 | 8444
Kong Manager | 8002 | 8445
Dev Portal | 8003 | 8446

> If you have different setups, please modify the commands in this demo to match your configuration.

## Create Services
You will first create a service **swapi-service** in the **default** workplace. If you want to create the service within another workplace please adapt the url to your own use. You can either use the command line :

```bash
$ curl -i -X POST \
  --url http://localhost:8001/default/services/ \
  --data 'name=swapi-service' \
  --data 'url=https://swapi.co/api/'
```

Or navigate to Kong Manager interface http://localhost:8002/default/services/create with you web browser at  and fill the form as :

![Create swapi-service](https://github.com/nexDigitalDev/kong-ratelimiting-demo/blob/master/img/createservice.PNG?raw=true)

Once you created the service with 

## Create Routes
Onece you created the service, let's add a route under this service. As above, you can either choose to use command line :
```bash
$ curl -i -X POST \
  --url http://localhost:8001/services/swapi-service/routes \
  --data 'paths[]=/sw'
```
Or use the Kong Manager interface

## Create Consumers

## Install Plugins

