# Overview
This tool is used to be run on macOS. It provides various commands to retrieve information from the end-user device. This is used when creating policy assignments or criteria scripts in Appgate SDP.

For more information regarding device scripts and extensions in Appgate SDP please read the [sdp extensions](https://github.com/appgate/sdp-extensions) 

## License
The project sdp-macos-check is licensed under the MIT license.

## Binary
See [latest release](https://github.com/appgate/sdp-macos-check/releases/latest).

## Commands
### Check
```
$./chkmos check 
 -appRunning string
    	Check if <bundle-id> (used as substring in search for bundle/app) is installed and process exists according to bundle's exec name. Returns JSON with found processe(s)
  -isAppInstalled string
    	Check if <name> is found amongst installed apps; case insensitive but must be exact name. Returns 'yes' OR 'no'
  -isCertInstalled string
    	Check if <name> is found among all installed certificates. Returns 'yes' OR 'no'.  'indeterminable' is returned if non conclusive match, e.g more than one, is found.
  -isDiskEncrypted
    	Is /System/Volumes/Data/ encrypted. Returns 'yes' OR 'no' OR 'indeterminable'
  -isFirewallOn
    	Check if firewall on: Returns 'yes' OR 'no' OR 'indeterminable'
  -isProcessRunning string
    	Check if <exec name> is substring among all process exe names. Returns a JSON with all matched:  PID, PPID, Executable name
  -osVersion
    	Get OS Product Version (or returns 'indeterminable')
  -publicIP
    	Print the public IP. Internally several providers are used to assure a quorum of the IP. If no quorum is achieved or not possible to retrieve 'indeterminable' is returned
```

### info
``` 
$./chkmos info
  -activeDirectory
    	Prints information on the machines active directory membership in JSON if any, otherwise empty.
  -apps
    	Print information about installed applications in JSON
  -certs
    	Print all names (labl) of installed certificates in JSON
  -disk
    	Print disk information in JSON
  -firewall
    	Print firewall information in JSON
  -os
    	Print OS information in JSON
  -processes
    	Print all running processes in JSON
  -publicIP
    	Lookup and print public IP from multiple reply services. Reports in JSON
```


## Return values
Return values can be `yes|no` (but can be also `indeterminable`), single text values or JSON strings which can be parsed in the scripts on Appgate.

# Examples
## Test if certain AV is installed
MacOS does not has a security center unlike Windows. So There are different ways to identify if a known AV is running on the device, example:
1. know the process name
1. know the app name

Note: this example can not verify any status of the running AV such as signature date or of it is paused or not. It simply detects if a real-time protection process of the known AV exists or not.

### By process name
If you know the process name then you would look for a running process. In the case of MalwareBytes we know that the real time protection process is called `RTProtectionDaemon`, so one could create a test like the following:
```shell
./chkmos check -isProcessRunning RTProtectionDaemon
```
The result comes with a JSON string:
```json
[
  {
    "executable": "RTProtectionDaem",
    "pid": 184,
    "ppid": 1
  }
]
```
The result can be consumed on a JS expression on appgate for policy asignement and entitlement's conditions. 

### By app name
Of course the names of processes might change over the versions of a product-- only the vendor has control over that. Another approach is to use the `check -appRunning` command which takes the bundle ID of an application then verify if such is installed and if so use it's executable file name to find the process. This might return more results, and the fine-tuning can be done in the JS expression rather than in the deployment of the device-script which allows a more control.

First: know or find a proper bundle ID which can be assumed working well with the AV vendor. Assuming with the example of MalwareBytes, lets check out what `chkmos` finds:

```shell
./chkmos info -apps |jq |less
```
I can find several entries referring to MalwareBytes, similar to `bundleIdentifier": "com.malwarebytes.mbam.frontend.app.kit`
Now, for the bundle identifier in my final query I will only need a substring referring to the most common name matching the product, like `malwarebytes`.

```
./chkmos check -appRunning malwarebytes
```

Result:
```json
[
  {
    "bundleName": "FrontendAgent",
    "bundleIdentifier": "com.malwarebytes.mbam.frontend.agent",
    "executableFile": "FrontendAgent",
    "pid": 580,
    "ppid": 1
  },
  {
    "bundleName": "SettingsDaemon",
    "bundleIdentifier": "com.malwarebytes.mbam.settings.daemon",
    "executableFile": "SettingsDaemon",
    "pid": 410,
    "ppid": 1
  },
  {
    "bundleName": "RTProtectionDaemon",
    "bundleIdentifier": "com.malwarebytes.mbam.rtprotection.daemon",
    "executableFile": "RTProtectionDaemon",
    "pid": 184,
    "ppid": 1
  }
]
```
The result can be consumed on a JS expression on Appgate for policy assignment and entitlement's conditions. The interpretation is kept to the implementer, but most possible you would like to look for the "real time protection".

#### Example of a condition
The following example assumes a claim name `mbrtpmacos` as created under the identity provider.

```JavaScript
if (!claims.device.mbrtpmacos) return false; // if claim not defined abort

var rtp_json;
try {
 	rtp_json = JSON.parse(claims.device.mbrtpmacos); 
}
catch(err) {
 	return false; // could not parse/JSON err
}

for (var i = 0; i < rtp_json.length; i++){
 	var app = rtp_json[i];
    // we assume if we find the bundleName this is prove RTP is running
    if (app["bundleName"] == "RTProtectionDaemon"){
    	return true;
    }
}
return false;

```

