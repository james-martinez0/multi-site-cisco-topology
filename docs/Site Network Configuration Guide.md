# Site Network Configuration Guide
___

# Setup Network Topology

## Create the Cisco Three Layer Model

1. Setup main router
2. Setup Core Switch - 3650 layer 3 switch
3. Setup Distribution Switch - 3650 layer 3 switch
4. Setup Access Switch for Each Department - 2960 layer 2 switch
    1. Connect Access Switch of Each Department to Distribution Switch (using gigabit port in 2960 switch)
    2. Add PC for each department
5. Setup Wireless Access Point for Wireless Devices (AccessPoint-PT-N)
    1. Setup SSID and Password for each Wireless Access Point
    2. Add Wireless Device for each department (Laptop)

## Setup Server End Devices

1. Place a switch for server and connect it at the last ethernet port of distribution switch using crossover cable
2. Setup Servers
    - SMTP
    - DHCP
    - DNS
    - Web Server

# Initial Setup for all switches

1. Disable ip domain lookup for all switches
    
    ```bash
    no ip domain-lookup
    ```
    
2. Enable cdp neighbors
    
    ```bash
    cdp run
    ```
    
3. Changed hostname (MD1 for Manila Distribution 1)
    
    ```bash
    en
    config t
    hostname MD1
    write memory
    ```
    
 Hostname off all switches

| Name                    | Hostname     |
| ----------------------- | ------------ |
| ManilaEngineering       | MLA_ENG      |
| ManilaSalesMarketing    | MLA_SALESMAR |
| ManilaServiceDesk       | MLA_SERVDESK |
| ManilaAccounting        | MLA_ACCOUNT  |
| ManilaTravelAndExpenses | MLA_TRAVEXP  |
| ManialHRDept            | MLA_HRDEPT   |
| ManilaProjectManagement | MLA_PROJMAN  |
| ManilaServers           | MLA_SERVERS  |
| ManilaDistribution1     | MLA_DIST1    |
| ManilaDistribution2     | MLA_DIST2    |
| ManilaCore1             | MLA_CORE1    |
| ManilaCore2             | MLA_CORE2    |
| ManilaRouter            | MLA_ROUTER   |

# Setup Server Static IP Address

1. Open SMTP server, then go config
2. Based on the VLSM table, get the first assignable IP address. (172.17.0.1)
3. Click on FastEthernet0, then input the server assigned IP address based on IP Address table (172.17.0.2).
4. Input also the subnet mask.
5. Repeat for all servers.

# VLAN Configuration

## Make changes on topology

1. Add 2 power supplies on each distribution and core switch to power them on.
    1. Just drag and drop the power supply image on the bottom right.
2. Adjust interfaces connection based the VLSM table if needed.
    1. There are changes between the first vid, so make sure you change the wiring.

## Creating and assigning VLANs (within VLAN connection)

<aside>
💡  The written code here is for Manila. Make sure to follow the VLSM table for a specific site

</aside>

### Setup on VLANs on Access Layer (Users)

1. Open ManilaEngineering access switch, then go to CLI
2. Assign all ports to VLAN 11
    
    ```bash
    en
    config t
    int range fa0/1-24
    switchport access vlan 11
    exit
    ```
    
3. Enable trunk
    
    ```bash
    en
    config t
    int range gig0/1-2
    switchport mode trunk
    do write memory
    ```
    
4. Repeat all step 1 and 2 for access layer switch while changing the vlan number
    - Complete code
        
        ```bash
        en
        config t
        int range fa0/1 - 24
        switchport access vlan 11
        exit
        int range gig0/1-2
        switchport mode trunk
        
        en
        config t
        int range fa0/1 - 24
        switchport access vlan 12
        exit
        int range gig0/1-2
        switchport mode trunk
        
        en
        config t
        int range fa0/1 - 24
        switchport access vlan 13
        exit
        int range gig0/1-2
        switchport mode trunk
        
        en
        config t
        int range fa0/1 - 24
        switchport access vlan 14
        exit
        int range gig0/1-2
        switchport mode trunk
        
        en
        config t
        int range fa0/1 - 24
        switchport access vlan 15
        exit
        int range gig0/1-2
        switchport mode trunk
        
        en
        config t
        int range fa0/1 - 24
        switchport access vlan 16
        exit
        int range gig0/1-2
        switchport mode trunk
        
        en
        config t
        int range fa0/1 - 24
        switchport access vlan 17
        exit
        int range gig0/1-2
        switchport mode trunk
        ```
        

