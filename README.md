# HAProxy Testing

This docker-compose sets up HAProxy in front of an echo server. 

## Usage:

* Start it up:

  ```
  docker-compose up
  ```

* Make config changes.  
  
  Edit _haproxy/haproxy.cfg_ locally on your host.

* Reload HAProxy to pick up any config changes:

  ```
  docker kill -s HUP haproxytesting_haproxy_1
  ```
  
* Make requests

  ```
  curl -X GET localhost/foo/bar
  ```

## Examples

All requests to localhost will echo back stats on the request.  For example:

```
curl -X GET localhost/foo/bar/

{
  "path": "/foo/bar/",
  "headers": {
    "host": "localhost",
    "user-agent": "curl/7.54.0",
    "accept": "*/*"
  },
  ...
}
```

I recently wanted to test some regex'd path changes, and this worked well.

For example, if you wanted `/foo/images/` to map to `/bar/images/`, you could add edit the backend config in _haproxy.cfg_ to be

```
listen echofrontend
    bind 0.0.0.0:80
    mode http
    reqrep ^([^\ ]*)\ /foo/images/(.*)     \1\ /bar/images/\2
    server echobackend echo:80 check
```

Now, the request is updated before hitting the backend:

```
curl -X GET localhost/foo/images/
{
  "path": "/bar/images/",
  "headers": {
    "host": "localhost",
    "user-agent": "curl/7.54.0",
    "accept": "*/*"
  },
  "method": "GET",
  "hostname": "localhost",
  "protocol": "http",
  ...
}
```