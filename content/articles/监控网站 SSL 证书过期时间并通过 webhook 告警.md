+++
title = '监控网站 SSL 证书过期时间并通过 Webhook 告警'
date = 2024-03-27T22:20:13+08:00
tags = ["ssl", "certificate", "alert", "webhook"]
draft = false
+++


cron 每天检查一次，提前30天告警：
```
10 8 * * * /root/bin/ssl_cert_about_to_expire_in_days.py example.com 30 'https://oapi.dingtalk.com/robot/send?access_token=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx'
```

/root/bin/ssl_cert_about_to_expire_in_days.py:
```python3
#!/usr/bin/python3

import sys
from datetime import datetime
import OpenSSL
import ssl
import socket
import requests

def send_webhook(url, msg):
    requests.post(url, headers={
        "Content-Type": "application/json"
    }, json={
        "msgtype": "text",
        "text": {
            "content": msg,
        },
    })

def main():
    hostname = sys.argv[1]
    port = 443
    threshold = int(sys.argv[2])
    wh_url = sys.argv[3]

    context = ssl.create_default_context()
    context.check_hostname = False
    context.verify_mode = ssl.CERT_NONE
    with socket.create_connection((hostname, port)) as sock:
        with context.wrap_socket(sock, server_hostname=hostname) as sslsock:
            der_cert = sslsock.getpeercert(True)
            x509 = OpenSSL.crypto.load_certificate(OpenSSL.crypto.FILETYPE_ASN1, der_cert)
            timestamp=x509.get_notAfter().decode('utf-8')
            notAfter = datetime.strptime(timestamp, '%Y%m%d%H%M%SZ')
            delta = notAfter - datetime.now()
            if delta.total_seconds() < 0:
                msg = f"{hostname}'s certificate has expired. notAfter: {timestamp}"
                send_webhook(wh_url, msg)
            else:
                days = delta.days
                if days < threshold:
                    msg = f"{hostname}'s certificate will expire in {days} days. notAfter: {timestamp}"
                    send_webhook(wh_url, msg)

if __name__ == '__main__':
    main()
```
