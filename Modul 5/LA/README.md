# Modul 5 - Vlan-Trunk-OSPF-MultiVendor

# 1. Topologi Jaringan

Topologi yang digunakan pada praktikum ini terdiri dari NET, MikroTik ISP, Addresiing Jakarta mulai dari Cisco Switch; Cisco Router; Mikrotik Router; Cisco Vios; Ubuntu Server; Fortigate; VPCS VLAN 20 10,Addresing Surabaya Cisco Switch; Mikrotik Router; Fortigate; VPCS VLAN 30 40 TinycoreVLAN 40. Topologi di bawah adalah simulasi jaringan enterprise menghubungkan 2 kantor pusat dan kantor cabang. Kantor pusat ada di Jakarta dan kantor cabang ada di Surabaya. 2 Kantor tersebut akan dihubungkan menggunakn teknologi Gre Tunel agar bisa saling berkomunikasi meskipun tidak dalam satu kantor.

## Screenshot Topologi

![Topologi](images/topologi.jpeg)


## Gambaran Topologi
### 3.1 HQ Jakarta

Sisi Jakarta terdiri dari:

* Cisco Switch Jakarta
* Cisco Router Jakarta
* MikroTik Router Jakarta
* Ubuntu Server Jakarta
* FortiGate Jakarta
* Client VLAN 10 dan VLAN 20

Cisco Router Jakarta dan MikroTik Router Jakarta digunakan sebagai dual gateway menggunakan VRRP. Ubuntu Server Jakarta digunakan sebagai DHCP Server terpusat untuk VLAN 10 dan VLAN 20. FortiGate Jakarta digunakan sebagai edge firewall, NAT gateway, dan GRE endpoint menuju Surabaya.

### 3.2 ISP

Sisi ISP disimulasikan menggunakan MikroTik RouterOS. MikroTik ISP menghubungkan FortiGate Jakarta dan FortiGate Surabaya. MikroTik ISP juga dapat dikonfigurasi NAT ke Cloud PNETLab agar perangkat internal dapat mengakses internet.

### 3.3 Branch Surabaya

Sisi Surabaya terdiri dari:

* FortiGate Surabaya
* MikroTik Router Surabaya
* Cisco Switch Surabaya
* Client VLAN 30 dan VLAN 40

FortiGate Surabaya digunakan sebagai edge firewall dan GRE endpoint. MikroTik Surabaya digunakan sebagai gateway VLAN 30 dan VLAN 40. VLAN 30 menggunakan DHCP Server dari MikroTik, sedangkan VLAN 40 menggunakan IP static.

---

# 2. Addressing Jakarta / HQ

## 2.1 VLAN Jakarta

| VLAN | Nama VLAN | Network         | Gateway Virtual | Keterangan                      |
| ---: | --------- | --------------- | --------------- | ------------------------------- |
|   10 | FINANCE   | 192.168.10.0/24 | 192.168.10.1    | DHCP dari Ubuntu Server Jakarta |
|   20 | IT        | 192.168.20.0/24 | 192.168.20.1    | DHCP dari Ubuntu Server Jakarta |
|   60 | SERVER-HQ | 192.168.60.0/24 | 192.168.60.1    | VLAN server Ubuntu Jakarta      |

---

## 2.2 IP Address Cisco Router Jakarta

| Interface |               VLAN / Link | IP Address      | Keterangan                                 |
| --------- | ------------------------: | --------------- | ------------------------------------------ |
| Gi0/1.10  |                   VLAN 10 | 192.168.10.2/24 | IP fisik Cisco untuk VLAN 10               |
| Gi0/1.20  |                   VLAN 20 | 192.168.20.2/24 | IP fisik Cisco untuk VLAN 20               |
| Gi0/1.60  |                   VLAN 60 | 192.168.60.2/24 | IP fisik Cisco untuk VLAN 60               |
| Gi0/0     | Link ke FortiGate Jakarta | 10.10.100.2/30  | Transit Cisco Jakarta ke FortiGate Jakarta |

---

## 2.3 IP Address MikroTik Router Jakarta

