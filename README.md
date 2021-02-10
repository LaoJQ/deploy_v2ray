# deploy v2ray

Use ansible to deploy v2ray in a new VPS.


## raw step

+ create a vps

+ apt install unzip
+ cd ~/ && wget https://github.com/v2ray/v2ray-core/releases/download/v4.16.0/v2ray-linux-64.zip
+ unzip v2ray-linux-64.zip -d v2ray/

+ mkdir /etc/v2ray/ /usr/bin/v2ray/ /var/log/v2ray/
+ cd ~/v2ray/
+ cp v2ray v2ctl geoip.dat geosite.dat -t /usr/bin/v2ray/
+ chmod a+x /usr/bin/v2ray/v2ray /usr/bin/v2ray/v2ctl
+ touch /etc/v2ray/config.json   ## then write config.json file

+ cp ~/v2ray/systemd/v2ray.service /etc/systemd/system/
+ systemctl enable v2ray.service
+ systemctl start v2ray.service
+ systemctl restart systemd-journald


## cmd

Use ansible to setup, you just need to modify VPS host ip and uuid which can be generated by `uuidgen` command. eg:

```shell
export ANSIBLE_HOST_KEY_CHECKING=False && ansible-playbook -i 127.0.0.1, setup.yml -u root -k -e "uuid=1234567" -e "vpsport=12345"
```

Or you can also add `host_key_checking = False` to `~/.andible.cfg` file.


## server v2ray config.json

make a uuid by linux command: `uuidgen`

```json
{
    "log": {
        "loglevel": "warning"
    },
    "inbounds": [
        {
            "port": 12345,
            "protocol": "vmess",
            "settings": {
                "clients": [
                    {
                        "id": "1234567"
                    }
                ]
            }
        }
    ],
    "outbounds": [
        {
            "protocol": "freedom",
            "settings": {}
        }
    ]
}
```


## client v2ray config.json

```json
{
    "log": {
        "loglevel": "warning"
    },
    "inbounds": [{
        "port": 10086,
        "listen": "127.0.0.1",
        "protocol": "socks",
        "sniffing": {
            "enabled": true,
            "destOverride": [
                "http",
                "tls"
            ]
        },
        "settings": {
            "udp": true
        }
    }],
    "outbounds": [{
        "protocol": "vmess",
        "settings": {
            "vnext": [{
                "address": "127.0.0.1",
                "port": 12345,
                "users": [{
                    "id": "1234567"
                }]
            }]
        }
    }]
}
```