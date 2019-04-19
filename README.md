# kong-ratelimiting-demo

This demo shows the basic concepts of Kong by creating Services, Routes, Consumers and installing Plugins. In this demo, you will be able to set up the Rate Limiting plugin for different consumers.

## Pre-requisites

- Install [Kong Enterprise](https://docs.konghq.com/enterprise/0.34-x/installation/docker/)
- In this demo, Kong Enterprise is installed locally with Docker and ports are configured as :


| Listeners  | HTTP Ports | HTTPS Ports |
| ------------- | ------------- |-------------|
| Proxy  | 8000  | 8443 |
| Admin API  | 8001  | 8444 |
| Kong Manager  | 8002  | 8445 |
| Dev Portal  | 8003  | 8446 |


> If you have different setups, please modify the commands in this demo to match your configuration.

## Start Kong
Before any manipulation, your Kong should be started. To do that, run the following lines:
```bash
$ docker start kong-database
$ docker start kong
```
> If your installation is not under docker, please adapt the above commands to match your own configuration.

## Create Services
You will first create a service **swapi-service** in the **default** workplace. If you want to create the service within another workplace please adapt the url to your own use. You can create a service with command line with *curl* or *httpie*:

```bash
# Using curl
$ curl -i -X POST \
  --url http://localhost:8001/default/services/ \
  --data 'name=swapi-service' \
  --data 'url=https://swapi.co/api/'

# Using httpie
$ http POST http://localhost:8001/default/services \
  name=swapi-service url=https://swapi.co/api

```

Or create the service with Kong Manager interface by navigating to the service creation page <http://localhost:8002/default/services/create> with a web browser and then fill the form as:

![Create swapi-service](https://github.com/nexDigitalDev/kong-ratelimiting-demo/blob/master/img/createservice.PNG?raw=true)

If you created the service through the browser, you will be redirected to the service page where you can find all services of the default workspace. In this page you can find the ID for each service and other related inforlation. Actually, every Kong object (service, route, plugin, customer, ...) can be identified with their unique ID. For further use, please remember that you can find the ID for each service at <https://localhost:8001/default/services>.

![Services](https://github.com/nexDigitalDev/kong-ratelimiting-demo/blob/master/img/services.PNG?raw=true)

## Create Routes
Once you created the service, let's add a route under this service. As above, you can either choose to use command line :
```bash
# Using curl
$ curl -i -X POST \
  --url http://localhost:8001/services/swapi-service/routes \
  --data 'paths[]=/sw'

# Using httpie
$ http POST http://localhost:8001/services/swapi-service/routes \
  paths:='["/sw"]'
```
Or use the Kong Manager interface to add the route:

![Create Routes](https://github.com/nexDigitalDev/kong-ratelimiting-demo/blob/master/img/createroute.PNG?raw=true)

> You must specify the protocols in the Kong Manager Interface when creating a route. In the command line above, the protocols property is not set thus by default it will be configured to ["http", "https"].

## Test your routes

Now you have configured your **swapi-service** and the route so you can test them !

Try the following command to verify if Kong is properly forwarding requests to the service that you created.

```bash
# Using curl
$ curl -i -X GET --url http://localhost:8000/sw/films/1/

# Using httpie
$ http http://localhost:8000/sw/films/1/
```
The response should looks like :

![Film Request](https://github.com/nexDigitalDev/kong-ratelimiting-demo/blob/master/img/filmsrequest.PNG?raw=true)

If you want to play with this service try other requests :

```bash
# Using curl
$ curl -i -X GET --url http://localhost:8000/sw/people/1/
$ curl -i -X GET --url http://localhost:8000/sw/planets/3/
$ curl -i -X GET --url http://localhost:8000/sw/starship/9/

# Using httpie
$ http http://localhost:8000/sw/films/1/
$ http http://localhost:8000/sw/planets/3/
$ http http://localhost:8000/sw/startship/9/
```

## Create Consumers



## Install Plugins