| Interface      |               VLAN / Link | IP Address      | Keterangan                                    |
| -------------- | ------------------------: | --------------- | --------------------------------------------- |
| vlan10-finance |                   VLAN 10 | 192.168.10.3/24 | IP fisik MikroTik untuk VLAN 10               |
| vlan20-it      |                   VLAN 20 | 192.168.20.3/24 | IP fisik MikroTik untuk VLAN 20               |
| vlan60-server  |                   VLAN 60 | 192.168.60.3/24 | IP fisik MikroTik untuk VLAN 60               |
| ether1         | Link ke FortiGate Jakarta | 10.10.101.2/30  | Transit MikroTik Jakarta ke FortiGate Jakarta |

---

## 2.4 VRRP Jakarta

| VLAN | Virtual IP   | Master                  | Backup                  | Keterangan              |
| ---: | ------------ | ----------------------- | ----------------------- | ----------------------- |
|   10 | 192.168.10.1 | Cisco Router Jakarta    | MikroTik Router Jakarta | Gateway virtual VLAN 10 |
|   20 | 192.168.20.1 | MikroTik Router Jakarta | Cisco Router Jakarta    | Gateway virtual VLAN 20 |
|   60 | 192.168.60.1 | Cisco Router Jakarta    | MikroTik Router Jakarta | Gateway virtual VLAN 60 |

---

## 2.5 Ubuntu Server Jakarta

| Perangkat             | VLAN | IP Address       | Gateway      | Service                              |
| --------------------- | ---: | ---------------- | ------------ | ------------------------------------ |
| Ubuntu Server Jakarta |   60 | 192.168.60.10/24 | 192.168.60.1 | ISC-DHCP Server dan Nginx Web Server |

---

## 2.6 DHCP Pool Jakarta

| VLAN | Network         | Range DHCP                      | Gateway yang Diberikan | DHCP Server           |
| ---: | --------------- | ------------------------------- | ---------------------- | --------------------- |
|   10 | 192.168.10.0/24 | 192.168.10.100 - 192.168.10.200 | 192.168.10.1           | Ubuntu Server Jakarta |
|   20 | 192.168.20.0/24 | 192.168.20.100 - 192.168.20.200 | 192.168.20.1           | Ubuntu Server Jakarta |

---

## 2.7 FortiGate Jakarta

| Interface   | Terhubung ke            | IP Address     | Keterangan               |
| ----------- | ----------------------- | -------------- | ------------------------ |
| port1       | Cisco Router Jakarta    | 10.10.100.1/30 | Link ke Cisco Jakarta    |
| port2       | MikroTik Router Jakarta | 10.10.101.1/30 | Link ke MikroTik Jakarta |
| port3       | MikroTik ISP            | 10.0.12.2/30   | Link WAN ke ISP          |
| GRE-JKT-SBY | FortiGate Surabaya      | 172.16.0.1/32  | IP GRE Tunnel Jakarta    |

---

# 3. Addressing ISP

## 3.1 MikroTik ISP

| Interface | Terhubung ke         | IP Address            | Keterangan              |
| --------- | -------------------- | --------------------- | ----------------------- |
| ether2    | FortiGate Jakarta    | 10.0.12.1/30          | Link ISP ke Jakarta     |
| ether3    | FortiGate Surabaya   | 10.0.13.1/30          | Link ISP ke Surabaya    |
| ether1    | Cloud NAT / Internet | DHCP / sesuai PNETLab | Akses internet simulasi |

---

## 3.2 Link WAN ISP

| Link           | Network      | Sisi A       | IP Sisi A | Sisi B             | IP Sisi B |
| -------------- | ------------ | ------------ | --------- | ------------------ | --------- |
| Jakarta ↔ ISP  | 10.0.12.0/30 | MikroTik ISP | 10.0.12.1 | FortiGate Jakarta  | 10.0.12.2 |
| ISP ↔ Surabaya | 10.0.13.0/30 | MikroTik ISP | 10.0.13.1 | FortiGate Surabaya | 10.0.13.2 |

---

# 4. Addressing Surabaya / Branch

## 4.1 VLAN Surabaya

| VLAN | Nama VLAN  | Network         | Gateway      | Keterangan                  |
| ---: | ---------- | --------------- | ------------ | --------------------------- |
|   30 | SALES      | 192.168.30.0/24 | 192.168.30.1 | DHCP dari MikroTik Surabaya |
|   40 | OPERATIONS | 192.168.40.0/24 | 192.168.40.1 | IP static manual            |

