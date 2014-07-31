Title:  How I have configured nginx
Date: 2012-02-01 10:20
Category: blog
Tags: nginx

Below is my nginx.conf configuration file, that I drop in to /etc/nginx/conf.d/.


```sh
server {
  listen 80;
  server_name dansysadm.com;
  access_log  /var/log/nginx/www.dansysadm.com.access.log;
  error_log  /var/log/nginx/www.dansysadm.com.error.log;
  error_page 404 /;
  error_page 403 /;
  location / {
    gzip on;
    root /var/www/dansysadm;
    expires 1h;
  }
  location /static {
    gzip on;
    root /var/www/dansysadm;
    expires 7d;
  }
}
```

Configuration breakdown
-----------------------

```sh
listen 80;
```

This is so the users do not need to set a custom port to access the website, as 80 is the default port for HTML.

```sh
server_name dansysadm.com;
```

The server is configured to response to the server name of dansysadm.com. 
server_name users the  Host header of the incoming HTTP request
It means that if I want to have more than one website on the same server running on the same port I can do so.

```sh
access_log  /var/log/nginx/www.dansysadm.com.access.log;
error_log  /var/log/nginx/www.dansysadm.com.error.log;
```

As I run more then 1 web server on the system, I wanted to divided up the logs per site.
This makes looking at the logs on a per-site basis very easy.

```sh
error_page 404 /;
error_page 403 /;
```

Redirect common errors back to the index page.
This stops the default ( and ugly ) nginx error page.

```sh
location / {
```

Default location of the website, it will allow the browser to request /index.html and the web server will return /var/www/dansysadm/index.html

```sh
gzip on;
```

Enable compression, this is a trade of CPU resources for bandwidth.
The files are compress so they transfer over the wire faster, however something needs to be responsible for compression and decompression...

```sh
root /var/www/dansysadm;
```
Where all the html files are for dansysadm.com.


```python
page_path = root + location + page_request 
page_path = "/var/www/dansysadm" + "/" + "index.html"
```
In the example above: The server will /var/www/dansysadm/index.html from the request of / or /index.html

```sh
expires 1h;
```
This adds expiry headers to the request so the page will be cached for 1 hour after the request.

```sh
expires 7d;
```
This also adds expiry headers to the request, however because its static content ( css, javascript, images ) It will not be changing very often.
So it has been set to 7days.


Conclusion
-----------------------

No only is nginx faster then apache, it is easy to configure.
