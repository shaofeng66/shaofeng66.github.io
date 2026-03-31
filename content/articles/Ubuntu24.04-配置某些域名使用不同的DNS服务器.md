+++
title = 'Ubuntu24.04 配置某些域名使用不同的DNS服务器'
date = 2026-03-31T10:53:18+08:00
draft = false
+++


client.conf
```
if_up_script=/home/shaofeng/vpn/sbwdn/update-resolved.up
if_down_script=/home/shaofeng/vpn/sbwdn/update-resolved.down
```

update-resolved.up
```
#!/bin/bash

sed -i '/# sbwip sites\.ips$/d' /etc/hosts

tmp=$(mktemp)
curl http://www.shaofeng.cc/sites.ips | sed s/$/' # sbwip sites.ips'/g > "$tmp"
cat "$tmp" >> /etc/hosts


echo '[Resolve]' > gfw.conf
echo 'DNS=8.8.8.8' >> gfw.conf
echo 'Domains=~google.com ~youtube.com ~chatgpt.com ~googleapi.com ~googlevideo.com ~github.com ~ggpht.com ~ytimg.com ~docker.io ~docker.com ~microsoft.com ~live.com' >> gfw.conf
# curl https://raw.githubusercontent.com/gfwlist/gfwlist/master/gfwlist.txt | base64 -d |grep '^||' | sed s/'^..'//g | sort | uniq > gfw.list
# echo "Domains=$(sed 's/^/~/g' gfw.list | tr '\n' ' ')" >> gfw.conf

mv gfw.conf /etc/systemd/resolved.conf.d/gfw.conf

systemctl restart systemd-resolved
```

update-resolved.down
```
#!/bin/bash

rm -f /etc/systemd/resolved.conf.d/gfw.conf

systemctl restart systemd-resolved

sed -i '/# sbwip sites\.ips$/d' /etc/hosts

```