### Setup on VLANs on Access Layer (Server)

1. Open Server access switch, then go to CLI
2. Assign all ports to VLAN 10
    
    ```bash
    en
    config t
    int range fa0/1 - 24
    switchport access vlan 30
    exit
    int range gig0/1-2
    switchport mode trunk
    write memory
    ```
    

### Setup on VLANs on Distribution Layer

1. Open ManilaDistribution1, then go to CLI
2. Press enter if needed, if ask to do initial configuration, type no then press enter
3. Create VLANs (VLAN 10 is MANILA Sever, VLAN 11-17 MANILA USERS)
    
    ```bash
    en
    config t
    vlan 30
    vlan 31
    vlan 32
    vlan 33
    vlan 34
    vlan 35
    vlan 36
    vlan 37
    ```
    

### Test connection within VLAN

1. Place 2 new test PC
2. Assign a temporary static ip address from assignable range on both PC (ex. on VLAN 10 - 192.168.0.2)
3. Connect both PC on ManilaEngineering access switch
4. Test ping each other

# Setup inter-VLAN connection and routing

## For Users VLANs

### On Distribution 1

1. Create virtual interface
    
    ```bash
    en
    config t
    int vlan 11
    ```
    
2. Set IP Address of vlan 11 with the last assignable range IP in VLSM table
    
    ```bash
    ip add 192.168.0.254 255.255.254.0
    ```
    
3. Set gateway address
    
    ```bash
    standby 11 ip 192.168.0.1
    ```
    
4. Set this switch a priority for VLAN 11 using HSRP
    
    ```bash
    standby 11 priority 105
    standby 11 preempt
    exit
    write memory
    ```
    
5. Repeat All the steps for all users VLANs
6. Enable IP Routing
    
    ```bash
    ip routing
    ```
    

- Complete commands for Distribution1
    
    ```bash
    en
    config t
    int vlan 31
    ip add 192.168.6.254 255.255.255.0
    standby 31 ip 192.168.6.1
    standby 31 priority 105
    standby 31 preempt
    exit
    
    int vlan 32
    ip add 192.168.7.254 255.255.255.0
    standby 32 ip 192.168.7.1
    standby 32 priority 105
    standby 32 preempt
    exit
    
    int vlan 33
    ip add 192.168.8.30 255.255.255.192
    standby 33 ip 192.168.8.1
    standby 33 priority 105
    standby 33 preempt
    exit
    
    int vlan 34
    ip add 192.168.8.62 255.255.255.192
    standby 34 ip 192.168.8.33
    standby 34 priority 105
    standby 34 preempt
    exit
    
    int vlan 15
    ip add 192.168.3.158 255.255.255.224
    standby 15 ip 192.168.3.129
    standby 15 priority 105
    standby 15 preempt
    exit
    
    int vlan 16
    ip add 192.168.3.190 255.255.255.224
    standby 16 ip 192.168.3.161
    standby 16 priority 105
    standby 16 preempt
    exit
    
    int vlan 17
    ip add 192.168.3.206 255.255.255.240
    standby 17 ip 192.168.3.193
    standby 17 priority 105
    standby 17 preempt
    exit
    ip routing
    write memory
    
    ```
    

### On Distribution 2

1. Repeat all steps on Distribution1 but change IP address and priority
    1. Change IP VLAN interface IP address by subtracting 1 on its IP
        
        ```bash
        ip add 192.168.0.253 255.255.254.0
        ```
        
    
    b. Change priority to 95
    
    ```bash
    standby 11 priority 95
    standby 11 preempt
    exit
    write memory
    ```
    
