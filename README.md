On all nodes within the cluster
-----------------------------------------------------
Required packages:  
•	pcs 
•	nginx 
•	pacemaker 
•	resource-agents 
•	fence-agents-all
# yum install pcs nginx pacemaker resource-agents fence-agents-all 

Set password for the user hacluster:
# echo ‘H@cluster123’ |   passwd --stdin hacluster

Update the /etc/hosts with the fqdn: 
# vi /etc/hosts
	10.10.10.3 webserv-1
	10.10.10.4 webserv-2

Install and configure NTP:
# yum install chrony 
# vi /etc/chrony.conf 
server 0.pool.ntp.org
server 1.pool.ntp.org
# systemctl enable --now chronyd 
# systemctl status chronyd
# timedatectl set-ntp true 
# chronyc sources -c 

Start and enable the pcsd daemon:
# systemctl enable --now pcsd

Disable the nginx daemon:
# systemctl disable --now nginx 

Configure the firewall rule to allow high-availability and the http service:
# firewall-cmd --add-service=high-availability --permanent 
# firewall-cmd --add-service=http --permanent 
# firewall-cmd --add-service=https --permanent 
# firewall-cmd --reload 


On one node within the cluster
-----------------------------------------------------------
Authenticate the nodes in the cluster:
# pcs host auth -u hacluster webserv-1 webserv-2
# pcs cluster setup lb_cluster --start webserv-1 webserv-2
# pcs cluster status 
# pcs property set stonith-enabled=false 
# pcs cluster status 
# pcs resource create virtual_ip ocf:heartbeat:IPaddr2 ip=10.10.10.2 cidr_netmask=32 op monitor interval=30s - -group lb
# pcs resource create loadbalancer ocf:heartbeat:nginx configfile=/etc/nginx/nginx.conf  op monitor timeout=”5s”  interval=”5s”  - -group lb
# pcs constraint colocation add loadbalancer with virtual_ip score=INFINITY 
# pcs resource status 
# systemctl enable - -now pacemaker corosync 
# systemctl status nginx 
# pcs constraint order virtual_ip then loadbalancer 
# pcs cluster stop --all
# pcs cluster start --all
# pcs cluster enable --all
# pcs status