---

## 4.2 IP Address MikroTik Router Surabaya

| Interface         |                VLAN / Link | IP Address      | Keterangan                                      |
| ----------------- | -------------------------: | --------------- | ----------------------------------------------- |
| vlan30-sales      |                    VLAN 30 | 192.168.30.1/24 | Gateway VLAN 30                                 |
| vlan40-operations |                    VLAN 40 | 192.168.40.1/24 | Gateway VLAN 40                                 |
| ether1            | Link ke FortiGate Surabaya | 10.10.200.2/30  | Transit MikroTik Surabaya ke FortiGate Surabaya |

---

## 4.3 DHCP Pool Surabaya

| VLAN | Network         | Range DHCP                      | Gateway yang Diberikan | DHCP Server            |
| ---: | --------------- | ------------------------------- | ---------------------- | ---------------------- |
|   30 | 192.168.30.0/24 | 192.168.30.100 - 192.168.30.200 | 192.168.30.1           | MikroTik Surabaya      |
|   40 | 192.168.40.0/24 | Static manual                   | 192.168.40.1           | Tidak menggunakan DHCP |

---

## 4.4 IP Client Surabaya

| Client        | VLAN | IP Address       | Gateway      | Keterangan                         |
| ------------- | ---: | ---------------- | ------------ | ---------------------------------- |
| PC Sales      |   30 | DHCP             | 192.168.30.1 | Mendapat IP dari MikroTik Surabaya |
| PC Operations |   40 | 192.168.40.10/24 | 192.168.40.1 | IP static manual                   |
| PC Operations Tinycore linux |   40 | 192.168.40.20/24 | 192.168.40.1 | IP static manual                   |

---

## 4.5 FortiGate Surabaya

| Interface   | Terhubung ke      | IP Address     | Keterangan                         |
| ----------- | ----------------- | -------------- | ---------------------------------- |
| port1       | MikroTik ISP      | 10.0.13.2/30   | Link WAN ke ISP                    |
| port2       | MikroTik Surabaya | 10.10.200.1/30 | Link ke jaringan internal Surabaya |
| GRE-SBY-JKT | FortiGate Jakarta | 172.16.0.2/32  | IP GRE Tunnel Surabaya             |

---

# 5. GRE Tunnel Jakarta–Surabaya

| Tunnel      | Perangkat          | Local WAN | Remote WAN | Tunnel IP     |
| ----------- | ------------------ | --------- | ---------- | ------------- |
| GRE-JKT-SBY | FortiGate Jakarta  | 10.0.12.2 | 10.0.13.2  | 172.16.0.1/32 |
| GRE-SBY-JKT | FortiGate Surabaya | 10.0.13.2 | 10.0.12.2  | 172.16.0.2/32 |

GRE Tunnel digunakan sebagai jalur virtual antara FortiGate Jakarta dan FortiGate Surabaya. OSPF dijalankan di atas GRE Tunnel agar route jaringan Jakarta dan Surabaya dapat saling dipertukarkan secara dinamis.

---

# 6. Network yang Diiklankan melalui OSPF

## 6.1 Network Jakarta

| Network         | Keterangan              |
| --------------- | ----------------------- |
| 192.168.10.0/24 | VLAN 10 Finance Jakarta |
| 192.168.20.0/24 | VLAN 20 IT Jakarta      |
| 192.168.60.0/24 | VLAN Server Jakarta     |
| 172.16.0.1/32   | GRE Tunnel Jakarta      |

---

## 6.2 Network Surabaya

| Network         | Keterangan                  |
| --------------- | --------------------------- |
| 192.168.30.0/24 | VLAN 30 Sales Surabaya      |
| 192.168.40.0/24 | VLAN 40 Operations Surabaya |
| 172.16.0.2/32   | GRE Tunnel Surabaya         |

---


# # Tugas Modul 1 — Konfigurasi Cisco Switch Jakarta

## Perangkat yang Dikonfigurasi

Cisco Switch Jakarta.


## Bukti yang Dikumpulkan

1. Screenshot topologi Jakarta.

2. Screenshot `show vlan brief`.

![](images/tumod/tumod1/1.png)

3. Screenshot `show interfaces trunk`.

