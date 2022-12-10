# Jarkom-Modul-5-D10-2022
Repositori Jawaban Soal Shift Jarkom Modul 5: Firewall

## Anggota Kelompok:
|Nama     |NRP    |
|----------|-------|
|Hasna Lathifah Purbaningdyah|5025201108|
|Anggito Anju Hartawan Manalu|5025201216|
|Hidayatullah|5025201031|

# Pembagian IP
Dalam mengerjakan soal ini, kami menggunakan teknik CIDR.

### Subnet
<img width="367" alt="image" src="https://user-images.githubusercontent.com/80145586/206858759-9124ebc2-6b49-4f98-ab42-5472c75ca587.png">

<img width="511" alt="image" src="https://user-images.githubusercontent.com/80145586/206858739-7f3f105d-7326-40db-ab11-2736eb74f91c.png">

Menggunakan CIDR didapat pembagian seperti berikut 
<img width="666" alt="image" src="https://user-images.githubusercontent.com/80145586/206858823-77e98875-af24-47dc-8366-cb4d8f14b602.png">

<img width="529" alt="image" src="https://user-images.githubusercontent.com/80145586/206858927-422467b4-8f91-49cd-8c77-294eaa359a5d.png">

### Routing
Dengan CIDR Routing cukup menuju ke router yang berhubungan dengan router lainnya, pada kasus ini hanya Strix.
```
# Routing Strix ke Ostania
route add -net 10.20.0.0 netmask 255.255.248.0 gw 10.20.0.2

# Routing Strix ke Westalis
route add -net 10.20.16.0 netmask 255.255.240.0 gw 10.20.16.2
```

### Configurasi Interface 

#### Strix
```
auto eth0
iface eth0 inet dhcp

# Ostania
auto eth1
iface eth1 inet static
        address 10.20.0.1
        netmask 255.255.255.252

# Westalis
auto eth2
iface eth2 inet static
        address 10.20.16.1
        netmask 255.255.255.252
```
#### Ostania
```
auto eth0
iface eth0 inet static
        address 10.20.0.2
        netmask 255.255.255.252
        gateway 10.20.0.1

# Blackbell
auto eth1
iface eth1 inet static
        address 10.20.6.1
        netmask 255.255.254.0

# Web Server
auto eth2
iface eth2 inet static
        address 10.20.4.1
        netmask 255.255.255.248

# Briar
auto eth3
iface eth3 inet static
        address 10.20.5.1
        netmask 255.255.255.0
```

#### Westalis

```
auto eth0
iface eth0 inet static
        address 10.20.16.2
        netmask 255.255.255.252
        gateway 10.20.16.1

# Desmond
auto eth1
iface eth1 inet static
        address 10.20.28.1
        netmask 255.255.252.0

# Forger
auto eth2
iface eth2 inet static
        address 10.20.24.129
        netmask 255.255.255.128
 
# DNS dan DHCP Server
auto eth3
iface eth3 inet static
        address 10.20.24.1
        netmask 255.255.255.248
```
#### Garden
```
auto eth0
iface eth0 inet static
        address 10.20.4.2
        netmask 255.255.255.248
        gateway 10.20.4.1
```
#### SSS
```
auto eth0
iface eth0 inet static
        address 10.20.4.3
        netmask 255.255.255.248
        gateway 10.20.4.1
```
#### Eden
```
auto eth0
iface eth0 inet static
        address 10.20.24.2
        netmask 255.255.255.248
        gateway 10.20.24.1
```
#### WISE
```
auto eth0
iface eth0 inet static
        address 10.20.24.3
        netmask 255.255.255.248
        gateway 10.20.24.1
```
#### Client
Semua klien menggunakan DHCP
```
auto eth0
iface eth0 inet dhcp
```

