## Lab1 - Useful Linux command
```
# Find my machine name
uname -a

# Find which processes use the most CPU or memory?
top

# Stop/Kill a process?
kill <process ID>

# Find out how much disk space is free?
df -h

# Find out who is logged in?
w

# Find a log of recent logins and login attempts?
vi /var/log/auth.log

# Find my IP and MAC addresses?
ifconfig

# Examine my OS name and version?
lsb_release -a 
or
hostnamectl

# Find kernel version?	
hostnamectl

# Examine which programs run at system boot time?	
ls -l /etc/init.d
or
sudo systemctl list-unit-files --type=service --state=enabled --all

# Stop a program from running at system boot time?
sudo systemctl disable servicename

# Find the list of trusted certificates installed on my system?	
ls /etc/ssl/certs

# Remove a trusted certificate from my system?
rm /usr/local/share/ca-certificates/{unnecessary_certificate}.crt
update-ca-certificates -fresh

# Compile my program?	
c compiler: gcc test.cpp
java compiler: javac test.java

# Display an object file?	
objdump -d test.o

# Start gdb?	
gdb <program>

# gdb: Set a breakpoint?	
b <function name>

# gdb: Show registers information?	
info reg

# gdb: Present stack values?	
info frame

# List all open network connections?	
ss

# Find the process responsible for each open network connection?	
ss -p 
ex) ss -p | grep firefox

# Find the binary executable responsible for each open network connection?	
1.	find the PID using ‘ss -p | grep firefox’
2.	It shows PID of firefox
3.	And then ‘ps -aux <PID>’ to find out what the executable file is

# Reset my network interface?	
sudo /etc/init.d/networking restart

# Find my default IP gateway?	
ip route

# Find my default name server?	
grep "nameserver" /etc/resolv.conf

# Examine contents of the ARP cache?	
arp -a

# Add an entry to the ARP cache?	
sudo arp -s <host ip> <mac address>
ex) sudo arp -s 10.0.0.2 00:0c:29:c0:94:bf

# Examine contents of the DNS cache?	
sudo killall -USR1 systemd-resolved
sudo journalctl -u systemd-resolved > ~/dns-cache.txt
less ~/dns-cache.txt

# Make a local DNS query respond with an IP of my choosing?	
nslookup somewhere.com some.dns.server
ex) nslookup google.com som.dns.server

# Bring the most recent suspended job to the foreground?	
fg

# List and resume stopped jobs in the background?	
list : jobs
resume : fg <job number>

# List files opened by processes?	
lsof
```