- Complete commands for Distribution2
    
    ```bash
    en
    config t
    int vlan 11
    ip add 192.168.0.253 255.255.254.0
    standby 11 ip 192.168.0.1
    standby 11 priority 105
    standby 11 preempt
    exit
    
    int vlan 12
    ip add 192.168.2.253 255.255.255.0
    standby 12 ip 192.168.2.1
    standby 12 priority 105
    standby 12 preempt
    exit
    
    int vlan 13
    ip add 192.168.3.61 255.255.255.192
    standby 13 ip 192.168.3.1
    standby 13 priority 95
    standby 13 preempt
    exit
    
    int vlan 14
    ip add 192.168.3.125 255.255.255.192
    standby 14 ip 192.168.3.65
    standby 14 priority 95
    standby 14 preempt
    exit
    
    int vlan 15
    ip add 192.168.3.157 255.255.255.224
    standby 15 ip 192.168.3.129
    standby 15 priority 95
    standby 15 preempt
    exit
    
    int vlan 16
    ip add 192.168.3.189 255.255.255.224
    standby 16 ip 192.168.3.161
    standby 16 priority 95
    standby 16 preempt
    exit
    
    int vlan 17
    ip add 192.168.3.205 255.255.255.240
    standby 17 ip 192.168.3.193
    standby 17 priority 95
    standby 17 preempt
    exit
    ip routing
    write memory
    ```
    

### Testing Inter-VLAN connection

1. Create another temporary PC connected to other switch with static IP Address
2. Ping each other

## For Servers VLANs

### On Distribution 1

1. Create virtual interface
    
    ```bash
    en
    config t
    int vlan 10
    ```
    
2. Set IP Address of vlan 10 with the last assignable range IP in VLSM table
    
    ```bash
    ip add 172.17.0.14 255.255.255.240
    ```
    
3. Set gateway address
    
    ```bash
    standby 30 ip 172.17.0.33
    ```
    
4. Set this switch as standby for VLAN 10 using HSRP
    
    ```bash
    standby 30 priority 95
    standby 30 preempt
    exit
    write memory
    ```
    
- Complete Commands for Distribution 1
    
    ```bash
    en
    config t
    int vlan 30
    ip add 172.17.0.45 255.255.255.240
    standby 30 ip 172.17.0.33
    standby 30 priority 105
    standby 30 preempt
    exit
    write memory
    ```
    

### On Distribution 2

1. Repeat all steps on Distribution1 but change IP address and priority
    1. Change IP VLAN interface IP address by subtracting 1 on its IP
        
        ```bash
        ip add 172.17.0.45 255.255.240.0
        ```
        
    
    b. Change priority to 105
    
    ```bash
    standby 30 ip 172.17.0.33
    standby 30 priority 105
    standby 30 preempt
    exit
    write memory
    ```
    
- Complete commands for Distribution2
    
    ```bash
    en
    config t
    int vlan 10
    ip add 172.17.0.13 255.255.255.240
    standby 10 ip 172.17.0.1
    standby 10 priority 105
    standby 10 preempt
    exit
    write memory
    ```
    

### Testing

1. Ping the server from a PC

# DHCP Setup

## DHCP Server Pool Setup

1. Turn on 2 Distribution Switches (if not turned on)
    1. Add power supply
2. Rename DHCP server to Manila DHCP
3. On Services, turn off HTTP and HTTPS
4. Select DHCP, then turn it on
5. On Pool Name, input vlan number based on the VLSM table
6. Input on Default Gateway the first IP in the assignable range column
7. Input 172.17.0.4 (This is for manila only) on DNS Server (Changed this based on the IP Address of DNS server on the site)
8. Input start IP address based assignable range column on USERS VLAN, add 1 to it, (first IP is assigned for gateway)
9. Input Subnet Mask from the subnet mask column
10. Input Maximum Number of Users based on allocated column
11. Repeat it for all VLANs

