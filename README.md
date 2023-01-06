# Gaining API access to VirginMedia Hub 5 

## Introduction
I have written a script that retrieves the Cable Modems (CM) stats by querying the hub5's Api, the data is stored in mysql and stats displayed in Grafana. I will will be uploaded the script to github soon.

### How to access APi
A bearer token is generated each time you login.
#### Get Token
```console
curl --location --request POST 'http://192.168.0.1/rest/v1/user/login' \
--header 'Content-Type: application/json' \
--data-raw '{"password":"ABC123"}'
```
The password is the that printed on the CM which you use to access the config page. Replace ABC123 with your password but keep the quotes.

The query will return:
```JSON
{
    "created": {
        "token": "e7a3c625616b8ad58493e1a9c91181c3",
        "userLevel": "regular",
        "userId": 3
    }
}
```
When a token is generated, it will not be possible to login from another device, you should delete it after you have finished using it or it will delete itself after 10 minutes of inactivity.
#### Delete Token
```console
curl --location --request DELETE 'http://192.168.0.1/rest/v1/user/3/token/e7a3c625616b8ad58493e1a9c91181c3' \
--header 'Authorization: Bearer e7a3c625616b8ad58493e1a9c91181c3'
```
No response will be returned when running the Delete Token. An incorrect/invalid key will return:
```json
{"status":401,"message":"Unauthorized","errorCode":7}
```

## Example Queries

#### Ping Servers 
When performing a Ping (ICMP) or Traceroute, it will create a job, the sequence is to POST, GET and DELETE. A further job cannot be created until the previous have been deleted.
```console 
curl --location --request POST 'http://192.168.0.1/rest/v1/system/diagnostics/ping/jobs' \
--header 'Token: Bearer e7a3c625616b8ad58493e1a9c91181c3' \
--header 'Content-Type: application/json' \
--data-raw '{"pingJob":{"parameters":{"host":"1.1.1.1","numberOfPings":3,"dataBlockSize":64}}}'
```
#### Retrieve Ping
```console
curl --location --request GET 'http://192.168.0.1/rest/v1/system/diagnostics/ping/job/1'
```
This will return:
```JSON
{
    "pingJob": {
        "state": "complete",
        "parameters": {
            "host": "1.1.1.1",
            "interface": "erouter0",
            "intervalMs": null,
            "numberOfPings": 3,
            "dataBlockSize": 64,
            "timeoutMs": 10
        },
        "results": {
            "successCount": 3,
            "failureCount": 0,
            "averageResponseTimeMs": "16.268",
            "minimumResponseTimeMs": "16.169",
            "maximumResponseTimeMs": "16.396"
        }
    }
}
```
#### Delete Ping Job
```console
curl --location --request DELETE 'http://192.168.0.1/rest/v1/system/diagnostics/ping/job/1' \
--header 'Token: Bearer e7a3c625616b8ad58493e1a9c91181c3' \
--header 'Content-Type: application/json' \
--data-raw '{"pingJob":{"parameters":{"host":"1.1.1.1","numberOfPings":3,"dataBlockSize":64}}}'
```
#### GET Information from DHCP Server
```console
curl --location --request GET 'http://192.168.0.1/rest/v1/network/ipv4/dhcp' \
--header 'Accept-Language: en-GB,en;q=0.9,en-US;q=0.8' \
--header 'Authorization: Bearer e7a3c625616b8ad58493e1a9c91181c3'
```
```JSON
{
    "dhcp": {
        "enable": true,
        "minAddress": "192.168.0.10",
        "maxAddress": "192.168.0.254",
        "subnetMask": "255.255.255.0",
        "leaseTime": 86400,
        "maxIps": 254,
        "allowedIps": [
            {
                "lanAllowedSubnetIp": "192.168.0.1",
                "lanAllowedSubnetMask": "255.255.255.0"
            }
        ],
        "blockedIps": [
            {
                "lanBlockedSubnetIp": "x.x.x.x",
                "lanBlockedSubnetMask": "0.0.0.0"
            },
            {
                "lanBlockedSubnetIp": "xxx.xxx.xxx.xxx",
                "lanBlockedSubnetMask": "255.255.252.0"
            },
            {
                "lanBlockedSubnetIp": "xx.xx.xxx.xxx",
                "lanBlockedSubnetMask": "255.255.252.0"
            },
            {
                "lanBlockedSubnetIp": "xxx.xxx.x.x",
                "lanBlockedSubnetMask": "255.255.255.0"
            },
            {
                "lanBlockedSubnetIp": "xxx.xxx.xxx.xxx",
                "lanBlockedSubnetMask": "255.255.255.252"
            },
            {
                "lanBlockedSubnetIp": "xxx.xxx.xx.x",
                "lanBlockedSubnetMask": "255.255.255.0"
            },
            {
                "lanBlockedSubnetIp": "xxx.xxx.xxx.xxx",
                "lanBlockedSubnetMask": "255.255.255.248"
            },
            {
                "lanBlockedSubnetIp": "xxx.xxx.xxx.x",
                "lanBlockedSubnetMask": "255.255.255.0"
            },
            {
                "lanBlockedSubnetIp": "xxx.xxx.xxx.xxx",
                "lanBlockedSubnetMask": "255.255.255.248"
            },
            {
                "lanBlockedSubnetIp": "xxx.xxx.xxx.xxx",
                "lanBlockedSubnetMask": "255.255.255.252"
            },
            {
                "lanBlockedSubnetIp": "xxx.xxx.x.x",
                "lanBlockedSubnetMask": "255.255.255.0"
            },
            {
                "lanBlockedSubnetIp": "xxx.xxx.x.x",
                "lanBlockedSubnetMask": "255.255.255.0"
            }
        ]
    }
}
```
### List Connected Devices
```console
curl --location --request GET 'http://192.168.0.1/rest/v1/network/hosts?connectedOnly=true' \
--header 'Accept-Language: en-GB,en;q=0.9,en-US;q=0.8' \
--header 'Authorization: Bearer e7a3c625616b8ad58493e1a9c91181c3'
```
```JSON
{
    "hosts": {
        "hosts": [
            {
                "macAddress": "B9:1E:BC:42:90:15",
                "config": {
                    "connected": true,
                    "deviceName": "",
                    "deviceType": "unknown",
                    "hostname": "unknown",
                    "interface": "ethernet",
                    "speed": 1000,
                    "ethernet": {
                        "port": 4
                    },
                    "ipv4": {
                        "address": "192.168.0.111",
                        "leaseTimeRemaining": 80492
                    }
                }
            },
            {
                "macAddress": "DE:3D:04:5C:7E:F3",
                "config": {
                    "connected": true,
                    "deviceName": "",
                    "deviceType": "unknown",
                    "hostname": "unknown",
                    "interface": "wifi",
                    "speed": 390,
                    "wifi": {
                        "band": "band5g",
                        "ssid": "VM1234567890_5G",
                        "rssi": -75
                    },
                    "ipv4": {
                        "address": "192.168.0.144",
                        "leaseTimeRemaining": 70714
                    }
                }
            }
        ]
    }
}
```
