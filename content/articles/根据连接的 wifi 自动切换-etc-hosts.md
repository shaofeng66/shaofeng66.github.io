+++
title = '根据连接的 wifi 自动切换 /etc/hosts'
date = 2025-12-30T10:20:16+08:00
tags = ["ubuntu", "wifi"]
draft = false
+++



```bash
#!/bin/bash
#
# this is a NetworkManager dispatcher script, use this to switch IPs of hosts in /etc/hosts
# usage:
# chmod 700 /home/shaofeng/bin/switch_hosts.dispatcher
# chown root:root /home/shaofeng/bin/switch_hosts.dispatcher
# sudo ln -s /home/shaofeng/bin/switch_hosts.dispatcher /etc/NetworkManager/dispatcher.d/switch_hosts.dispatcher
# sudo systemctl restart NetworkManager

SSID_LIST=("71-502-5G" "71-502" "66")

if [[ "$2" == "up" ]]; then
    # 获取当前连接的 SSID
    CURRENT_SSID=$(iwgetid -r)

    for SSID in "${SSID_LIST[@]}"; do
	if [[ "$CURRENT_SSID" == "$SSID" ]]; then
		sed -i -e '/dynremote/ s/^[^#]/#&/' -e '/dynlocal/ s/^#//' /etc/hosts
		exit 0
	fi
    done

    # not in list, use remote
    sed -i -e '/dynlocal/ s/^[^#]/#&/' -e '/dynremote/ s/^#//' /etc/hosts
elif [[ "$2" == "vpn-up" ]]; then
    if [[ "${CONNECTION_ID}" == "home" ]]; then
	sed -i -e '/dynremote/ s/^[^#]/#&/' -e '/dynlocal/ s/^#//' /etc/hosts
	exit 0
    fi
elif [[ "$2" == "vpn-down" ]]; then
    if [[ "${CONNECTION_ID}" == "home" ]]; then
	sed -i -e '/dynlocal/ s/^[^#]/#&/' -e '/dynremote/ s/^#//' /etc/hosts
	exit 0
    fi
fi

```

/etc/hosts:
```
# machines in kaiyun
192.168.4.249 tapedev.yinhe # dynremote
192.168.4.240 sunflower.yinhe # dynremote
192.168.4.241 daisy.yinhe # dynremote

#127.0.4.249 tapedev.yinhe # dynlocal
#127.0.4.240 sunflower.yinhe # dynlocal
#127.0.4.241 daisy.yinhe # dynlocal
```
