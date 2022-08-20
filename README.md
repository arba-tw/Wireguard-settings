# Wireguard-settings (Wireguard的設定)

Ubuntu 20.04

Server 伺服器端 ，設備直接有Public address

vi /etc/sysctl.conf <br>&nbsp;
net.ipv4.ip_forward=1

Device_A:Home_Nas <br>&nbsp;&nbsp;
    eth0:192.168.0.11/24
    route:0.0.0.0/0 -> 192.168.0.1
    route:172.16.0.0/12 -> 192.168.0.101
    
Device_B:Wireguard_router
    eth0:192.168.0.101/24
    wg1:172.16.0.101/32
    route:0.0.0.0/0 -> 192.168.0.1

Device_C:Home_AP
    eth0:ISP_address
    route:0.0.0.0/0 -> ISP_gateway
    eth1:192.168.0.1/24

Service_A:VPS
    eth0:VPS_Network_address
    wg0:172.16.0.1/24
    route:0.0.0.0/0 -> VPS_Network_gateway
    route:172.16.0.0/24 via 172.16.0.1
    route:192.168.0.0/24 -> 172.16.0.1
    
Device_B
