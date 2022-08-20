# Wireguard-settings (Wireguard的設定)

Device_A:Home_Nas <br>&nbsp;&nbsp;
    介面設定： <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
    - eth0:192.168.0.11/24 <br>&nbsp;&nbsp;
    路由設定： <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
    - route:0.0.0.0/0 -> 192.168.0.1 <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
    - route:172.16.0.0/12 -> 192.168.0.101

Device_B:Wireguard_router <br>&nbsp;&nbsp;
    介面設定：  <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
    - eth0:192.168.0.101/24 <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
    - wg1:172.16.0.101/32  <br>&nbsp;&nbsp;
    路由設定： <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
    - route:0.0.0.0/0 -> 192.168.0.1 <br>&nbsp;&nbsp;
    封包轉送設定：
<pre><code>vi /etc/sysctl.conf
  net.ipv4.ip_forward=1 </code></pre>
&nbsp;&nbsp;Wireguard的Key產生：
<pre><code>wg genkey | tee /etc/wireguard/keys/Client_Private_key | wg pubkey > /etc/wireguard/keys/Client_Public_key </code></pre>

<pre><code>cat /etc/wireguard/keys/Client_Private_key
  54321xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

cat /etc/wireguard/keys/Client_Public_key
  54321ooooooooooooooooooooooooooooooooooooooooooooooooooo </code></pre>

Device_C:Home_AP <br>&nbsp;&nbsp;
    eth0:ISP_address <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
    route:0.0.0.0/0 -> ISP_gateway <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
    eth1:192.168.0.1/24

Service_A:VPS <br>&nbsp;&nbsp;
    eth0:VPS_Network_address <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
    wg0:172.16.0.1/24 <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
    route:0.0.0.0/0 -> VPS_Network_gateway <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
    route:172.16.0.0/24 via 172.16.0.1 <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
    route:192.168.0.0/24 -> 172.16.0.1

<pre><code>vi /etc/sysctl.conf
  net.ipv4.ip_forward=1 </code></pre>
<pre><code>wg genkey | tee /etc/wireguard/keys/Client_Private_key | wg pubkey > /etc/wireguard/keys/Client_Public_key </code></pre>

<pre><code>cat /etc/wireguard/keys/Client_Private_key
  12345xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
cat /etc/wireguard/keys/Client_Public_key
  12345ooooooooooooooooooooooooooooooooooooooooooooooooooo </code></pre>
