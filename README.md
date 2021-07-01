# Powershell_stuff
Useful set of Powershell related commands during engagements

#ENABLE TLS1.2 if you receive a message Regarding an ISSUE with TLS
[Net.ServicePointManager]::SecurityProtocol = [Net.ServicePointManager]::SecurityProtocol -bor [Net.SecurityProtocolType]::Tls12



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

