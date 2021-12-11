# Jarkom-Modul-5-A12-2021
Jarkom-Modul-5-A12-2021

## SETUP STATIC INTERNET (Foosha)
auto eth0
iface eth0 inet static
    address 192.168.122.25
    netmask 255.255.255.0
    gateway 192.168.122.1

### NO 1 
#### Foosha
```
iptables -t nat -A POSTROUTING -s 10.18.0.0/20 -o eth0 -j SNAT --to-source 192.168.122.25
```
-t nat: Menggunakan tabel NAT karena akan mengubah alamat asal dari paket

-A POSTROUTING: Menggunakan chain POSTROUTING dikarenakan mengubah source paket setelah routing

-s 10.18.0.0/20: Mendefinisikan alamat source dari paket yaitu semua alamat IP dari subnet 10.18.0.0/20

-o eth0: Paket keluar dari eth0 Foosha

-j SNAT: Menggunakan target SNAT untuk mengubah source dari paket

--to-s (ip eth0): Mendefinisikan IP source, di mana digunakan eth0 Foosha

—
### NO 2 
#### Doriki (DNS Server) & Jipangu (DHCP Server)
```
iptables -A FORWARD -d 10.5.0.8/29 -i eth0 -p tcp --dport 80 -j DROP
```

-A FORWARD: Menggunakan chain FORWARD

-p tcp: Mendefinisikan protokol yang digunakan, yaitu tcp

--dport 80: Mendefinisikan port yang digunakan, yaitu 80 (HTTP)

-d 10.5.0.8/29: Mendefinisikan alamat tujuan dari paket (DHCP & DNS Server) 

-i eth0: Paket masuk dari eth0 Foosha

-j DROP: Paket di-drop

#### Testing NO 2
1. Install netcat di server Jipangu dan Doriki: apt-get install netcat

2. Pada Jipangu dan Doriki ketikkan: nc -l -p 80

3. Pada Foosha: nmap -p 80 10.5.0.10 atau nmap -p 80 10.5.0.10

—
### NO 3
#### Doriki (DNS Server) & Jipangu (DHCP Server)
```
iptables -A INPUT -p icmp -m connlimit --connlimit-above 3 --connlimit-mask 0 -j DROP
```

-A INPUT: Menggunakan chain INPUT

-p icmp: Mendefinisikan protokol yang digunakan, yaitu ICMP (ping)

-m connlimit: Menggunakan rule connection limit

--connlimit-above 3: Limit yang ditangkap paket adalah di atas 3

--connlimit-mask 0 : Hanya memperbolehkan 3 koneksi setiap subnet dalam satu waktu

-j DROP: Paket di-drop

#### Testing NO 3
Masuk ke 4 node berbeda

Ping ke Jipangu/Doriki secara bersamaan


—
