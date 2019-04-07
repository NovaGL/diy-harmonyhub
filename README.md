# Harmony Hub HTTP & Websockets API
This repo explains how to access the new Harmony Hub API over Websockets and HTTP POST.

Harmony Hubs have port 8088 open which can be used for issuing commands over HTTP POST or Websocket.
This is a simple guide of what sorts of commands are available, it is not a definative list. 
Please feel free to add any missing information via a PR.

HTTP POST Commands
------
All HTTP commands are sent to the following address: 
````http://HUB_IP:8088````

You need to supply the following headers for the request to be authenticated

```
Origin: http://sl.dhg.myharmony.com
Content-Type	application/json
Accept	utf-8
```

You can now issue the following commands by sending different messages in the body. 
Change the ID to a random number.

****ONLY ACCOUNT INFORMATION IS CURRENTLY WORKING SINCE THE LATEST UPDATE****

#### ACCOUNT INFORMATION
------
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
		"susChannel": "production",
		"mode": 3,
		"accountId": "[MyAccountID]"
	},
	"code": "200"
}
````
Websocket Commands
------

For the following commands use the remote ID which is obtained from the above command "activeRemoteId" where required.
All Websocket commands are sent to the following address: 

```http://HUB_IP:8088/?domain=svcs.myharmony.com&hubId=[MyREMOTEID]```

### CONFIGURATION
------
All the information we need can be found by issuing the following command it shows all the devices and activites in the response.

REQUEST

````json
{
	"hubId": "[MyRemoteID]",
	"timeout": 60,
	"hbus": {
		"cmd": "vnd.logitech.harmony\/vnd.logitech.harmony.engine?config",
		"id": "0",
		"params": {
			"verb": "get"
		}
	}
}
````

RESPONSE
````json
{
  "cmd": "vnd.logitech.harmony\/vnd.logitech.harmony.engine?config",
  "code": 200,
  "id": "0",
  "msg": "OK",
  "data": {
    "dataConsent": false,
    "sequence": [],
	"global": {},
	"device": [{}],
	"sla": {},
	"content": {},
	"activity": [{}],
	}
}
````

Use JSONata to filter results.

Copy the response (from the above request) into the left side and the following query on the right side at this site: http://try.jsonata.org/

````
{
  "Activity": data.activity.({
  $.label:$.id  
  }),
  "Devices": data.device.({
  $.label:$.contentProfileKey  
  })
}
````


### ACTIVITIES
------

##### START ACTIVITIES
REQUEST

```json
{
	"hubId": "[MyREMOTEID]",
	"timeout": 60,
	"hbus": {
		"cmd": "vnd.logitech.harmony\/vnd.logitech.harmony.engine?config",
		"id": "0",
		"params": {
			"async": "true",
			"timestamp": 0,
			"args": {
				"rule": "start"
			},
			"activityId": "-1"
		}
	}
}
```

RESPONSE
```json
{
  "cmd": "vnd.logitech.harmony\/vnd.logitech.harmony.engine?config",
  "code": 200,
  "id": "0",
  "msg": "OK",
    "data": {}
}
```

### DEVICES
------

##### DEVICE BUTTON PRESS
Status can either be press, hold or release.

REQUEST
```json
{
	"hubId": "[MyREMOTEID]",
	"timeout": 30,
	"hbus": {
		"cmd": "vnd.logitech.harmony/vnd.logitech.harmony.engine?holdAction",
		"id": "0",
		"params": {
			"status": "press",
			"timestamp": "0",
			"verb": "render",
			"action": "{\"command\":\"VolumeDown\",\"type\":\"IRCommand\",\"deviceId\":\"999\"}"
		}
	}
}
```

RESPONSE
```json
{
  "type": "harmonyengine.metadata?notify",
  "data": {}
}
```

### HOME AUTOMATION
------

##### GET HOMEAUTOMATION STATE
REQUEST
````json
{"hubId":"[MyREMOTEID]","hbus":{"id":"1555605542","cmd":"harmony.automation?getstate","params":{}}} 
````

RESPONSE
```json
{
	"cmd": "harmony.automation?getstate",
	"code": 200,
	"id": "1555605542",
	"msg": "OK",
	"data": {

		"hue-light.harmony_virtual_button_1": {
			"color": {
				"mode": "xy",
				"xy": {
					"y": 0,
					"x": 0
				},
				"temp": 300,
				"hueSat": {
					"hue": 0,
					"sat": 0
				}
			},
			"brightness": 254,
			"on": false,
			"status": 0
		},

	}
}
````
##### SET HOMEAUTOMATION STATE

Allows you to change the power, brightness and colour by adding the same fields as used from the get request.

REQUEST
````json
{"hubId":"[MyREMOTEID]","hbus":{"id":"1794236100","cmd":"harmony.automation?setstate","params":{"state":{"hue-light.harmony_virtual_button_1":{"on":true}}}}}
````

RESPONSE
```json
{
	"cmd": "harmony.automation?setstate",
	"code": 200,
	"id": "968549983",
	"msg": "OK"
}
````
THEN FOWLLOWED BY 
```json
{
	"type": "automation.state?notify",
	"data": {
		"hue-light.harmony_virtual_button_1": {
			"color": {
				"mode": "xy",
				"xy": {
					"y": 0,
					"x": 0
				},
				"temp": 300,
				"hueSat": {
					"hue": 0,
					"sat": 0
				}
			},
			"brightness": 254,
			"on": true,
			"status": 0
		}
	}
}
````

Deprecated Commands
------

All the following commands no longer work with the current version, however they share similar traits with Web socket commands and can be adjusted to work via Web Socket.

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
  "Devices": data.resource.Activities.({
  "Name":$.Name,
  "ActivityId": $."Id-",  
  })
}
````

JSONATA RESULTS
````json
{
  "Activities": [
    {
      "Name": "Watch TV",
      "ActivityId": 0000
    }
]}
````

##### GET ACTIVITY DEVICE COMMANDS
In addition to the above way to get Activities you can also get Activites and Device information via the hub configuration

REQUEST
````json
{"id": 0,"cmd": "harmony.engine?config","timeout": 90000}
````

RESPONSE
````json
{
   "id":23771041,
   "msg":"OK",
   "data":{
      "dataConsent":false,
      "sequence":[

      ],
      "global":{ },
      "device":[ ],
      "sla":{ },
      "content":{ },
      "activity":[ ]
   },
   "code":"200"
}
````

Filter the results to only show activity names and device commands
````
{
  "Activity": data.activity.({
  $.label:$.id,
  "DeviceCommands": $.controlGroup.function.action
  })
}
````

You are then left with a much smaller list to look through.

```json
{
   "Activity":[
      {
         "Activity Name":"Activity ID",
         "DeviceCommands":[ ]
      }
    ]
}
````

##### GET CURRENT ACTIVITY
To get the Current Activity running use the body.

REQUEST
````json
{"cmd":"harmony.engine?getCurrentActivity"}
````

RESPONSE

````json
{
	"msg": "OK",
	"data": {
		"result": "-1"
	},
	"code": "200"
}
````

-1 indicates no activity is running otherwise it will show the number related to an activity such as Watch TV

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
         "Capabilities":[{}],
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
  "Devices": data.resource.DevicesWithFeatures.({
  "Name":$.Device.Name,
  "DeviceId": $.Device."Id-",
  "Commands": $.Commands.Name
  })
}
````

RESULTS
````
{
  "Devices": [
    {
      "Name": "Watch TV",
      "DeviceId": 0000,
      "Commands": []
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