# Setting DHCP Server di WISE
- Install `isc-dhcp-server`
```
apt-get update
apt-get install isc-dhcp-server -y
```
- Setup /etc/default/isc-dhcp-server dengan uncomment `INTERFACES="eth0"`
- Tambahkan pengaturan subnetting berikut pada /etc/dhcp/dhcpd.conf
```
subnet 10.20.24.0 netmask 255.255.255.248 {

}

subnet 10.20.24.128 netmask 255.255.255.128 {
    range 10.20.24.130 10.20.24.254;
    option routers 10.20.24.129;
    option broadcast-address 10.20.24.255;
    option domain-name-servers 10.20.24.2;
    default-lease-time 600;
    max-lease-time 7200;
}

subnet 10.20.28.0 netmask 255.255.252.0 {
    range 10.20.28.2 10.20.31.254;
    option routers 10.20.28.1;
    option broadcast-address 10.20.31.255;
    option domain-name-servers 10.20.24.2;
    default-lease-time 600;
    max-lease-time 7200;
}

subnet 10.20.6.0 netmask 255.255.254.0 {
    range 10.20.6.2 10.20.7.254;
    option routers 10.20.6.1;
    option broadcast-address 10.20.7.255;
    option domain-name-servers 10.20.24.2;
    default-lease-time 600;
    max-lease-time 7200;
}

subnet 10.20.5.0 netmask 255.255.255.0 {
    range 10.20.5.2 10.20.5.254;
    option routers 10.20.5.1;
    option broadcast-address 10.20.5.255;
    option domain-name-servers 10.20.24.2;
    default-lease-time 600;
    max-lease-time 7200;
}
```
- Jalankan
```
service isc-dhcp-server restart
service isc-dhcp-server status
```
<img width="368" alt="image" src="https://user-images.githubusercontent.com/80145586/206859413-4234cfd9-561b-4918-9e16-9b98fb9d89f4.png">


# Setting DHCP Relay
- Install `isc-dhcp-relay` dan sekaligus memasukkan argumen konfigurasi IP yang dituju (IP DHCP Server) dan Interface yang dihandle menggunakan `echo`. 
```
apt-get update
echo -e "10.20.24.3\neth0 eth1 eth2 eth3\n\n" | apt-get install isc-dhcp-relay -y
```
Opsi `-e` digunakan untuk newline. Dan untuk Strix hanya cukup sampai eth2.

- Isi `/etc/default/isc-dhcp-relay`
## Westalis
<img width="335" alt="image" src="https://user-images.githubusercontent.com/80145586/206859420-80e313ad-60e9-46ac-a230-de64adc15c3a.png">

## Strix
<img width="333" alt="image" src="https://user-images.githubusercontent.com/80145586/206859431-19dfcba2-4b83-4ccf-abbd-e0197aa3a9fe.png">

## Ostania
<img width="332" alt="image" src="https://user-images.githubusercontent.com/80145586/206859461-cb2a2894-eb79-4d79-950a-4474a0689bec.png">


# No 1
> Agar topologi yang kalian buat dapat mengakses keluar, kalian diminta untuk mengkonfigurasi Strix menggunakan iptables, tetapi Loid tidak ingin menggunakan MASQUERADE

Menggunakan `SNAT` dengan argumen `--to-source` menuju ke IP Strix yang berhubungan dengan NAT. IP Strix yang berhubungan dengan NAT menggunakan DHCP sehingga terkadang IPnya berbeda. Karena dari itu, perlu ditemukan IP Strix saat ini terlebih dahulu sebelum memberi aturan Iptables. 

Command `hostname -I | awk'{ print $1 }'` dapat menunjukkan IP Strix pada interface pertama, dalam kasus ini interface `eth0` yang berhubungan dengan NAT. Hasilnya disimpan dalam variable `thisip` dan digunakan dalam command iptables.

```bash
# Strix
thisip=$(hostname -I | awk '{ print $1 }')
iptables -t nat -A POSTROUTING -s 10.20.0.0/19 -o eth0 -j SNAT --to-source $thisip
```
## Testing
Testing pada node selain node yang berhubungan langsung dengan NAT.

<img width="415" alt="image" src="https://user-images.githubusercontent.com/80145586/206859608-9df950e0-f625-4547-91e6-d475b97c1793.png">

<img width="416" alt="image" src="https://user-images.githubusercontent.com/80145586/206859690-ba66ac07-b974-49f4-9a25-c044e1582d5f.png">

# No 2
> Kalian diminta untuk melakukan drop semua TCP dan UDP dari luar Topologi kalian pada server yang merupakan DHCP Server demi menjaga keamanan.