![](images/tumod/tumod1/2.png)

---

# Tugas Modul 2 — Konfigurasi Cisco Router Jakarta

## Perangkat yang Dikonfigurasi

Cisco Router Jakarta.

## Bukti yang Dikumpulkan

1. Screenshot `show ip interface brief`.

![alt text](images/tumod/tumod2/1.png)

2. Screenshot `show vrrp brief`.

![alt text](images/tumod/tumod2/2.png)

3. Screenshot konfigurasi subinterface.

4. Screenshot ping dari Cisco Router ke FortiGate Jakarta.
```
CISCO-JAKARTA#ping 10.10.100.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.10.100.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms
CISCO-JAKARTA#

```

---

# Tugas Modul 3 — Konfigurasi MikroTik Router Jakarta

## Perangkat yang Dikonfigurasi

MikroTik Router Jakarta.

## Bukti yang Dikumpulkan

1. Screenshot `/ip address print`.
```
[admin@Mikrotik-Jakarta] > ip address print
Flags: X - disabled, I - invalid, D - dynamic 
 #   ADDRESS            NETWORK         INTERFACE                                     
 0   192.168.10.3/24    192.168.10.0    vlan10-finance                                
 1   192.168.20.3/24    192.168.20.0    vlan20-it                                     
 2   192.168.60.3/24    192.168.60.0    vlan60-ubuntu-server                          
 3   ;;; TO-FORTINET
     10.10.101.2/30     10.10.101.0     ether1                                        
 4   192.168.20.1/32    192.168.20.1    vrrp20                                        
 5   192.168.10.1/32    192.168.10.1    vrrp10                                        
 6   192.168.60.1/32    192.168.60.1    vrrp60                                        
[admin@Mikrotik-Jakarta] > 

```
2. Screenshot `/interface vrrp print`.
```
[admin@Mikrotik-Jakarta] > interface vrrp print
Flags: X - disabled, I - invalid, R - running, M - master, B - backup 
 #     NAME         INTERFACE    MAC-ADDRESS       VRI PRI INTERVAL             V V3..
 0  RM vrrp10       vlan10-fi... 00:00:5E:00:01:0A  10  90 1s                   3 ipv4
 1  RM vrrp20       vlan20-it    00:00:5E:00:01:14  20 120 1s                   3 ipv4
 2  RM vrrp60       vlan60-ub... 00:00:5E:00:01:3C  60  90 1s                   3 ipv4
[admin@Mikrotik-Jakarta] > 

```
3. Screenshot `/ip dhcp-relay print`.
```
[admin@Mikrotik-Jakarta] > ip dhcp-relay print
Flags: X - disabled, I - invalid 
 #   NAME                   INTERFACE                  DHCP-SERVER     LOCAL-ADDRESS  
 0   relay-vlan10           vlan10-finance             192.168.60.10   192.168.10.3   
 1   relay-vlan20           vlan20-it                  192.168.60.10   192.168.20.3   
[admin@Mikrotik-Jakarta] > 

```
4. Screenshot `/ip route print`.
```
[admin@Mikrotik-Jakarta] > ip route print
Flags: X - disabled, A - active, D - dynamic, 
C - connect, S - static, r - rip, b - bgp, o - ospf, m - mme, 
B - blackhole, U - unreachable, P - prohibit 
 #      DST-ADDRESS        PREF-SRC        GATEWAY            DISTANCE
 0 A S  0.0.0.0/0                          10.10.101.1               1
 1 ADC  10.10.101.0/30     10.10.101.2     ether1                    0
 2 ADC  192.168.10.0/24    192.168.10.3    vlan10-finance            0
 3 ADC  192.168.10.1/32    192.168.10.1    vrrp10                    0
 4 ADC  192.168.20.0/24    192.168.20.3    vlan20-it                 0
 5 ADC  192.168.20.1/32    192.168.20.1    vrrp20                    0
 6 ADC  192.168.60.0/24    192.168.60.3    vlan60-ubuntu-s...        0
 7 ADC  192.168.60.1/32    192.168.60.1    vrrp60                    0
[admin@Mikrotik-Jakarta] > 

```
5. Screenshot ping dari MikroTik ke FortiGate Jakarta.
```
[admin@Mikrotik-Jakarta] > ping 10.10.101.1
  SEQ HOST                                     SIZE TTL TIME  STATUS                  
    0 10.10.101.1                                56 255 1ms  
    1 10.10.101.1                                56 255 0ms  
    2 10.10.101.1                                56 255 0ms  
    3 10.10.101.1                                56 255 0ms  
    4 10.10.101.1                                56 255 0ms  
    sent=5 received=5 packet-loss=0% min-rtt=0ms avg-rtt=0ms max-rtt=1ms 

[admin@Mikrotik-Jakarta] > 

```

