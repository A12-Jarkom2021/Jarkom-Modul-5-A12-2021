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


### D
Tugas berikutnya adalah memberikan ip pada subnet Blueno, Cipher, Fukurou, dan Elena secara dinamis menggunakan bantuan DHCP server. Kemudian kalian ingat bahwa kalian harus setting DHCP Relay pada router yang menghubungkannya.

## SETUP STATIC INTERNET (Foosha)
```
auto eth0
iface eth0 inet static
    address 192.168.122.25
    netmask 255.255.255.0
    gateway 192.168.122.1
```
—

### NO 1 
#### Foosha
```
iptables -t nat -A POSTROUTING -s 10.18.0.0/20 -o eth0 -j SNAT --to-alamat asal 192.168.122.25
```
-t nat: Tabel NAT digunakan karena akan mengubah alamat asal paket

-A POSTROUTING: Chain POSTROUTING digunakan karena mengubah alamat asal paket setelah routing

-s 10.18.0.0/20: Mendefinisikan alamat alamat asal dari paket yaitu semua alamat IP dari subnet 10.18.0.0/20

-o eth0: Output (paket) keluar dari eth0 Foosha

-j SNAT: Menggunakan target SNAT untuk mengubah alamat asal paket

--to-s (ip eth0): Mendefinisikan IP alamat asal, yaitu eth0 Foosha

—

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

—

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

### NO 4
#### Doriki (DNS Server) 

##### BATAS AKSES CHIPER KE DORIKI
```
iptables -A INPUT -s 10.5.0.130/25 -d 10.5.0.10/29 -m time --timestart 07:00 --timestop 15:00 --weekdays Mon,Tue,Wed,Thu -j ACCEPT
```

```
iptables -A INPUT -s 10.5.0.130/25 -j REJECT
```

#####  BATAS AKSES BLUENO KE DORIKI
```
iptables -A INPUT -s 10.5.4.2/22  -d 10.5.0.10/29 -m time --timestart 07:00 --timestop 15:00 --weekdays Mon,Tue,Wed,Thu -j ACCEPT
```

```
iptables -A INPUT -s 10.5.4.2/22  -j REJECT
```

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

#####  BATAS AKSES FUKUROU KE DORIKI
```
iptables -A INPUT -s 10.5.1.2/24 -m time --timestart 15:01 --timestop 06:59 -j ACCEPT
```

```
iptables -A INPUT -s 10.5.1.2/24-j REJECT
```

Keterangan:
A INPUT : Chain INPUT

s 10.5.2.2/23 : Alamat asal dari paket yaitu IP dari subnet Elena

s 10.5.1.2/24 : Alamat asal dari paket yaitu IP dari subnet Fukurou

m time : Menggunakan rule time

-timestart 15:01 : Mendefinisikan waktu mulai yaitu 07:00

-timestop 06:59 : Mendefinisikan waktu berhenti yaitu 15:00

-j ACCEPT : Paket diterima

-j REJECT : Paket ditolak

#### TESTING NO 6

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