Salah satu caranya ialah dengan meng*intercept* paket yang berasal dari NAT dan menuju ke DHCP Server sejak pertama kali masuk ke dalam topologi (Strix). Dengan menambahkan aturan untuk forwarding packet pada router Strike dengan protokol udp dan tcp yang menuju ke IP DHCP Server untuk di Drop.

```Bash
# Strix (-d IP DHCP Server)
iptables -A FORWARD -p tcp -d 10.20.24.3 -i eth0 -j DROP # Drop semua TCP
iptables -A FORWARD -p udp -d 10.20.24.3 -i eth0 -j DROP # Drop semua UDP
```
## Testing
Apt-get menggunakan HTTP/HTTPS sehingga seharusnya tidak dapat menerima paket pada node WISE 
<img width="413" alt="image" src="https://user-images.githubusercontent.com/80145586/206860210-c4132790-fdfd-406f-a8fa-a8eaa052e06f.png">

# No 3
> Loid meminta kalian untuk membatasi DHCP dan DNS Server hanya boleh menerima maksimal 2 koneksi ICMP secara bersamaan menggunakan iptables, selebihnya didrop.

Menambahkan aturan INPUT pada Iptables dengan protokol icmp dengan menggunakan ekstensi connlimit. Mensetting aturan dimana jika koneksi lebih dari 2 dan berlaku global, paket akan di drop.
```Bash
# Wise & Eden
iptables -A INPUT -p icmp -m connlimit --connlimit-above 2 --connlimit-mask 0 -j DROP
```
## Testing
<img width="292" alt="image" src="https://user-images.githubusercontent.com/80145586/206860318-01fa31e9-2eb0-49ce-8be7-0b532e7623bb.png">
<img width="294" alt="image" src="https://user-images.githubusercontent.com/80145586/206860326-ea5407de-25c1-4a29-9fa7-0054abc7e3ac.png">
<img width="342" alt="image" src="https://user-images.githubusercontent.com/80145586/206860331-5c490ec1-8786-46fb-bf4f-25fd430f6b83.png">

WISE akan mendrop paket ketika ada lebih dari 2 koneksi secara bersamaan

# No 4
> Akses menuju Web Server hanya diperbolehkan disaat jam kerja yaitu Senin sampai Jumat pada pukul 07.00 - 16.00.

Menambahkan aturan INPUT pada iptables di kedua web server dimana untuk semua paket yang menuju ke IP tiap web server dari jam 7-16 pada hari kerja akan diterima, selain itu aka ditolak. Aturan dapat diimplementasi dengan menggunakan ekstensi time yang menambahkan aturan `--timestart`, `--timestop`, serta `weekdays`.

```bash
# Garden (-d IP Garden)
iptables -A INPUT -d 10.20.4.2/29 -m time --timestart 07:00 --timestop 16:00 --weekdays Mon,Tue,Wed,Thu,Fri -j ACCEPT
iptables -A INPUT -d 10.20.4.2/29 -j REJECT

# SSS (-d IP SSS)
iptables -A INPUT -d 10.20.4.3/29 -m time --timestart 07:00 --timestop 16:00 --weekdays Mon,Tue,Wed,Thu,Fri -j ACCEPT
iptables -A INPUT -d 10.20.4.3/29 -j REJECT
```
## Testing
- `Mengakses pada jam kerja`
<img width="319" alt="image" src="https://user-images.githubusercontent.com/80145586/206860093-0e0dafd4-060e-4fcb-8c9c-7fdcdcd7b963.png">

- `Mengakses diluar jam kerja`
<img width="388" alt="image" src="https://user-images.githubusercontent.com/80145586/206860182-cbdac2c4-0383-4f9a-b22a-bfe459796ad2.png">

# No 5 

```
#Ostania
iptables -A PREROUTING -t nat -p tcp --dport 80 -d 10.20.24.2 -m statistic --mode nth --every 2 --packet 0 -j DNAT --to-destination 10.20.24.3:80
iptables -A PREROUTING -t nat -p tcp --dport 443 -d 10.20.24.3 -m statistic --mode nth --every 2 --packet 0 -j DNAT --to-destination 10.20.24.2:443
```
Kendala:
- Ada masalah dengan implementasi web server
