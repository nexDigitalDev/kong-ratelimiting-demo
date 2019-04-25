# kong-ratelimiting-demo

This demo shows the basic concepts of Kong by creating Services, Routes, Consumers and installing Plugins. In this demo, you will be able to set up the Rate Limiting plugin for different consumers.

## Pre-requisites

- Install [Kong Enterprise](https://docs.konghq.com/enterprise/0.34-x/installation/docker/)
> You can use [Kong Community](https://docs.konghq.com/install/docker/?_ga=2.72917264.2041191503.1556005121-1619453956.1550568933) if you don't have the Enterprise version. In this case, you will not have access to the Kong Manager interface to make operations. Yet, you can always use the command lines to configure and operate with your Kong system.
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
```docker
$ docker start kong-database
$ docker start kong
```
> If your installation is not under docker, please adapt the above commands to match your own configuration.

## Create Services
You will first create a service **swapi-service** in the **default** workspace. If you want to create the service within another workspace please adapt the url to your own use. You can create a service with command line with *curl* or *httpie*:

```bash
# Using curl
$ curl -i -X POST \
  --url http://localhost:8001/services/ \
  --data 'name=swapi-service' \
  --data 'url=https://swapi.co/api/'

# Using httpie
$ http POST http://localhost:8001/services \
  name=swapi-service url=https://swapi.co/api

```
> Generally, you must mention the workspace in the url like **http://localhost:8001/default/services/**. Without any definition of workspace, Kong will consider the default workspace. If you do not work in the default workspace, please modify the URL to match your configuration.

Or create the service with Kong Manager interface by navigating to the service creation page <http://localhost:8002/default/services/create> with a web browser and then fill the form as :

![Create swapi-service](https://github.com/nexDigitalDev/kong-ratelimiting-demo/blob/master/img/createservice.PNG?raw=true)

If you created the service through the browser, you will be redirected to the service page where you can find all services of the default workspace. In this page you can find the **ID** for each service and other related information. Actually, every Kong object (service, route, plugin, customer, ...) can be identified with their unique ID. For further use, please remember that you can find the ID for each service at <https://localhost:8002/default/services>.

![Services](https://github.com/nexDigitalDev/kong-ratelimiting-demo/blob/master/img/services.PNG?raw=true)

## Create Routes
Once you created the service, let's add a Route under this Service. By defining routes, you define rules to match client requests. Based on these rules, every request will be properly proxied to its corresponding Service.

As above, you can either choose to use command line :
```bash
# Using curl
$ curl -i -X POST \
  --url http://localhost:8001/services/swapi-service/routes \
  --data 'paths[]=/sw'

# Using httpie
$ http POST http://localhost:8001/services/swapi-service/routes \
  paths:='["/sw"]'
```
Or use the Kong Manager interface to add the route at [http://localhost:8002/default/route/create](http://localhost:8002/default/route/create) :

![Create Routes](https://github.com/nexDigitalDev/kong-ratelimiting-demo/blob/master/img/createroute.PNG?raw=true)

> You must specify the protocols in the Kong Manager Interface when creating a route. In the command line above, the protocols property is not set thus by default it will be configured to ["http", "https"].

## Test your routes

Now you have configured your **swapi-service** and the route so you can test them !

Try the following commands to verify if Kong is properly forwarding requests to the service that you created.

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

The consumer or a user of a service is represented as a **Consumer** object in Kong. It is used for tracking, access control, ...

We will first create a consumer named **Consumer1**. As above you can use command line :
               
```bash
# Using curl
$ curl -i -X POST \
  --url http://localhost:8001/consumers/ \
  --data "username=Consumer1"

# Using httpie
$ http POST http://localhost:8001/consumers username=Consumer1
```

You can either use Kong Manager Interface. In this case, navigate to [http://localhost:8002/](http://localhost:8002/) and in the **default** workspace navigate to the **Consumers** page. Then click on "**New Consumer**" and fill the form as bellow :

![Create Consumer](https://github.com/nexDigitalDev/kong-ratelimiting-demo/blob/master/img/createconsumer.PNG?raw=true)

> You can also navigate directly to [http://localhost:8002/default/consumers/create](http://localhost:8002/default/consumers/create).

## How to enable plugins: the example of Key Authentication plugin

In Kong, the notion of Authentication is highly related to Consumers. There are multiple plugins for Authentication. In this tutorial, the simple **[Key Authentication](https://docs.konghq.com/hub/kong-inc/key-auth/)** is used. With this example, you will have a general idea of how to enable plugins in Kong.
> If you want to use other Authentication plugins or simply have a look on the available Kong plugins, you can refer to the [Kong Hub](https://docs.konghq.com/hub/) page.

Generally, each plugin has its own documentation page. Within this page, there is information about how to enable that plugin, how to configure and use it. A plugin can be configured as global plugin which means it is not associated to any Service, Route or Consumer. Global plugins will be run on every request. Otherwise, you can enbale a plugin on a specific Service or Route and sometimes on a Consumer.

Basically, if you want to install a plugin the best practice is to follow the documentation page of that plugin. Now, enable the **[Key Authentication](https://docs.konghq.com/hub/kong-inc/key-auth/)** on **swapi-service**. According to the documentation page, you can use the command line by executing the following commands:
```bash
# Using curl
$ curl -i -X POST \
  --url http://localhost:8001/services/swapi-service/plugins \
  --data "name=key-auth" \
  --data "config.key_names=X-API-KEY"

# Using httpie
$ http POST http://localhost:8001/services/swapi-service/plugins \
 name=key-auth config.key_names=X-API-KEY
```

Or use the Kong Manager interface by navigating to the plugins page in the default workspace [http://localhost:8002/default/plugins](http://localhost:8002/default/plugins). Then, click on **New Plugin** and find the **Key Authentication** plugin. Fill the form as bellow: 

![Enable Key Authentication](https://github.com/nexDigitalDev/kong-ratelimiting-demo/blob/master/img/enablekeyauth.PNG?raw=true)
> Don't forget to replace the **service.id** by the ID of your **swapi-service**. You can find the ID at <http://localhost:8002/default/services>.

Now try to consume the **swapi-service**. You will notice that Authentication is required and failed.

```bash
# Using curl
$ curl -i -X GET --url http://localhost:8000/sw/starship/3/

# Using httpie
$ http http://localhost:8000/sw/starship/3/
```
![Authentication Required](https://github.com/nexDigitalDev/kong-ratelimiting-demo/blob/master/img/authrequired.PNG?raw=true)

You can also visualize all your previous requests and this unauthorized request in the Kong Manager Dashboard at <http://localhost:8002/default/dashboard>. This dashboard is related to the whole default workspace. If you want to be more precise, you can navigate to the desired Service, Route or Consumer details page to get its specific activity.

![Dashboard](https://github.com/nexDigitalDev/kong-ratelimiting-demo/blob/master/img/traffic.PNG?raw=true)


You need to pass credentials if you want to consume this service. Credentials are associated to Consumers in Kong. You can provide new credentials for **Consumer1** created before by executing the following HTTP request:
```bash
#Using curl
$ curl -i -X POST \
    --url http://localhost:8001/consumers/Consumer1/key-auth/ \
    --data 'key=nexDigital'

#Using httpie
$ http POST http://localhost:8001/consumers/Consumer1/key-auth/ key=nexDigital
```
You can either create Key Authentication credential by navigating to <http://localhost:8002/default/consumers> and click on **Consumer1**. Then, select **Credentials** and click on **New Key Auth Credential**. Fill the form as:

![Create Key Auth Credential](https://github.com/nexDigitalDev/kong-ratelimiting-demo/blob/master/img/createkey.PNG?raw=true)

> If you want to generate automatically the key, do not pass the '**key**' data.

Now **Consumer1** has credential to consume the **swapi-service**, try with the following commands to verify if credential is valid :

```bash
#Using curl
$ curl -i -X GET --url http://localhost:8000/sw/planets/11/ -H "X-API-KEY:nexDigital"

#Using httpie
$ http http://localhost:8000/sw/planets/11/  X-API-KEY:nexDigital
```
> If you have another key, don't forget to replace it in the commands above.

Get back to the Kong Manager Dashboard, you can observe that your previous command is valid with status code 200. You can also find this request in the **Consumer1** details page.

![Dashboard](https://github.com/nexDigitalDev/kong-ratelimiting-demo/blob/master/img/traffic2.PNG?raw=true)

If you navigate to the **Consumer1** details page on Kong Manager, you can find this successful request in its activity onglet.


Now you know how to enable plugins in Kong, try to enable other plugins !

## Rate Limiting
In this part, you will learn about the precedence in Kong. Kong offers the possibility to configure plugins globally, by entities or combination of entities. Consequently, there is an order of precedence for running plugin. The more specific a plugin is, the higher its priority. You can refer to this [page](https://docs.konghq.com/1.1.x/admin-api/#precedence) to get more details about precedence.
This part is based on the [Rate Limiting](https://docs.konghq.com/hub/kong-inc/rate-limiting/) plugin.
You will configure this plugin at two levels:
- Global level: with 60 calls per hour and 3 calls per minute
- Consumer level: make an exception for Consumer1 with 600 calls per hour

### Enable at Global level
First, enable the plugin as Global Plugin using the following commands or with Kong Manager interface:
```bash
#Using curl
$ curl -i -X POST \
    --url http://localhost:8001/plugins \
    --data 'name=rate-limiting' \
    --data 'config.hour=60' \
    --data 'config.minute=3'

#Using httpie
$ http http://localhost:8001/plugins name=rate-limiting config.hour=60 config.minute=3
```
Now try to consume the service several times manually or automatically with the following commands. You will notice at a moment that your requests will be blocked since you exceeded the rate limit:
```bash
#Using curl
$ seq 50 | parallel -n0 "curl -i -X GET \
  --url http://localhost:8000/sw/films/1/ \
  -H 'X-API-KEY:nexDigital'"

#Using httpie
$ seq 50 | parallel -n0 "http --ignore-stdin \
  http://localhost:8000/sw/films/1/  \
  X-API-KEY:nexDigital"
```
> You must install the parallel package with this command: **sudo apt install parallel**

![Rate Limit Exceeded](https://github.com/nexDigitalDev/kong-ratelimiting-demo/blob/master/img/ratelimiting.PNG?raw=true)

### Enable at Consumer level

Now, make an exception for **Consumer1** with **600 calls per hour**. You can use the following lines :

```bash
#Using curl
$ curl -i -X POST \
    --url http://localhost:8001/consumers/Consumer1/plugins \
    --data 'name=rate-limiting' \
    --data 'config.hour=600'

#Using httpie
$ http http://localhost:8001/consumers/Consumer1/plugins \
  name=rate-limiting config.hour=600
```
Now, try again the following commands, you may notice that the limit is increased for the **Consumer1** and requests are not blocked.
```bash
#Using curl
$ seq 50 | parallel -n0 "curl -i -X GET \
  --url http://localhost:8000/sw/films/1/ \
  -H 'X-API-KEY:nexDigital'"

#Using httpie
$ seq 50 | parallel -n0 "http --ignore-stdin \
  http://localhost:8000/sw/films/1/  \
  X-API-KEY:nexDigital"
```
In the Kong Manager Dashboard, the traffic should looks like:

![Dashboard](https://github.com/nexDigitalDev/kong-ratelimiting-demo/blob/master/img/traffic3.PNG?raw=true)

You can also verify the rate by doing a single request :
```bash
#Using curl
$ curl -i -X GET \
  --url http://localhost:8000/sw/films/1/ \
  -H 'X-API-KEY:nexDigital'

#Using httpie
$ http http://localhost:8000/sw/films/1/ X-API-KEY:nexDigital
```
In the response headers, you can find that the **X-Rate-Limit-hour** header is set to **600**. Clients can also check how many calls remain in the hour with the **X-RateLimit-Remaining-hour** header.

![Rate Limiting Response Headers](https://github.com/nexDigitalDev/kong-ratelimiting-demo/blob/master/img/detailsrate.PNG?raw=true)

In Kong, you can make distinction between consumers. Try to create another consummer **Consumer2** as above. Then, create a credential with key set to **'kong'** and give him a rate limit of **10 calls per minute**. When you finished, execute the follwing commands:
```bash
#Using curl
$ seq 50 | parallel -n0 "curl -i -X GET \
  --url http://localhost:8000/sw/films/1/ \
  -H 'X-API-KEY:kong'"

#Using httpie
$ seq 50 | parallel -n0 "http --ignore-stdin \
  http://localhost:8000/sw/films/1/  \
  X-API-KEY:kong"
```
The **Consumer2** has a rate limit more restricted than **Consumer1**. If you observe the dashboard, you may find something like:

![Dashboard](https://github.com/nexDigitalDev/kong-ratelimiting-demo/blob/master/img/traffic4.PNG?raw=true)

## Links
There is the end of this tutorial, for more information please refer to:
- [Kong Enterprise Documentation](https://docs.konghq.com/enterprise/)
- [Kong Hub](https://docs.konghq.com/hub/)
- [Kong Nation](https://discuss.konghq.com/)