# Wireguard-settings (Wireguard的設定)

## 架構

NAS <--> Wireguard_router < == > Home_AP < == > Internet < == > VPS 

##    設備
* **Device_A:Home_Nas**
    * 介面設定：
        * eth0:192.168.0.11/24
    * 路由設定：
        * route:0.0.0.0/0 -> 192.168.0.1
        * route:172.16.0.0/12 -> 192.168.0.101

* **Device_B:Wireguard_router**
    * 介面設定：
        * eth0:192.168.0.101/24
        * wg1:172.16.0.101/32
    * 路由設定：
        * route:0.0.0.0/0 -> 192.168.0.1
    * 封包轉送設定：

        `user@WG_R1:~$ sudo vi /etc/sysctl.conf`
        
        `net.ipv4.ip_forward=1`

    * Wireguard的Key產生：

        <pre><code>user@WG_R1:~$ wg genkey | tee /etc/wireguard/keys/Client_Private_key | wg pubkey > /etc/wireguard/keys/Client_Public_key</code></pre>

        <pre><code>user@WG_R1:~$ cat /etc/wireguard/keys/Client_Private_key
      12345xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx</code></pre>

        <pre><code>user@WG_R1:~$ cat /etc/wireguard/keys/Client_Public_key
      12345ooooooooooooooooooooooooooooooooooooooooooooooooooo</code></pre>

    * 設定Wireguard：
    
        `user@WG_R1:~$ sudo vi /etc/wireguard/wg1.conf`

        ```c++
        #=============================================
        # Client's wg interface for
        #---------------------------------------------
        [Interface]
        Address = 172.16.0.101/32
        MTU = 1384
        ListenPort = 55555
        PrivateKey = 12345xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
        Table = auto

        #=============================================
        # Our Wireguard Server
        #---------------------------------------------
        [Peer]
        PublicKey = 54321ooooooooooooooooooooooooooooooooooooooooooooooooooo
        AllowedIPs = 0.0.0.0/0
        Endpoint = "VPSNetworkaddress":55555
        PersistentKeepalive = 25
        ```



    * 啟動：

        `user@WG_R1:~$ sudo systemctl start `

    * 檢查狀態：

        `user@WG_R1:~$ sudo systemctl status `

    * 開機啟動：

        `user@WG_R1:~$ sudo systemctl enable `


* **Device_C:Home_AP**
    * 介面設定：
        * eth0:ISP_address
        * eth1:192.168.0.1/24
    * 路由設定：
        * route:0.0.0.0/0 -> ISP_gateway
* **Service_A:VPS**
    * 介面設定：
        * eth0:VPS_Network_address
        * wg0:172.16.0.1/24
    * 路由設定：
        * route:0.0.0.0/0 -> VPS_Network_gateway
        * route:172.16.0.0/24 via 172.16.0.1
        * route:192.168.0.0/24 -> 172.16.0.1
   * 封包轉送設定：

        `user@WG_R0:~$ sudo vi /etc/sysctl.conf`
        
        `net.ipv4.ip_forward=1`

   * Wireguard的Key產生：

        <pre><code>user@WG_R0:~$ wg genkey | tee /etc/wireguard/keys/Server_Private_key | wg pubkey > /etc/wireguard/keys/Server_Public_key</code></pre>

        <pre><code>user@WG_R0:~$ cat /etc/wireguard/keys/Server_Private_key
      54321xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx</code></pre>

        <pre><code>user@WG_R1:~$ cat /etc/wireguard/keys/Server_Public_key
      54321ooooooooooooooooooooooooooooooooooooooooooooooooooo</code></pre>


    * 設定Wireguard
    
    
        `user@WG_R1:~$ sudo vi /etc/wireguard/wg1.conf`

        ```c++
        #=============================================
        # Client's wg interface for
        #---------------------------------------------
        [Interface]
        Address = 172.16.0.101/32
        MTU = 1384
        ListenPort = 55555
        PrivateKey = 12345xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
        Table = auto

        #=============================================
        # Our Wireguard Server
        #---------------------------------------------
        [Peer]
        PublicKey = 54321ooooooooooooooooooooooooooooooooooooooooooooooooooo
        AllowedIPs = 0.0.0.0/0
        Endpoint = "VPSNetworkaddress":55555
        PersistentKeepalive = 25
        ```


    * 設定iptables

        `user@WG`
        
## 參考(Reference)
    