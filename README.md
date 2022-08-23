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
        * `user@WG_R1:~$ ip addr show`
            * eth0:192.168.0.101/24
            * wg1:172.16.0.101/32
    * 路由設定：
        * `user@WG_R1:~$ ip route show`
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
        # wg1 interface
        #---------------------------------------------
        [Interface]
        Address = 172.16.0.101/32
        MTU = 1384
        ListenPort = 55555
        PrivateKey = 12345xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
        Table = auto

        #=============================================
        # Wireguard Server
        #---------------------------------------------
        [Peer]
        PublicKey = 54321ooooooooooooooooooooooooooooooooooooooooooooooooooo
        AllowedIPs = 0.0.0.0/0
        Endpoint = 2.3.4.5:55555
        PersistentKeepalive = 25
        ```



    * 啟動：

        `user@WG_R1:~$ sudo systemctl start wg-quick@wg1.service`

    * 檢查狀態：

        `user@WG_R1:~$ sudo systemctl status wg-quick@wg1.service`

    * 開機啟動：

        `user@WG_R1:~$ sudo systemctl enable wg-quick@wg1.service`


* **Device_C:Home_AP**
    * 介面設定：
        * eth0:1.2.3.4/24
        * eth1:192.168.0.1/24
    * 路由設定：
        * route:0.0.0.0/0 -> 1.2.3.1
* **Service_A:VPS**
    * 介面設定：
        * eth0:2.3.4.5/24
        * wg0:172.16.0.1/24
    * 路由設定：
        * route:0.0.0.0/0 -> 2.3.4.1
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
    
    
        `user@WG_R0:~$ sudo vi /etc/wireguard/wg0.conf`

        ```c++
        #=============================================
        # wg0 interface
        #---------------------------------------------
        [Interface]
        Address = 172.16.0.1/24
        MTU = 1420
        ListenPort = 55555
        PrivateKey = 54321xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
        SaveConfig = false
        PostUp = /etc/wireguard/iptables/add-nat-routing.sh
        PostDown = /etc/wireguard/iptables/remove-nat-routing.sh

        #=============================================
        # Wireguard Client
        #---------------------------------------------
        [Peer]
        PublicKey = 12345ooooooooooooooooooooooooooooooooooooooooooooooooooo
        AllowedIPs = 172.21.32.31/32,192.168.32.0/24
        ```


    * 設定iptables

        * `user@WG_R0:~$ sudo vi /etc/wireguard/iptables/add-nat-routing.sh`

        ```bash
        #!/bin/bash

        IPT="/sbin/iptables"
        IN_FACE="eth0"
        WG_FACE="wg0"
        SUB_NET="172.16.0.0/24"
        WG_PORT="55555"

        $IPT -t nat -I POSTROUTING 1 -s $SUB_NET -o $IN_FACE -j MASQUERADE
        $IPT -I INPUT 1 -i $WG_FACE -j ACCEPT
        $IPT -I FORWARD 1 -i $IN_FACE -o $WG_FACE -j ACCEPT
        $IPT -I FORWARD 1 -i $WG_FACE -o $IN_FACE -j ACCEPT
        $IPT -I INPUT 1 -i $IN_FACE -p udp --dport $WG_PORT -j ACCEPT
        ```
        * `user@WG_R0:~$ sudo vi /etc/wireguard/iptables/add-nat-routing.sh`

            ```bash
            #!/bin/bash

            IPT="/sbin/iptables"
            IN_FACE="eth0"
            WG_FACE="wg0"
            SUB_NET="172.16.0.0/24"
            WG_PORT="55555"

            $IPT -t nat -I POSTROUTING 1 -s $SUB_NET -o $IN_FACE -j MASQUERADE
            $IPT -I INPUT 1 -i $WG_FACE -j ACCEPT
            $IPT -I FORWARD 1 -i $IN_FACE -o $WG_FACE -j ACCEPT
            $IPT -I FORWARD 1 -i $WG_FACE -o $IN_FACE -j ACCEPT
            $IPT -I INPUT 1 -i $IN_FACE -p udp --dport $WG_PORT -j ACCEPT
            ```

    * 啟動：

        `user@WG_R0:~$ sudo systemctl start wg-quick@wg0.service`

    * 檢查狀態：

        `user@WG_R0:~$ sudo systemctl status wg-quick@wg0.service`

    * 開機啟動：

        `user@WG_R0:~$ sudo systemctl enable wg-quick@wg0.service`

## 參考(Reference)

* https://www.wireguard.com/quickstart/
* 