## Distribution Switch DHCP Helper

1. Setup DHCP on both Distribution 1 and 2 (172.17.0.3 is the IP address of DHCP on the site)
    
    ```bash
    en
    config t
    int range vlan 31-34
    ip helper-address 172.17.0.34
    write memory
    ```
    
2. Set all user end devices to DHCP
    1. if already on DHCP, set it on Static then set to DHCP

# Core IP Configuration

<aside>
💡 Replaced the IP Address of the configuration based on the LINKS section of the VLSM

</aside>

## Manila Distribution 1 Configuration

1. Use show cdp neighbor to know which interface is connected to which.
    
    ```bash
    show cdp neighbor
    ```
    
    <aside>
    💡 No “no switchport” means only layer 3, layer 2 is routed from here.
    
    </aside>
    
2. To Manila Core 1 (combine two interface and set IP Address)
    
    ```bash
    en
    config t
    int range gig1/0/1-2
    no switchport
    channel-group 1 mode desirable
    
    int po1
    ip add 172.17.100.33 255.255.255.252
    no shut
    do write mem
    ```
    
3. To Manila Core 2 (set ip address
    
    ```bash
    en
    config t
    int gig1/0/3
    no switchport
    ip add 172.17.100.41 255.255.255.252
    no shut
    do write mem
    ```
    

## Manila Distribution 2 Configuration

1. Use show cdp neighbor
2. To Manila Core 2 (combine two interface and set IP Address)
    
    ```bash
    en
    config t
    int range gig1/0/1-2
    no switchport
    channel-group 1 mode desirable
    
    int po1
    ip add 172.17.100.45 255.255.255.252
    no shut
    do write mem
    ```
    
3. To Manila Core 1 (set IP address)
    
    ```bash
    en
    config t
    int gig1/0/3
    no switchport
    ip add 172.17.100.37 255.255.255.252
    no shut
    do write mem
    ```
    

## Manila Core 1 Configuration

1. To Manila Distribution 1 (combine two interface and set IP Address)
    
    ```bash
    en
    config t
    int range gig1/0/2-3
    no switchport
    channel-group 1 mode desirable
    
    int po1
    ip add 172.17.100.34 255.255.255.252
    no shut
    do write mem
    ```
    
2. To Manila Distribution 2 (set IP Address)
    
    ```bash
    en
    config t
    int gig1/0/4
    no switchport
    ip add 172.17.100.38 255.255.255.252
    no shut
    do write mem
    ```
    
3. To Manila Core 2 (set IP Address)
    
    ```bash
    en
    config t
    int gig1/0/5
    no switchport
    ip add 172.17.100.57 255.255.255.252
    no shut
    do write mem
    ```
    
4. To Manila Router (set IP Address)
    
    ```bash
    en
    config t
    int gig1/0/1
    no switchport
    ip add 172.17.100.49 255.255.255.252
    no shut
    do write mem
    ```
    

## Manila Core 2

1. To Manila Distribution 2 (combine two interface and set IP Address)
    
    ```bash
    en
    config t
    int range gig1/0/2-3
    channel-group 1 mode desirable
    
    int po1
    no switchport
    ip add 172.17.100.46 255.255.255.252
    no shut
    do write mem
    ```
    
2. To Manila Distribution 1 (set IP Address)
    
    ```bash
    en
    config t
    int gig1/0/4
    no switchport
    ip add 172.17.100.42 255.255.255.252
    no shut
    do write mem
    ```
    
3. To Manila Core 1 (set IP Address)
    
    ```bash
    en
    config t
    int gig1/0/5
    no switchport
    ip add 172.17.100.58 255.255.255.252
    no shut
    do write mem
    ```
    
4. To Manila Router (set IP Address)
    
    ```bash
    en
    config t
    int gig1/0/1
    no switchport
    ip add 172.17.100.53 255.255.255.252
    no shut
    do write mem
    ```
    

## Manila Router

1. To Manila Core  1 (set IP address)
    
    ```bash
    en
    config t
    int gig0/0
    ip add 172.17.100.50 255.255.255.252
    no shut
    do write mem
    ```
    
2. To Manila Core 2 (set IP address)
    
    ```bash
    en
    config t
    int gig0/1
    ip add 172.17.100.54 255.255.255.252
    no shut
    do write mem
    ```
    
3. To Internet
    
    ```bash
    
    ```
    

## Cebu Router

# OSPF Configuration

1. Add configuration to all routers
    
    ```bash
    en
    config t
    
    ip routing
    router ospf 1
    network 172.17.0.0 0.0.255.255 area 0
    network 192.168.0.0 0.0.255.255 area 0
    network 62.62.62.0 0.0.255.255 area 0
    ```
    

# NAT Configuration with PAT

## Internet Router Config

```bash
en
config t

ip routing
no router ospf 1
router ospf 1
network 172.17.0.0 0.0.255.255 area 0
network 192.168.0.0 0.0.255.255 area 0
network 62.62.62.0 0.0.0.255 area 0
```

## Manila Router

```bash
access-list 120 permit ip 192.168.0.0 0.0.127.255 142.251.0.0 0.0.255.255
ip nat pool manila 62.62.62.1 62.62.62.30 netmask 255.255.255.224
ip nat inside source list 120 pool manila overload

int gig0/0
ip nat inside
exit

int gig0/1
ip nat inside
exit

int se0/3/0
ip nat outside
exit

ip route 62.62.62.0 255.255.255.224 null 0
router ospf 1
redistribute static subnets
write mem
```

- OLD Config for reference onl
    
    ```bash
    en
    config t
    access-list 110 permit ip 192.168.0.0 0.0.127.255 192.168.4.0 0.0.1.255
    access-list 110 permit ip 172.17.0.0 0.0.0.127 any
    access-list 110 deny ip any any
    
    ip nat pool manila 62.62.62.1 62.62.62.30 netmask 255.255.255.224
    ip nat inside source list 110 pool manila overload
    
    int gig0/0
    ip nat inside
    exit
    
    int gig0/1
    ip nat inside
    exit
    
    int se0/3/0
    ip nat outside
    exit
    
    ip route 62.62.62.0 255.255.255.224 null 0
    router ospf 1
    redistribute static subnets
    write mem
    ```
    

## Cebu Router

```bash
en
conf
access-list 120 permit ip 192.168.4.0 0.0.1.255 142.251.0.0 0.0.255.255
access-list 120 deny ip any any

ip nat pool cebu 62.62.62.33 62.62.62.62 netmask 255.255.255.224
ip nat inside source list 120 pool cebu overload

int gig0/0
ip nat inside
exit

int gig0/1
ip nat inside
exit

int se0/3/0
ip nat outside
exit

ip route 62.62.62.0 255.255.255.224 null 0
router ospf 1
redistribute static subnets
write mem
```

## Bohol

```bash
en
conf
access-list 120 permit ip 192.168.9.0 0.0.0.255 142.251.0.0 0.0.255.255
access-list 120 deny ip any any

ip nat pool bohol 62.62.62.97 62.62.62.126 netmask 255.255.255.224
ip nat inside source list 120 pool bohol overload

int gig0/0
ip nat inside
exit

int gig0/1
ip nat inside
exit

int se0/3/0
ip nat outside
exit

ip route 62.62.62.0 255.255.255.224 null 0
router ospf 1
redistribute static subnets
write mem
```

## Testing and Verifiying

```bash
show ip nat translation
```

# VPN

1. To check version of the security 

```bash
show version
```

## Manila Router
```bash
en
config t
license boot module c2900 technology-package securityk9
do write mem
do reload
do show version

crypto isakmp policy 1
encryption aes 256
authentication pre-share
lifetime 86400
group 5
exit

crypto isakmp policy 2
encryption aes 256
authentication pre-share
lifetime 86400
group 5
exit

crypto isakmp policy 3
encryption aes 256
authentication pre-share
lifetime 86400
group 5
exit

crypto isakmp key 1a2b3c4d address 62.62.62.33
crypto isakmp key 1a2b3c4d address 62.62.62.65
crypto isakmp key 1a2b3c4d address 62.62.62.97

crypto ipsec transform-set Cebu-Transform-Set esp-aes esp-sha-hmac
crypto ipsec transform-set Makati-Transform-Set esp-aes esp-sha-hmac
crypto ipsec transform-set Bohol-Transform-Set esp-aes esp-sha-hmac

access-list 110 permit ip 192.168.0.0 0.0.2.255 192.168.4.0 0.0.1.255
access-list 111 permit ip 192.168.0.0 0.0.2.255 192.168.6.0 0.0.2.255
access-list 112 permit ip 192.168.0.0 0.0.2.255 192.168.9.0 0.0.0.255

crypto map Cebu-VPN-MAP 1 ipsec-isakmp
set peer 62.62.62.33
set transform-set Cebu-Transform-Set
match address 110
exit

crypto map Makati-VPN-MAP 2 ipsec-isakmp
set peer 62.62.62.65
set transform-set Makati-Transform-Set
match address 111
exit

crypto map Bohol-VPN-MAP 3 ipsec-isakmp
set peer 62.62.62.97
set transform-set Bohol-Transform-Set
match address 112
exit

int se0/3/0
crypto map Cebu-VPN-MAP
crypto map Makati-VPN-MAP
crypto map Bohol-VPN-MAP
write mem
```

## Cebu Router

```bash
en
config t
license boot module c2900 technology-package securityk9
write mem
reload
show version

crypto isakmp policy 1
encryption aes 256
authentication pre-share
lifetime 86400
group 5
exit

crypto isakmp policy 2
encryption aes 256
authentication pre-share
lifetime 86400
group 5
exit

crypto isakmp policy 3
encryption aes 256
authentication pre-share
lifetime 86400
group 5
exit

crypto isakmp key 1a2b3c4d address 62.62.62.1
crypto isakmp key 1a2b3c4d address 62.62.62.65
crypto isakmp key 1a2b3c4d address 62.62.62.97

crypto ipsec transform-set Manila-Transform-Set esp-aes esp-sha-hmac
crypto ipsec transform-set Makati-Transform-Set esp-aes esp-sha-hmac
crypto ipsec transform-set Bohol-Transform-Set esp-aes esp-sha-hmac

access-list 110 permit ip 192.168.4.0 0.0.1.255 192.168.0.0 0.0.2.255
access-list 111 permit ip 192.168.4.0 0.0.1.255 192.168.6.0 0.0.2.255
access-list 112 permit ip 192.168.4.0 0.0.1.255 192.168.9.0 0.0.0.255

crypto map Manila-VPN-MAP 1 ipsec-isakmp
set peer 62.62.62.1
set transform-set Manila-Transform-Set
match address 110
exit

crypto map Makati-VPN-MAP 2 ipsec-isakmp
set peer 62.62.62.65
set transform-set Makati-Transform-Set
match address 111
exit

crypto map Bohol-VPN-MAP 3 ipsec-isakmp
set peer 62.62.62.97
set transform-set Bohol-Transform-Set
match address 112
exit

int se0/3/0
crypto map Manila-VPN-MAP
crypto map Makati-VPN-MAP
crypto map Bohol-VPN-MAP
write mem
```

## Makati Router

```bash
en
config t
license boot module c2900 technology-package securityk9
write mem
reload
show version

crypto isakmp policy 1
encryption aes 256
authentication pre-share
lifetime 86400
group 5
exit

crypto isakmp policy 2
encryption aes 256
authentication pre-share
lifetime 86400
group 5
exit

crypto isakmp policy 3
encryption aes 256
authentication pre-share
lifetime 86400
group 5
exit

crypto isakmp key 1a2b3c4d address 62.62.62.1
crypto isakmp key 1a2b3c4d address 62.62.62.33
crypto isakmp key 1a2b3c4d address 62.62.62.97

crypto ipsec transform-set Manila-Transform-Set esp-aes esp-sha-hmac
crypto ipsec transform-set Cebu-Transform-Set esp-aes esp-sha-hmac
crypto ipsec transform-set Bohol-Transform-Set esp-aes esp-sha-hmac

access-list 110 permit ip 192.168.6.0 0.0.2.255 192.168.0.0 0.0.2.255
access-list 111 permit ip 192.168.6.0 0.0.2.255 192.168.4.0 0.0.1.255
access-list 112 permit ip 192.168.6.0 0.0.2.255 192.168.9.0 0.0.0.255

crypto map Manila-VPN-MAP 1 ipsec-isakmp
set peer 62.62.62.1
set transform-set Manila-Transform-Set
match address 110
exit

crypto map Cebu-VPN-MAP 2 ipsec-isakmp
set peer 62.62.62.33
set transform-set Cebu-Transform-Set
match address 111
exit

crypto map Bohol-VPN-MAP 3 ipsec-isakmp
set peer 62.62.62.97
set transform-set Bohol-Transform-Set
match address 112
exit

int se0/3/0
crypto map Manila-VPN-MAP
crypto map Cebu-VPN-MAP
crypto map Bohol-VPN-MAP
write mem
```

## Bohol Router

```bash
en
config t
license boot module c2900 technology-package securityk9
write mem
reload
show version

crypto isakmp policy 1
encryption aes 256
authentication pre-share
lifetime 86400
group 5
exit

crypto isakmp policy 2
encryption aes 256
authentication pre-share
lifetime 86400
group 5
exit

crypto isakmp policy 3
encryption aes 256
authentication pre-share
lifetime 86400
group 5
exit

crypto isakmp key 1a2b3c4d address 62.62.62.1
crypto isakmp key 1a2b3c4d address 62.62.62.33
crypto isakmp key 1a2b3c4d address 62.62.62.65

crypto ipsec transform-set Manila-Transform-Set esp-aes esp-sha-hmac
crypto ipsec transform-set Cebu-Transform-Set esp-aes esp-sha-hmac
crypto ipsec transform-set Makati-Transform-Set esp-aes esp-sha-hmac

access-list 110 permit ip 192.168.9.0 0.0.0.255 192.168.0.0 0.0.2.255
access-list 111 permit ip 192.168.9.0 0.0.0.255 192.168.4.0 0.0.1.255
access-list 112 permit ip 192.168.9.0 0.0.0.255 192.168.6.0 0.0.2.25

crypto map Manila-VPN-MAP 1 ipsec-isakmp
set peer 62.62.62.1
set transform-set Manila-Transform-Set
match address 110
exit

crypto map Cebu-VPN-MAP 2 ipsec-isakmp
set peer 62.62.62.33
set transform-set Cebu-Transform-Set
match address 111
exit

crypto map Makati-VPN-MAP 3 ipsec-isakmp
set peer 62.62.62.65
set transform-set Makati-Transform-Set
match address 112
exit

int se0/3/0
crypto map Manila-VPN-MAP
crypto map Cebu-VPN-MAP
crypto map Makati-VPN-MAP
write mem
```


# Configure Security

```bash
en
config t
enable password M@nila2024$$
enable secret M@nila2024&&
service password-encryption

en
config t
enable password C3bu2024$$
enable secret C3bu2024&&
service password-encryption

en
config t
enable password M@kati2024$$
enable secret M@kati2024&&
service password-encryption

en
config t
enable password B0hol2024$$
enable secret B0hol2024&&
service password-encryption

```

### Working

1. VLAN (with Intervlan Routing)
2. DHCP
3. HSRP
4. OSPF
5. Etherchannel
6. Local Web Server
7. Local DNS
8. NAT (for internet server only)
9. Site to Site VPN (by pair only - Manila and Bohol, Makati and Cebu)

### Passwords for all routers and switches per site

M@nila2024&&

C3bu2024&&

M@kati2024&&

B0hol2024&&