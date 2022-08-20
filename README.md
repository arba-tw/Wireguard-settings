# Wireguard-settings (Wireguard的設定)

Device_A:Home_Nas <br>&nbsp;&nbsp;
    eth0:192.168.0.11/24 <br>&nbsp;&nbsp;
    route:0.0.0.0/0 -> 192.168.0.1 <br>&nbsp;&nbsp;
    route:172.16.0.0/12 -> 192.168.0.101

Device_B:Wireguard_router <br>&nbsp;&nbsp;
    eth0:192.168.0.101/24 <br>&nbsp;&nbsp;
    wg1:172.16.0.101/32 <br>&nbsp;&nbsp;
    route:0.0.0.0/0 -> 192.168.0.1 <br>&nbsp;&nbsp;
    ```
    vi /etc/sysctl.conf
    ```
    <br> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
    ```
      net.ipv4.ip_forward=1
    ``` 
    <br>&nbsp;&nbsp;&nbsp;
    ```
    wg genkey | tee /etc/wireguard/keys/Client_Private_key | wg pubkey > /etc/wireguard/keys/Client_Public_key
    ```
    <br>&nbsp;&nbsp;
    cat /etc/wireguard/keys/Client_Private_key <br>&nbsp;&nbsp;&nbsp;
      xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx <br>&nbsp;&nbsp;
    cat /etc/wireguard/keys/Client_Public_key <br>&nbsp;&nbsp;&nbsp;
      oooooooooooooooooooooooooooooooooooooooooooooooooooooooo <br>&nbsp;&nbsp;


Device_C:Home_AP <br>&nbsp;&nbsp;
    eth0:ISP_address <br>&nbsp;&nbsp;
    route:0.0.0.0/0 -> ISP_gateway <br>&nbsp;&nbsp;
    eth1:192.168.0.1/24

Service_A:VPS <br>&nbsp;&nbsp;
    eth0:VPS_Network_address <br>&nbsp;&nbsp;
    wg0:172.16.0.1/24 <br>&nbsp;&nbsp;
    route:0.0.0.0/0 -> VPS_Network_gateway <br>&nbsp;&nbsp;
    route:172.16.0.0/24 via 172.16.0.1 <br>&nbsp;&nbsp;
    route:192.168.0.0/24 -> 172.16.0.1 <br>&nbsp;&nbsp;
    vi /etc/sysctl.conf <br>&nbsp;&nbsp;&nbsp;
      net.ipv4.ip_forward=1 <br>
