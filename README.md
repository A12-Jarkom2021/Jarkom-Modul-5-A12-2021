# Jarkom-Modul-5-A12-2021
Jarkom_Modul5_Lapres_A12

**Praktikum Modul 5 - Jaringan Komputer 2021**
**A12**
-   Fiqey Indriati Eka Sari (05111940000015)
-   Dyah Putri Nariswari (05111940000047)
-   Muhammad Farrel Abhinaya (05111940000173)
---

### A
Tugas pertama kalian yaitu membuat topologi jaringan sesuai dengan rancangan yang diberikan Luffy dibawah ini:

![image](https://user-images.githubusercontent.com/57583780/145675825-650a4a39-8423-48fc-9e46-b9d83bd9a967.png)

Keterangan : 	
        Doriki adalah DNS Server
		Jipangu adalah DHCP Server
		Maingate dan Jorge adalah Web Server
		Jumlah Host pada Blueno adalah 100 host
		Jumlah Host pada Cipher adalah 700 host
		Jumlah Host pada Elena adalah 300 host
		Jumlah Host pada Fukurou adalah 200 host

### B
Karena kalian telah belajar subnetting dan routing, Luffy ingin meminta kalian untuk membuat topologi tersebut menggunakan teknik CIDR atau VLSM. setelah melakukan subnetting, 

#### Subnetting menggunakan VLSM

![image](https://user-images.githubusercontent.com/57583780/145675893-f3938a87-5bce-47e4-b9ee-303f0de54393.png)


### C
Kalian juga diharuskan melakukan Routing agar setiap perangkat pada jaringan tersebut dapat terhubung.

#### Routing 

![image](https://user-images.githubusercontent.com/57583780/145675902-3c7b1145-37cd-419a-8839-d8726b6fc8c1.png)

##### Routing Foosha
```
route add -net 10.5.0.8 netmask 255.255.255.248 gw 10.5.0.2
route add -net 10.5.0.128 netmask 255.255.255.128 gw 10.5.0.2
route add -net 10.5.4.0 netmask 255.255.252.0 gw 10.5.0.2
route add -net 10.5.2.0 netmask 255.255.254.0 gw 10.5.0.6
route add -net 10.5.1.0 netmask 255.255.255.0 gw 10.5.0.6
route add -net 10.5.0.16 netmask 255.255.255.248 gw 10.5.0.6
```

##### Routing Water7
```
route add -net 0.0.0.0 netmask 0.0.0.0 gw 10.5.0.1
```

##### Routing Guanhao
```
route add -net 0.0.0.0 netmask 0.0.0.0 gw 10.5.0.5
```

### D
Tugas berikutnya adalah memberikan ip pada subnet Blueno, Cipher, Fukurou, dan Elena secara dinamis menggunakan bantuan DHCP server. Kemudian kalian ingat bahwa kalian harus setting DHCP Relay pada router yang menghubungkannya.

## Setup DHCP Relay di Foosha, Guanhao, dan Water7
- Install `isc-dhcp-server`
```
#install DHCP Relay
apt-get update
apt-get install isc-dhcp-relay
```
- Jalankan dhcp server
```
service isc-dhcp-relay restart
service isc-dhcp-relay status
```
- Ubah file /etc/default/isc-dhcp-relay
```
# Defaults for isc-dhcp-relay initscript
# sourced by /etc/init.d/isc-dhcp-relay
# installed at /etc/default/isc-dhcp-relay by the maintainer scripts

#
# This is a POSIX shell fragment
#

# What servers should the DHCP relay forward requests to?
SERVERS="10.5.0.11"

# On what interfaces should the DHCP relay (dhrelay) serve DHCP requests?
INTERFACES="eth1 eth2"

# Additional options that are passed to the DHCP relay daemon?
OPTIONS=""
```
- Ubah file /etc/sysctl.conf dengan uncomment `net.ipv4.ip_forward=1`

#### Setup DHCP Server di Jippangu
- Install `isc-dhcp-server`
```
apt-get update
apt-get install isc-dhcp-server -y
```
- Setup /etc/default/isc-dhcp-server dengan uncomment `INTERFACES="eth0"`
- Tambahkan pengaturan subnetting berikut pada /etc/dhcp/dhcpd.conf
```
#A2
subnet 10.5.0.128 netmask 255.255.255.128 {
    range 10.5.0.130 10.5.0.255;
    option routers 10.5.0.129;
    option broadcast-address 10.5.0.255;
    option domain-name-servers 10.5.0.10;
    default-lease-time 360;
    max-lease-time 7200;
}

#A1
subnet 10.5.0.8 netmask 255.255.255.248 {
}

#A3
subnet 10.5.4.0 netmask 255.255.252.0 {
    range 10.5.4.2 10.5.7.255;
    option routers 10.5.4.1;
    option broadcast-address 10.5.7.255;
    option domain-name-servers 10.5.0.10;
    default-lease-time 360;
    max-lease-time 7200;
}

#A6
subnet 10.5.2.0 netmask 255.255.254.0 {
    range 10.5.2.2 10.5.3.255;
    option routers 10.5.2.1;
    option broadcast-address 10.5.3.255;
    option domain-name-servers 10.5.0.10;
    default-lease-time 360;
    max-lease-time 7200;
}

#A7
subnet 10.5.1.0 netmask 255.255.255.0 {
    range 10.5.1.2 10.5.1.255;
    option routers 10.5.1.1;
    option broadcast-address 10.5.1.255;
    option domain-name-servers 10.5.0.10;
    default-lease-time 360;
    max-lease-time 7200;
}
```
- Jalankan
```
service isc-dhcp-server start
service isc-dhcp-server restart
service isc-dhcp-server status
```

![Screenshot from 2021-12-11 20-59-09](https://user-images.githubusercontent.com/57583780/145679314-324a0c1b-1503-4eab-a831-6ad825375f03.png)


#### Setup DNS Server di Doriki
- Install bind9 dengan 
```
apt-get update
apt-get install bind9 -y
```
- Ubah forwarder pada `/etc/bind/named.conf.options`
```
	forwarders {
	 	10.5.0.1;
	};

```
- Start
```
service bind9 restart
```


## SETUP STATIC INTERNET (Foosha)
```
auto eth0
iface eth0 inet static
    address 192.168.122.25
    netmask 255.255.255.0
    gateway 192.168.122.1
```

- Jalankan `iptables -t nat -A POSTROUTING -s 10.5.0.0/21 -o eth0 -j SNAT --to-source 192.168.122.25` pada Foosha
???

### NO 1 
#### Foosha
```
iptables -t nat -A POSTROUTING -s 10.18.0.0/20 -o eth0 -j SNAT --to-source asal 192.168.122.25
```
-t nat: Tabel NAT digunakan karena akan mengubah alamat asal paket

-A POSTROUTING: Chain POSTROUTING digunakan karena mengubah alamat asal paket setelah routing

-s 10.18.0.0/20: Mendefinisikan alamat alamat asal dari paket yaitu semua alamat IP dari subnet 10.18.0.0/20

-o eth0: Output (paket) keluar dari eth0 Foosha

-j SNAT: Menggunakan target SNAT untuk mengubah alamat asal paket

--to-s (ip eth0): Mendefinisikan IP alamat asal, yaitu eth0 Foosha


![Screenshot from 2021-12-11 21-02-15](https://user-images.githubusercontent.com/57583780/145679380-6608afe0-696e-41a5-8b27-a253cd38d24c.png)

???

### NO 2 
#### Doriki (DNS Server) & Jipangu (DHCP Server)
```
iptables -A FORWARD -d 10.5.0.8/29 -i eth0 -p tcp --dport 80 -j DROP
```

-A FORWARD: Chain FORWARD

-p tcp: Mendefinisikan protokol yang digunakan, yaitu tcp

--dport 80: Mendefinisikan port yang digunakan, yaitu 80 (HTTP)

-d 10.5.0.8/29: Mendefinisikan alamat tujuan dari paket (DHCP & DNS Server) 

-i eth0: Input (paket) masuk dari eth0 Foosha

-j DROP: Paket di-drop

#### TESTING NO 2
1. Install netcat di server **Jipangu dan Doriki** : `apt-get install netcat`

2. Pada **Jipangu dan Doriki**: `nc -l -p 80`

3. Pada Foosha: `nmap -p 80 10.5.0.10` atau `nmap -p 80 10.5.0.10`

Keterangan: 10.5.0.10 adalah IP dari subnet Jipangu

![Screenshot from 2021-12-11 21-05-10](https://user-images.githubusercontent.com/57583780/145679470-4214f368-c90a-4aee-940a-b0beebaeef77.png)

???

### NO 3
#### Doriki (DNS Server) & Jipangu (DHCP Server)
```
iptables -A INPUT -p icmp -m connlimit --connlimit-above 3 --connlimit-mask 0 -j DROP
```

-A INPUT: Chain INPUT

-p icmp: Mendefinisikan protokol yang digunakan, yaitu ICMP (ping)

-m connlimit: Rule connection limit

--connlimit-above 3: Limit paket yang ditangkap adalah lebih dari 3

--connlimit-mask 0 : Hanya memperbolehkan 3 koneksi setiap subnet dalam satu waktu

-j DROP: Paket di-drop

#### TESTING NO 3
Masuk ke empat node berbeda
Ping ke Jipangu/Doriki secara bersamaan
Jika berhasil, maka node ke-4 tidak dapat melakukan ping ke Jipangu/Doriki karena sudah lebih dari 3

![Screenshot from 2021-12-11 21-09-09](https://user-images.githubusercontent.com/57583780/145679586-3ab30896-e2c7-4464-9abb-ba6f75512abe.png)

### NO 4
#### Doriki (DNS Server) 

##### BATAS AKSES CHIPER KE DORIKI
```
iptables -A INPUT -s 10.5.0.130/25 -d 10.5.0.10/29 -m time --timestart 07:00 --timestop 15:00 --weekdays Mon,Tue,Wed,Thu -j ACCEPT
```

```
iptables -A INPUT -s 10.5.0.130/25 -j REJECT
```

![Screenshot from 2021-12-11 21-18-53](https://user-images.githubusercontent.com/57583780/145679866-02980785-bd76-46fe-a0ce-4fe99bd95e53.png)

#####  BATAS AKSES BLUENO KE DORIKI
```
iptables -A INPUT -s 10.5.4.2/22  -d 10.5.0.10/29 -m time --timestart 07:00 --timestop 15:00 --weekdays Mon,Tue,Wed,Thu -j ACCEPT
```

```
iptables -A INPUT -s 10.5.4.2/22  -j REJECT
```

![Screenshot from 2021-12-11 21-18-03](https://user-images.githubusercontent.com/57583780/145679839-fe41d918-32af-4d2f-b684-b82ee221a6d9.png)


Keterangan:
A INPUT : Menggunakan chain INPUT

s 10.5.0.130/25 : Alamat asal dari paket yaitu IP dari subnet Blueno

s 10.5.4.2/22 : Alamat asal dari paket yaitu IP dari subnet Chiper

d 10.5.0.10/29 : Alamat tujuan dari paket yaitu IP dari subnet Doriki

m time : Menggunakan rule time

-timestart 07:00 : Mendefinisikan waktu mulai yaitu 07:00

-timestop 15:00: : Mendefinisikan waktu berhenti yaitu 15:00

--weekdays Mon,Tue,Wed,Thu : Mendefinisikan hari yaitu Senin hingga Kamis

-j ACCEPT : Paket diterima

-j REJECT : Paket ditolak

### NO 5
#### Doriki (DNS Server) 

#####  BATAS AKSES ELENA KE DORIKI
```
iptables -A INPUT -s 10.5.2.2/23 -m time --timestart 15:01 --timestop 06:59 -j ACCEPT
```

```
iptables -A INPUT -s 10.5.2.2/23 -j REJECT
```

![Screenshot from 2021-12-11 21-22-36](https://user-images.githubusercontent.com/57583780/145679984-becbd3e9-2412-422c-8078-bd50092ad7ff.png)


#####  BATAS AKSES FUKUROU KE DORIKI
```
iptables -A INPUT -s 10.5.1.2/24 -m time --timestart 15:01 --timestop 06:59 -j ACCEPT
```

```
iptables -A INPUT -s 10.5.1.2/24-j REJECT
```

![Screenshot from 2021-12-11 21-21-27](https://user-images.githubusercontent.com/57583780/145679960-fbb566c5-9b0d-4462-9ecb-107684e17488.png)


Keterangan:
A INPUT : Chain INPUT

s 10.5.2.2/23 : Alamat asal dari paket yaitu IP dari subnet Elena

s 10.5.1.2/24 : Alamat asal dari paket yaitu IP dari subnet Fukurou

m time : Menggunakan rule time

-timestart 15:01 : Mendefinisikan waktu mulai yaitu 07:00

-timestop 06:59 : Mendefinisikan waktu berhenti yaitu 15:00

-j ACCEPT : Paket diterima

-j REJECT : Paket ditolak

#### TESTING NO 5

1. `date -s "8 Nov 2021 10:00:00"`

2. `date -s "8 Nov 2021 17:00:00"`

### NO 6
#### Guanhao

#### Didistribusikan Jorge dan Maingate
```
iptables -A PREROUTING -t nat -p tcp -d 10.5.0.8/29 --dport 80 -m statistic --mode nth --every 2 --packet 0 -j DNAT --to-destination  10.5.0.18:80
```

```
iptables -A PREROUTING -t nat -p tcp -d 10.5.0.8/29 --dport 80 -j DNAT --to-destination 10.5.0.19:80
```

```
iptables -t nat -A POSTROUTING -p tcp -d 10.5.0.18 --dport 80 -j SNAT --to-alamat asal 10.5.0.8:80
```

```
iptables -t nat -A POSTROUTING -p tcp -d 10.5.0.19 --dport 80 -j SNAT --to-alamat asal 10.5.0.8:80
```

#### TESTING NO 6
1. Guanhao, Jorge, Maingate dan Elena:  install apt-get install netcat

2. Jorge: nc -l -p 80

3. Maingate: nc -l -p 80

4. Elena: nc 10.18.4.128 80

5. Ketikkan sembarang pada client Elena, nanti akan muncul bergantian

![Screenshot from 2021-12-11 21-24-27](https://user-images.githubusercontent.com/57583780/145680049-fb9dabf8-6384-4ed2-bde1-d8a73d7561ac.png)

![Screenshot from 2021-12-11 21-28-40](https://user-images.githubusercontent.com/57583780/145680196-81dbd78a-e0be-4eb8-8cfe-6545aee64214.png)