---

# Tugas Modul 4 — Konfigurasi Ubuntu Server Jakarta

## Perangkat yang Dikonfigurasi

Ubuntu Server Jakarta.

## Bukti yang Dikumpulkan

1. Screenshot `ip a`.
```
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 50:8c:e5:00:ef:00 brd ff:ff:ff:ff:ff:ff
    inet 192.168.60.10/24 brd 192.168.60.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::528c:e5ff:fe00:ef00/64 scope link 
       valid_lft forever preferred_lft forever
root@kvm:/# 

```
2. Screenshot `ip route`.
```
root@kvm:/# ip route
default via 192.168.60.1 dev eth0 proto static 
192.168.60.0/24 dev eth0 proto kernel scope link src 192.168.60.10 
root@kvm:/# 

```
3. Screenshot isi file `/etc/dhcp/dhcpd.conf`.
```
root@kvm:/# sudo cat /etc/dhcp/dhcpd.conf
authoritative;

default-lease-time 600;
max-lease-time 7200;

# DNS
option domain-name-servers 8.8.8.8, 1.1.1.1;

# VLAN 10 - Finance
subnet 192.168.10.0 netmask 255.255.255.0 {
  range 192.168.10.100 192.168.10.200;
  option routers 192.168.10.1;
  option subnet-mask 255.255.255.0;
  option broadcast-address 192.168.10.255;
}

# VLAN 20 - IT
subnet 192.168.20.0 netmask 255.255.255.0 {
  range 192.168.20.100 192.168.20.200;
  option routers 192.168.20.1;
  option subnet-mask 255.255.255.0;
  option broadcast-address 192.168.20.255;
}

# VLAN 60 - Server Network
subnet 192.168.60.0 netmask 255.255.255.0 {
  option routers 192.168.60.1;
  option subnet-mask 255.255.255.0;
  option broadcast-address 192.168.60.255;
}
root@kvm:/# 

```
4. Sreenshot `ping 8.8.8.8`
```
root@kvm:/# ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=3 ttl=109 time=25.7 ms
64 bytes from 8.8.8.8: icmp_seq=4 ttl=109 time=24.0 ms
^C
--- 8.8.8.8 ping statistics ---
10 packets transmitted, 2 received, 80% packet loss, time 9132ms
rtt min/avg/max/mdev = 24.003/24.871/25.740/0.868 ms
root@kvm:/# 
```
---

# Tugas Modul 5 — Konfigurasi FortiGate Jakarta

## Perangkat yang Dikonfigurasi

FortiGate Jakarta.


## Bukti yang Dikumpulkan

1. Screenshot `/ip address print` **hasil di ether 1 bisa saja berbeda karena emang dinamic**.
```
[admin@mikrotik-isp] > ip address print
Flags: X - disabled, I - invalid, D - dynamic 
 #   ADDRESS            NETWORK         INTERFACE                                     
 0 D 10.0.137.149/24    10.0.137.0      ether1                                        
 1   10.0.12.1/30       10.0.12.0       ether2                                        
 2   10.0.13.1/30       10.0.13.0       ether3                                        
[admin@mikrotik-isp] > 

```
2. Screenshot `/ip route print`.
```
[admin@mikrotik-isp] > ip route print
Flags: X - disabled, A - active, D - dynamic, 
C - connect, S - static, r - rip, b - bgp, o - ospf, m - mme, 
B - blackhole, U - unreachable, P - prohibit 
 #      DST-ADDRESS        PREF-SRC        GATEWAY            DISTANCE
 0 ADS  0.0.0.0/0                          10.0.137.1                1
 1 ADC  10.0.12.0/30       10.0.12.1       ether2                    0
 2 ADC  10.0.13.0/30       10.0.13.1       ether3                    0
 3 ADC  10.0.137.0/24      10.0.137.149    ether1                    0
[admin@mikrotik-isp] > 

```
3. Screenshot `/ip firewall nat print`.
```
[admin@mikrotik-isp] > ip firewall nat print
Flags: X - disabled, I - invalid, D - dynamic 
 0    chain=srcnat action=masquerade out-interface=ether1 
[admin@mikrotik-isp] > 

```
4. Screenshot ping ke 8.8.8.8.
5. Screenshot ping antar-WAN FortiGate.

