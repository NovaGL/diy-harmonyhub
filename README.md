# Harmony Hub HTTP & Websockets API
This repo explains how to access the new Harmony Hub API over Websockets and HTTP POST.

Harmony Hubs have port 8088 open which can be used for issuing commands over HTTP POST or Websocket.
This is a simple guide of what sorts of commands are available, it is not a definative list. 
Please feel free to add any missing information via a PR.

HTTP POST Commands
------
All HTTP commands are send to the following address: http://HUB_IP:8088

You need to supply the following headers for the request to be authenticated

```
Origin: http//:localhost.nebula.myharmony.com
Content-Type	application/json
Accept	utf-8
```

You can now issue the following commands by sending different messages in the body. 
Change the ID to a random number.

##### PING
REQUEST
````json
{"id":0,"cmd":"connect.ping","timeout":5000}
````

RESPONSE
````json
{
	"id": 0,
	"msg": "OK",
	"data": {
		"status": "alive",
		"uuid": "XXXX",
		"susTrigger": "hbus_http",
		"id": "0"
	},
	"code": "200"
}
````

##### ACCOUNT INFORMATION
REQUEST
````json
{"id":29549457,"cmd":"setup.account?getProvisionInfo","timeout":90000}
````

RESPONSE
````json
{
	"id": 0,
	"msg": "OK",
	"data": {
		"discoveryServerCF": "Discovery.svc",
		"email": "my@email.com",
		"username": "my@email.com",
		"discoveryServer": "/Discovery.svc",
		"activeRemoteId": "[MyRemoteID]",
		"susChannel": "preview",
		"mode": 3,
		"accountId": "[MyAccountID]"
	},
	"code": "200"
}
````

For the following commands use the remote ID which is obtained from the above command "activeRemoteId" where required.

##### GET ACTIVITIES
REQUEST
````json
{"id":0,"cmd":"proxy.resource?get","params":{"uri":"harmony://Account/[MyRemoteID]/ActivityList"},"timeout":90000}
````

RESPONSE
````json
{
"id": 0,
"msg": "OK",
"data":{
"hetag": "XXX",
"resource":{
"Activities":[{"DefaultStationName": null, "IsTuningDefault": false, "AccountId-": "XXX", "Alternatives": null}]
},
"code": "200",
"etag": "XXXX",
"uri": "harmony://Account/XXXX/ActivityList"
},
"code": "200"
}
````

JSONata to filter results
Copy the response (from the above request) into the left side and the following query on the right side at this site: http://try.jsonata.org/

````
{
    "Activities": $spread(data.resource.Activities {
        Name: [
            $ ~> | $ | {}, "Name" |            
        ]
    }).{
        "Name": $keys()[0],
        "ActivityId":*."Id-"
    }
}
````

RESULTS
````json
{
  "Activities": [
    {
      "Name": "Watch TV",
      "ActivityId": 0000
    }
]}
````

##### START ACTIVITY
Find the activity number from the above command.
````json
{"cmd": "harmony.activityengine?runactivity", "params":{"activityId":"-1"}}
````

##### GET DEVICES
REQUEST
````json
{"id":0,"cmd":"proxy.resource?get","params":{"uri":"harmony://Account/[MyRemoteID]/DeviceList"},"timeout":90000}
````

RESPONSE
````json
{
   "id":0,
   "msg":"OK",
   "data":{
      "hetag":"xxxx",
      "resource":{
         "MaxActivities":null,
         "Capabilities":[
            {

            }
         ],
         "MaxDevices":8,
         "DisplayName":"Harmony Smart Control"
      },
      "code":"200",
      "etag":"XXX",
      "uri":"harmony:\/\/Account\/[MyRemoteID]\/CapabilityList"
   },
   "code":"200"
}
````

JSONata to filter results
Copy the response (from the above request) into the left side and the following query on the right side at this site: http://try.jsonata.org/

````
{
    "Devices": $spread(data.resource.DevicesWithFeatures.Device {
        Name: [
            $ ~> | $ | {}, "Name" |            
        ]
    }).{
        "Name": $keys()[0],
        "DeviceId":*."Id-"
    }
}
````

RESULTS
````json
{
  "Devices": [
    {
      "Name": "Watch TV",
      "DeviceId": 0000
    }
]}
````
#### DEVICE BUTTON PRESS
Status can either be press, hold or release.

REQUEST
````json
{"cmd":"harmony.engine?holdAction","timeout": 60,"params":{"status":"press","timestamp":"0","verb":"render","action": "{\"command\":\"Mute\",\"type\":\"IRCommand\",\"deviceId\":\"[DEVICEID]\"}"}}
````

RESPONSE
````json
{
    "msg": "OK",
    "data": {
        "deviceId": "[DEVICEID]"
    },
    "code": 200
}
````

##### HUB INFORMATION
REQUEST
````json
{"id":24269474,"cmd":"connect.sysinfo?get","timeout":10000}
````

RESPONSE
````json
{
	"id": 24269474,
	"msg": "OK",
	"data": {
		"link_packet_length": 64,
		"link_hw": "usb",
		"arch": "0x11",
		"ram_size": "32MB",
		"fw_type": "0x00",
		"bt_addr": "XXXX",
		"usb_product_id": "0xc129",
		"feature": ["Infrared", "BluetoothHid"],
		"serial_number": "XXXX",
		"fw_ver": "4.15.206",
		"skin": "0x61",
		"hw_ver": "01.00",
		"usb_vendor_id": "0x046d",
		"link_type": "hid",
		"status": "normal"
	},
	"code": "200"
}
````

##### HUB CAPABLITITIES
REQUEST
````json
{"id":76452280,"cmd":"proxy.resource?get","params":{"uri":"harmony://Account/[MyREMOTEID]/CapabilityList"},"timeout":90000}
````
RESPONSE (TRIMMED)
````json
{
	"id": 0,
	"msg": "OK",
	"data": {
		"hetag": "xxxx",
		"resource": {
			"MaxActivities": null,
			"Capabilities": [{}],
      "MaxDevices": 8,
			"DisplayName": "Harmony Smart Control"
		},
		"code": "200",
		"etag": "XXX",
		"uri": "harmony:\/\/Account\/[MyRemoteID]\/CapabilityList"
	},
	"code": "200"
}
````

##### HOME AUTOMATION CONFIGURATION
REQUEST
````json
{"id":0,"cmd":"proxy.resource?get","params":{"uri":"dynamite://HomeAutomationService/Config/"},"timeout":90000}
````

RESPONSE
````json
{
	"id": 0,
	"msg": "OK",
	"data": {
		"hetag": "XXX",
		"resource": {
			"programs": [],
      "maps": {},
      "unknownDevices": [],
      "gateways": {},
      "devices": {},
      "groups": {},
      "account_uri": "svcs.myharmony.com",
			"scenes": [],
			"scenes2": []
		},
		"code": "200",
		"etag": "XXX",
		"uri": "dynamite:\/\/HomeAutomationService\/Config\/"
	},
	"code": "200"
}
````

##### HOME AUTOMATION STATUS
REQUEST
````json
{"id":0,"cmd":"harmony.automation?status","params":{},"timeout":90000}
````

RESPONSE
```json
{
	"id": 0,
	"msg": "OK",
	"data": {
		"XXXX": {
			"errorStatus": 0
		}
	},
	"code": "200"
}
````

#### MISC

````json
{"id":23771041,"cmd":"connect.rf?info","timeout":90000}
````

````json
{"id":75689134,"cmd":"setup.firmware?check","timeout":45000}

````
