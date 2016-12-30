![image](https://cloud.githubusercontent.com/assets/22159102/21554484/9d542f5a-cdc4-11e6-8c4c-7730a9e9e2d1.png)

# prerendercloud-nginx
nginx middleware for prerendering javascript-rendered pages with https://www.prerender.cloud for isomorphic/universal server side rendering

## Usage
* The default.conf file is intended as a drop in config for a single standard, single nginx static host serving a single page application (redirects 404s to index.html)
  * You can also copy and paste parts or all of the config into your own nginx configuration

### Docker usage

If you're using the standard nginx docker image from https://hub.docker.com/_/nginx/

...and assuming your index.html and css and js are in `./build`

...and assuming you've copied `./default.conf` into your current directory

```
docker pull nginx
docker run -p8080:80 -v $(pwd)/build:/usr/share/nginx/html:ro -v $(pwd)/default.conf:/etc/nginx/conf.d/default.conf:ro nginx
```

## Avoid rate limiting by setting your prerendercloud secret/token
1. Uncomment the `set $prerendercloud_token` line
2. replace `YOUR_SECRET_API_TOKEN_HERE` with your actual token you got after signing up at https://www.prerender.cloud/
3. Uncomment the `proxy_set_header X-Prerender-Token` line

![image](https://cloud.githubusercontent.com/assets/16573/21571692/842d9562-ce86-11e6-94da-422b4229dad4.png)

## How errors from the server (service.prerender.cloud) are handled

* when prerender.cloud service returns
  * **400 client error (bad request)**
    * e.g. try to prerender a localhost URL as opposed to a publicly accessible URL
    * the client itself returns the 400 error (the web page will not be accessible)
  * **429 client error (rate limited)**
    * the original server payload (not prerendered) is returned, so **the request is not interrupted due to unpaid bills or free accounts**
    * only happens while on the free tier (paid subscriptions are not rate limited)
    * no messages will be written to console, comment out `proxy_intercept_errors` to see the errors over HTTP (but don't do this in production)
  * **5xx (server error)**
    * the original server payload (not prerendered) is returned, so **the request is not interrupted due to server error**
    * no messages will be written to console, comment out `proxy_intercept_errors` to see the errors served over HTTP (but don't do this in production)

## How caching works

This nginx config will cache responses from service.prerender.cloud for 5 minutes, any request after that will download the latest copy from service.prerender.cloud. If a request happens while that previous request reached out to service.prerender.cloud, that (2nd and anything beyond) will use a stale cache copy. Unfortunately nginx does not support returning stale copies for that _first_ request that triggers a download of the latest copy. See http://serverfault.com/questions/576402/nginx-serving-stale-cache-response-while-updating for more details