---

# Tugas Modul 7 — Konfigurasi Switch dan MikroTik Surabaya

## Perangkat yang Dikonfigurasi

Cisco Switch Surabaya dan MikroTik Router Surabaya.


## Bukti yang Dikumpulkan

1. Screenshot `show vlan brief`.
```
VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Gi1/0, Gi1/1, Gi1/2, Gi1/3
10   VLAN0010                         active    
20   VLAN0020                         active    
30   sales                            active    Gi0/1
40   operations                       active    Gi0/2, Gi0/3
1002 fddi-default                     act/unsup 
1003 token-ring-default               act/unsup 
1004 fddinet-default                  act/unsup 
1005 trnet-default                    act/unsup 
SWITCH-SURABAYA#

```
2. Screenshot `show interfaces trunk`.
```
SWITCH-SURABAYA#show interfaces tr

Port        Mode             Encapsulation  Status        Native vlan
Gi0/0       on               802.1q         trunking      1

Port        Vlans allowed on trunk
Gi0/0       30,40

Port        Vlans allowed and active in management domain
Gi0/0       30,40

Port        Vlans in spanning tree forwarding state and not pruned
Gi0/0       30,40
SWITCH-SURABAYA#

```
3. Screenshot `/ip address print`.
```
[admin@mikrotik surabaya] > ip address print
Flags: X - disabled, I - invalid, D - dynamic 
 #   ADDRESS            NETWORK         INTERFACE                              
 0   10.10.200.2/30     10.10.200.0     ether1                                 
 1   192.168.30.1/24    192.168.30.0    vlan30-sales                           
 2   192.168.40.1/24    192.168.40.0    vlan40-operations                      
[admin@mikrotik surabaya] > 

```
4. Screenshot `/ip dhcp-server print`.
```
Flags: D - dynamic, X - disabled, I - invalid 
 #    NAME      INTERFACE    RELAY           ADDRESS-POOL    LEASE-TIME ADD-ARP
 0    dhcp1     vlan30-sales                 dhcp_pool0      10m       
[admin@mikrotik surabaya] > 

```
5. Screenshot `/ip pool print`.
```
[admin@mikrotik surabaya] > ip pool print
 # NAME                                         RANGES                         
 0 dhcp_pool0                                   192.168.30.2-192.168.30.254    
[admin@mikrotik surabaya] > 

```
6. Screenshot `/ip route print`.
```
[admin@mikrotik surabaya] > ip route print
Flags: X - disabled, A - active, D - dynamic, 
C - connect, S - static, r - rip, b - bgp, o - ospf, m - mme, 
B - blackhole, U - unreachable, P - prohibit 
 #      DST-ADDRESS        PREF-SRC        GATEWAY            DISTANCE
 0 A S  0.0.0.0/0                          10.10.200.1               1
 1 ADC  10.10.200.0/30     10.10.200.2     ether1                    0
 2 ADC  192.168.30.0/24    192.168.30.1    vlan30-sales              0
 3 ADC  192.168.40.0/24    192.168.40.1    vlan40-operations         0
[admin@mikrotik surabaya] > 

```
7. Screenshot client VLAN 30 mendapat IP DHCP.
```
VPCS> ip dhcp
DORA IP 192.168.30.254/24 GW 192.168.30.1

VPCS> show ip

NAME        : VPCS[1]
IP/MASK     : 192.168.30.254/24
GATEWAY     : 192.168.30.1
DNS         : 8.8.8.8  
DHCP SERVER : 192.168.30.1
DHCP LEASE  : 597, 600/300/525
MAC         : 00:50:79:66:68:e7
LPORT       : 20000
RHOST:PORT  : 127.0.0.1:30000
MTU         : 1500

VPCS> 

```

