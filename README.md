# prerendercloud-nginx

<img align="right" src="https://cloud.githubusercontent.com/assets/22159102/21554484/9d542f5a-cdc4-11e6-8c4c-7730a9e9e2d1.png">

nginx middleware for pre-rendering JavaScript single page apps with [Headless-Render-API.com](https://headless-render-api.com) (formerly named prerender.cloud from 2016 - 2022)

## Usage

- The default.conf file is intended as a drop in config for a single standard, single nginx static host serving a single page application (redirects 404s to index.html)
  - You can also copy and paste parts or all of the config into your own nginx configuration

### Docker usage

If you're using the standard nginx docker image from https://hub.docker.com/_/nginx/

...and assuming your index.html and css and js are in `./build`

...and assuming you've copied `./default.conf` into your current directory

```
docker pull nginx
docker run -p8080:80 -v $(pwd)/build:/usr/share/nginx/html:ro -v $(pwd)/default.conf:/etc/nginx/conf.d/default.conf:ro nginx
```

## Avoid rate limiting by setting your prerendercloud secret/token

1. Uncomment the `proxy_set_header X-Prerender-Token` line
2. replace `YOUR_SECRET_API_TOKEN_HERE` with your actual token you got after signing up at [Headless-Render-API.com](https://headless-render-api.com)

![image](https://user-images.githubusercontent.com/22159102/30871247-04adf0ea-a29b-11e7-843f-c8d3d6639d40.png)

## Troubleshooting

![image](https://user-images.githubusercontent.com/16573/34540302-df9b30b0-f088-11e7-91ee-3c374dd9be7a.png)

## How errors from the server (service.headless-render-api.com) are handled

- when service.headless-render-api.com service returns
  - **400 client error (bad request)**
    - e.g. try to prerender a localhost URL as opposed to a publicly accessible URL
    - the client itself returns the 400 error (the web page will not be accessible)
  - **429 client error (rate limited)**
    - the original server payload (not prerendered) is returned, so **the request is not interrupted due to unpaid bills or free accounts**
    - only happens while on the free tier (paid subscriptions are not rate limited)
    - no messages will be written to console, comment out `proxy_intercept_errors` to see the errors over HTTP (but don't do this in production)
  - **5xx (server error)**
    - the original server payload (not prerendered) is returned, so **the request is not interrupted due to server error**
    - no messages will be written to console, comment out `proxy_intercept_errors` to see the errors served over HTTP (but don't do this in production)

## How caching works

This nginx config will cache responses from service.headless-render-api.com for 5 minutes, any request after that will download the latest copy from service.headless-render-api.com. If a request happens while that previous request reached out to service.headless-render-api.com, that (2nd and anything beyond) will use a stale cache copy. Unfortunately nginx does not support returning stale copies for that _first_ request that triggers a download of the latest copy. See http://serverfault.com/questions/576402/nginx-serving-stale-cache-response-while-updating for more details
