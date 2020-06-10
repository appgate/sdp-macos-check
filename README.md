# Overview
This tool is used to be run on macOS. It provides various commands to retrieve information from the end-user device. This is used when creating policy assignments or criteria scripts in AppGate SDP.

For more information regarding device scripts and extensions in AppGate SDP please read the 

## Binary
See [latest release]().

## Commands
### Check
```
$./chkmos check
  -isAppInstalled string
    	Check if argument name is found amongst installed apps; case insensitive but must be exact name. Returns 'yes' OR 'no'
  -isDiskEncrypted
    	Is /System/Volumes/Data/ encrypted. Returns 'yes' OR 'no' OR 'inderterminable'
  -isFirewallOn
    	Check if firewall on: Returns 'yes' OR 'no' OR 'inderterminable'
  -isProcessRunning string
    	Check if argument is substring among all process exe names. Returns a JSON with all matched:  PID, PPID, Executable name
  -osVersion
    	Get OS Product Version (or returns 'inderterminable')
```

### info
``` 
$./chkmos info
  -app
    	Print information about installed applications in JSON
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
Return values can be `yes|no`, single text values or JSON strings which can be parsed in the scripts on AppGate.