8. Screenshot ping client Surabaya ke 8.8.8.8.
```
VPCS> ping 8.8.8.8

84 bytes from 8.8.8.8 icmp_seq=1 ttl=109 time=23.941 ms
84 bytes from 8.8.8.8 icmp_seq=2 ttl=109 time=24.313 ms
84 bytes from 8.8.8.8 icmp_seq=3 ttl=109 time=24.125 ms
84 bytes from 8.8.8.8 icmp_seq=4 ttl=109 time=24.357 ms
84 bytes from 8.8.8.8 icmp_seq=5 ttl=109 time=26.724 ms

VPCS> 

```

---

# Tugas Modul 8 — Konfigurasi FortiGate Surabaya

## Perangkat yang Dikonfigurasi

FortiGate Surabaya.


## Hasil yang Diharapkan

1. FortiGate Surabaya dapat ping MikroTik ISP.
2. FortiGate Surabaya dapat ping 8.8.8.8.
3. Client Surabaya dapat akses internet.
4. GRE Tunnel ke Jakarta aktif.
5. OSPF neighbor dengan FortiGate Jakarta berstatus Full.
6. Route Jakarta muncul di routing table FortiGate Surabaya.

## Bukti yang Dikumpulkan

1. Screenshot `get system interface physical`.
2. Screenshot `get router info routing-table all`.
```
Routing table for VRF=0
S*      0.0.0.0/0 [10/0] via 10.0.13.1, port1, [1/0]
C       10.0.13.0/30 is directly connected, port1
C       10.10.200.0/30 is directly connected, port2
C       172.16.0.1/32 is directly connected, GRE-SBY-JKT
C       172.16.0.2/32 is directly connected, GRE-SBY-JKT
O E2    192.168.10.0/24 [110/10] via 172.16.0.1, GRE-SBY-JKT, 02:27:36, [1/0]
O E2    192.168.20.0/24 [110/10] via 172.16.0.1, GRE-SBY-JKT, 02:27:36, [1/0]
S       192.168.30.0/24 [10/0] via 10.10.200.2, port2, [1/0]
S       192.168.40.0/24 [10/0] via 10.10.200.2, port2, [1/0]
O E2    192.168.60.0/24 [110/10] via 172.16.0.1, GRE-SBY-JKT, 02:27:36, [1/0]


Fortinet-Surabaya # 
```
3. Screenshot firewall policy.
4. Screenshot ping ke 8.8.8.8.
5. Screenshot ping ke IP tunnel Jakarta.
```
Fortinet-Surabaya # execute ping 172.16.0.1
PING 172.16.0.1 (172.16.0.1): 56 data bytes
64 bytes from 172.16.0.1: icmp_seq=0 ttl=255 time=1.1 ms
64 bytes from 172.16.0.1: icmp_seq=1 ttl=255 time=1.0 ms
64 bytes from 172.16.0.1: icmp_seq=2 ttl=255 time=0.6 ms
64 bytes from 172.16.0.1: icmp_seq=3 ttl=255 time=0.6 ms
64 bytes from 172.16.0.1: icmp_seq=4 ttl=255 time=0.7 ms

--- 172.16.0.1 ping statistics ---
5 packets transmitted, 5 packets received, 0% packet loss
round-trip min/avg/max = 0.6/0.8/1.1 ms

Fortinet-Surabaya # 

```
6. Screenshot `get router info ospf neighbor`.
```
Fortinet-Surabaya # get router info ospf neighbor 
OSPF process 0, VRF 0:
Neighbor ID     Pri   State           Dead Time   Address         Interface
1.1.1.1           1   Full/ -         00:00:38    172.16.0.1      GRE-SBY-JKT



Fortinet-Surabaya # 


```
7. Screenshot `get router info routing-table ospf`.
```
Fortinet-Surabaya # get router info routing-table ospf
Routing table for VRF=0
O E2    192.168.10.0/24 [110/10] via 172.16.0.1, GRE-SBY-JKT, 02:29:23, [1/0]
O E2    192.168.20.0/24 [110/10] via 172.16.0.1, GRE-SBY-JKT, 02:29:23, [1/0]
O E2    192.168.60.0/24 [110/10] via 172.16.0.1, GRE-SBY-JKT, 02:29:23, [1/0]


Fortinet-Surabaya # 
```

