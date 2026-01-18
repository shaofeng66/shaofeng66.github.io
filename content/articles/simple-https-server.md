+++
title = 'Simple Https Server'
date = 2024-04-24T15:55:32+08:00
tags = ["https"]
draft = false

+++


```python
# taken from https://gist.github.com/DannyHinshaw/a3ac5991d66a2fe6d97a569c6cdac534
# which taken from http://www.piware.de/2011/01/creating-an-https-server-in-python/
# generate server.pem with the following command:
#    openssl req -new -x509 -keyout key.pem -out server.pem -days 365 -nodes


import http.server
import ssl
import sys

print("usage: <simple-https-server> <port> <cert pem file> <key pem file>")

port, certfile, keyfile = sys.argv[1:]

server_address = ('0.0.0.0', int(port))
httpd = http.server.HTTPServer(server_address, http.server.SimpleHTTPRequestHandler)
httpd.socket = ssl.wrap_socket(httpd.socket,
                               server_side=True,
                               certfile=certfile,
                               keyfile=keyfile,
                               ssl_version=ssl.PROTOCOL_TLS)
httpd.serve_forever()

```
