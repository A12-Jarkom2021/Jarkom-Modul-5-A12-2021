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