---

# Tugas Modul 9 — Konfigurasi GRE Tunnel dan OSPF over GRE

## Perangkat yang Dikonfigurasi

FortiGate Jakarta dan FortiGate Surabaya.


## Bukti yang Dikumpulkan

1. Screenshot ping WAN antar-FortiGate.
2. Screenshot ping tunnel antar-FortiGate.
```
Fortinet-Surabaya # execute ping 172.16.0.1
PING 172.16.0.1 (172.16.0.1): 56 data bytes
64 bytes from 172.16.0.1: icmp_seq=0 ttl=255 time=0.7 ms
64 bytes from 172.16.0.1: icmp_seq=1 ttl=255 time=0.6 ms
64 bytes from 172.16.0.1: icmp_seq=2 ttl=255 time=0.7 ms
^C
--- 172.16.0.1 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 0.6/0.6/0.7 ms

Fortinet-Surabaya # 

```
3. Screenshot `get router info ospf neighbor`.
4. Screenshot `get router info routing-table ospf`.
5. Screenshot ping client Jakarta ke client Surabaya.
6. Screenshot ping client Surabaya ke client Jakarta.

---

# Tugas Modul 10 — Pengujian Akhir

## Perangkat yang Diuji

Seluruh perangkat pada topologi.

## Hal yang Harus Diuji

1. Client VLAN 10 Jakarta mendapat IP DHCP dari Ubuntu Server.
2. Client VLAN 20 Jakarta mendapat IP DHCP dari Ubuntu Server.
3. Client VLAN 30 Surabaya mendapat IP DHCP dari MikroTik Surabaya.
4. Client VLAN 40 Surabaya menggunakan IP static.
5. Client Jakarta dapat ping 8.8.8.8.
6. Client Surabaya dapat ping 8.8.8.8.
7. Client Jakarta dapat ping client Surabaya.
8. Client Surabaya dapat ping client Jakarta.
9. Client Surabaya dapat mengakses web server Jakarta.


---

# 4. Hasil Pengujian

1. Screenshot IP DHCP client Jakarta (Vlan 10).

2. Screenshot IP DHCP client Surabaya (Vlan 20).


3. Screenshot ping internet dari Jakarta.



4. Screenshot ping internet dari Surabaya.

5. Screenshot ping antar-site (ini aku tes dari vlan 10 ke 
6. Screenshot akses web server Jakarta dari Surabaya.


7. Screenshot routing table OSPF.
8. Analisis singkat jalur traffic Jakarta ke Surabaya.

---

# 5. Analisis

Pada praktikum ini FortiGate berfungsi sebagai firewall utama yang menghubungkan jaringan WAN, LAN, dan DMZ. Routing statis yang dikonfigurasi memungkinkan komunikasi antarsegmen jaringan, sedangkan firewall policy mengatur akses yang diperbolehkan dan ditolak.

Implementasi DMZ memungkinkan server web dapat diakses dari jaringan luar tanpa membuka akses langsung ke jaringan LAN. Hal ini meningkatkan keamanan karena client WAN hanya dapat mengakses layanan yang dipublikasikan melalui Virtual IP (VIP) pada FortiGate.

Hasil pengujian menunjukkan bahwa seluruh konfigurasi berjalan sesuai dengan desain topologi. Client LAN dapat mengakses server DMZ secara langsung, sedangkan Client WAN hanya dapat mengakses web server melalui alamat publik yang diterjemahkan menggunakan mekanisme NAT dan port forwarding.

---

# 6. Kesimpulan

1. MikroTik ISP berhasil berfungsi sebagai gateway menuju jaringan luar.
2. FortiGate berhasil dikonfigurasi sebagai firewall yang menghubungkan WAN, LAN, dan DMZ.
3. Cisco Router berhasil menghubungkan jaringan internal LAN dengan FortiGate.
4. Server DMZ berhasil diakses dari Client LAN maupun Client WAN melalui konfigurasi VIP dan firewall policy.
5. Firewall berhasil membatasi akses langsung dari jaringan WAN menuju jaringan LAN dan IP asli server DMZ.
6. Implementasi NAT, routing, firewall policy, dan DMZ berjalan sesuai dengan tujuan praktikum.
