# Overview
This tool is used to be run on macOS. It provides various commands to retrieve information from the end-user device. This is used when creating policy assignments or criteria scripts in AppGate SDP.

For more information regarding device scripts and extensions in AppGate SDP please read the [sdp extensions](https://github.com/appgate/sdp-extensions) 

## Binary
See [latest release](https://github.com/appgate/sdp-macos-check/releases/latest).

## Commands
### Check
```
$./chkmos check
  -appRunning string
    	Check if <argument> bundle-name of App is installed and process exists according to bundles exe name. Returns 'yes' OR 'no' if installed, otherwise 'inderterminable'
  -isAppInstalled string
    	Check if <argument> name is found amongst installed apps; case insensitive but must be exact name. Returns 'yes' OR 'no'
  -isCertInstalled string
    	Check if <argument> label is found among all installed certificates. Returns 'yes' OR 'no'.  'inderterminable' is returned if non conclusive match, e.g more than one, is found.
  -isDiskEncrypted
    	Is /System/Volumes/Data/ encrypted. Returns 'yes' OR 'no' OR 'inderterminable'
  -isFirewallOn
    	Check if firewall on: Returns 'yes' OR 'no' OR 'inderterminable'
  -isProcessRunning string
    	Check if <argument> is substring among all process exe names. Returns a JSON with all matched:  PID, PPID, Executable name
  -osVersion
    	Get OS Product Version (or returns 'inderterminable')
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
```


## Return values
Return values can be `yes|no` (but can be also `inderterminable`), single text values or JSON strings which can be parsed in the scripts on AppGate.


