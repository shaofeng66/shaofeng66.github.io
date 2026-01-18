+++
title = 'This Service Run Ssh Remote Port Forwarding'
date = 2026-01-06T14:59:06+08:00
tags = "ssh"
draft = false
+++



```shell
# This service run ssh remote port forwarding(https://help.ubuntu.com/community/SSH/OpenSSH/PortForwarding#Remote_Port_Forwarding)
# it open a tcp port on remote host which usually on cloud, forwarding the tcp connection to port 22 on host in private network.
# usage:
# first ssh-copy-id -l <username of remote host> <remote host>
# then on remote host, add a line ```GatewayPorts clientspecified``` to /etc/ssh/sshd_config, and ```systemctl restart sshd```
# copy this file to /etc/systemd/system/ssh-tunnel@.service
# then: systemctl daemon-reload ; systemctl start ssh-tunnel@<remote host>:<port to listen on remote host>:<username of remote host>:<path to id file>:<host in private network>; systemctl enable ssh-tunnel@<remote host>:<port to listen on remote host>:<username of remote host>:<path to id file>:<host in private network>
# e.g.: systemctl daemon-reload ; systemctl start ssh-tunnel@61.171.66.209:32249:root:/root/.ssh/id_rsa:127.0.0.1; systemctl enable ssh-tunnel@61.171.66.209:32249:root:/root/.ssh/id_rsa:127.0.0.1
# multipe systemd service with different remote hosts or different remote port could be setup.

[Unit]
Description=SSH Tunnel with Dynamic Port Forwarding
After=network.target

[Service]
Type=simple
Environment=REMOTE=%I
ExecStart=/bin/bash -c 'IFS=":" read -r REMOTE_HOST PORT USER KEY_FILE LOCAL_HOST <<< "$REMOTE"; (while true;do echo 'date';sleep 60;done) | ssh -l $${USER} -i $${KEY_FILE} -R 0.0.0.0:$${PORT}:$${LOCAL_HOST}:22 -o PreferredAuthentications=publickey -o IdentitiesOnly=yes -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no $${REMOTE_HOST}'
Restart=always

[Install]
WantedBy=multi-user.target

```
