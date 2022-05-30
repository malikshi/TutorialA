# TutorialA
V2rayA Tutorial untuk VVIP IPTUNNELS

**Table of Contents**

- [TutorialA](#tutoriala)
- [Beli VVIP IPTUNNELS](#beli-vvip-iptunnels)
- [Features](#features)
- [Setting Network](#setting-network)
- [Setting Mwan3](#setting-mwan3)
- [Install V2rayA diOpenWRT](#install-v2raya-diopenwrt)
- [Setting V2rayA](#setting-v2raya)
  - [Main Setting](#main-setting)
  - [DNS](#dns)
  - [OutBound](#outbound)
    - [Memilih Proxy Outbound](#memilih-proxy-outbound)
  - [RoutingA](#routinga)

# Beli VVIP IPTUNNELS

OpenClash Config untuk VVIP IPTUNNELS
- [Buy VVIP IPTUNNELS](https://linktr.ee/iptunnelscom)
- [Join Telegram](https://t.me/joinchat/RihiceTtK1QhBMm7)
- [Requests Rules](https://github.com/malikshi/TutorialA/issues/new/choose)

# Features

* Support MWAN3 rekomendasi 3 modem(2 modem inject + 1 modem reguler untuk game).
* Pisah Traffic TCP & UDP via MWAN3 & RoutingA V2rayA
* Pisah traffik umum, sosmed, streaming, gaming.
* Adblock, Privacy rules & P0rn.
* Support Gaming filtering port.
* Support Direct/Bypass traffik.

# Setting Network

Terdapat 3 Modem, dengan 2 modem sebagai inject, dan 1 modem reguler. Kita setting interface name sebagai berikut,
* WAN A : Inject
* WAN B : Inject
* WAN C : Reguler

Berikut Settingan Networknya
```conf

config interface 'loopback'
        option device 'lo'
        option proto 'static'
        option ipaddr '127.0.0.1'
        option netmask '255.0.0.0'

config globals 'globals'
        option ula_prefix 'fd95:7b72:6986::/48'

config device
        option name 'br-lan'
        option type 'bridge'
        list ports 'eth0'

config interface 'lan'
        option device 'br-lan'
        option proto 'static'
        option ipaddr '192.168.1.1'
        option netmask '255.255.255.0'
        option ip6assign '60'

config interface 'WAN_A'
        option proto 'dhcp'
        option device 'eth1'
        option metric '10'

config interface 'WAN_B'
        option proto 'dhcp'
        option metric '20'

config interface 'WAN_C'
        option proto 'dhcp'
        option device 'eth2'
        option metric '30'

```
Simpan config network
```sh
uci commit network
```
Saya anggap firewall sudah disetting, lan-br/eth0 sebagai LAN. WAN A, WAN B, WAN C sebagai WAN.

**REBOOT OPENWRT JIKA DIPERLUKAN**

# Setting Mwan3

Karena sudah disediakan modem reguler untuk handle traffic udp maka kita pisahkan traffic tcp dan udp dengan detail sebagai berikut:
* rule balanced : WAN A & WAN B mode load-balance 50-50. rule ini akan menghandle segala traffic TCP.
* rule UDP : karena mwan3 wajib minimal 2 WAN maka perlu disetting WAN C sebagai utama dan WAN A/B sebagai member bayangan. mode load-balance 99-1(WAN C- WAN A/B).

Berikut config mwan3:
```conf

config globals 'globals'
        option mmx_mask '0x3F00'

config policy 'balanced'
        option last_resort 'unreachable'
        list use_member 'WAN_A_ETH1'
        list use_member 'WAN_B_WLAN0'

config rule 'https'
        option sticky '1'
        option dest_port '443'
        option proto 'tcp'
        option use_policy 'balanced'

config rule 'default_rule_v4'
        option dest_ip '0.0.0.0/0'
        option use_policy 'balanced'
        option family 'ipv4'
        option proto 'tcp'
        option sticky '0'

config rule 'default_rule_v6'
        option dest_ip '::/0'
        option use_policy 'balanced'
        option family 'ipv6'
        option proto 'tcp'
        option sticky '0'

config interface 'WAN_A'
        option enabled '1'
        option initial_state 'offline'
        option family 'ipv4'
        list track_ip '10.0.8.1'
        option track_method 'ping'
        option reliability '1'
        option count '1'
        option size '56'
        option max_ttl '60'
        option timeout '4'
        option interval '10'
        option failure_interval '5'
        option recovery_interval '5'
        option down '5'
        option up '5'

config interface 'WAN_B'
        option initial_state 'offline'
        option family 'ipv4'
        list track_ip '192.168.20.1'
        option track_method 'ping'
        option reliability '1'
        option count '1'
        option size '56'
        option max_ttl '60'
        option timeout '4'
        option interval '10'
        option failure_interval '5'
        option recovery_interval '5'
        option down '5'
        option up '5'
        option enabled '1'

config interface 'WAN_C'
        option enabled '1'
        option initial_state 'offline'
        option family 'ipv4'
        list track_ip '192.168.11.1'
        option track_method 'ping'
        option reliability '1'
        option count '1'
        option size '56'
        option max_ttl '60'
        option timeout '4'
        option interval '10'
        option failure_interval '5'
        option recovery_interval '5'
        option down '5'
        option up '5'

config member 'WAN_A_ETH1'
        option interface 'WAN_A'
        option metric '1'
        option weight '50'

config member 'WAN_B_WLAN0'
        option interface 'WAN_B'
        option metric '1'
        option weight '50'

config policy 'UDP'
        option last_resort 'unreachable'
        list use_member 'UDP_A_ETH1'
        list use_member 'UDP_C_ETH2'

config rule 'udp_v4'
        option family 'ipv4'
        option dest_ip '0.0.0.0/0'
        option sticky '0'
        option use_policy 'UDP'
        option proto 'udp'

config rule 'udp_v6'
        option family 'ipv6'
        option proto 'udp'
        option dest_ip '::/0'
        option sticky '0'
        option use_policy 'UDP'

config member 'UDP_A_ETH1'
        option interface 'WAN_A'
        option metric '2'
        option weight '50'

config member 'UDP_C_ETH2'
        option interface 'WAN_C'
        option metric '2'
        option weight '50'

```
Simpan config mwan3
```sh
uci commit mwan3
```
Silahkan Ganti `list track_ip` menggunakan gateway modem masing-masing masing.
Jangan Lupa enable mwan3 saat startup.

**REBOOT OPENWRT JIKA DIPERLUKAN**

# Install V2rayA diOpenWRT

Merujuk referensi pada [V2rayA Openwrt](https://github.com/v2rayA/v2raya-openwrt)

```sh
wget https://osdn.net/projects/v2raya/storage/openwrt/v2raya.pub -O /etc/opkg/keys/94cc2a834fb0aa03
echo "src/gz v2raya https://osdn.net/projects/v2raya/storage/openwrt/$(. /etc/openwrt_release && echo "$DISTRIB_ARCH")" | tee -a "/etc/opkg/customfeeds.conf"
opkg update
opkg install v2ray-core
opkg install v2fly-geoip v2fly-geosite
uci set v2raya.config.enabled='1'
uci commit v2raya
/etc/init.d/v2raya start
```
Silahkan akses WEBUI V2rayA melalui IPOpenWRT:2017 contoh http://192.168.1.1:2017

# Setting V2rayA
Setelah akses webui nanti akan diminta membuat akun admin, silahkan isikan username dan password sesuai keinginan.

## Main Setting

Perhatian Gambar, dan mohon untuk Sesuaikan.
<img src="https://raw.githubusercontent.com/malikshi/TutorialA/main/assets/Setting.jpg" border="0">


## DNS

Untuk DNS kita akan memilih DNS query dari google atau cloudflare. Silahkan ke menu `prevent DNS Spoofing` dan `klik Configure` kemudian isikan dns sebagai berikut:
<img src="https://raw.githubusercontent.com/malikshi/TutorialA/main/assets/DNS.jpg" border="0">
- Domain Query Servers
```conf
8.8.8.8 ->proxy
8.8.4.4 ->direct
```
- External Domain Query Servers
```conf
https://1.1.1.1:443/dns-query->proxy
https://1.0.0.1:443/dns-query->proxy
1.1.1.1->direct
1.0.0.1->direct
```
Jika server/proxy support IPv6, anda bisa ganti main setting `Special Mode` dengan `FakeDNS`.

## OutBound

Outbound ini kita akan membuat proxy-groups dan memilih proxy untuk setiap group yang telah dibuat. Disini default terdapat proxy-group: `proxy`, dan kita perlu menambahkan proxy-groups Streaming, Sosmed, dan Gaming.
Silahkan klik `Add an outbound` dan beri nama tiap outbound sebagai berikut:
- Streaming
- Sosmed
- Gaming


Perhatikan Gambar
<img src="https://raw.githubusercontent.com/malikshi/TutorialA/main/assets/Outbound.jpg" border="0">

### Memilih Proxy Outbound

Cara memilih proxy seperti vmess, vless, trojan, trojan-go, shadowsocks akan digunakan oleh outbound proxy-groups yang telah dibuat.
- Klik `PROXY` kemudian Pilih akun server yang akan digunakan oleh Outbound `proxy` dengan cara klik `Select` pada server.
- Klik `PROXY` kemudian arahkan dan klik ke outbound `Streaming`, pilih server dengan cara klik `Select` pada server.
- Klik `PROXY` kemudian arahkan dan klik ke outbound `Sosmed`, pilih server dengan cara klik `Select` pada server.
- Klik `PROXY` kemudian arahkan dan klik ke outbound `Gaming`, pilih server dengan cara klik `Select` pada server.

Pemilihan Server bisa lebih dari 1, saran tiap `Outbound` terdapat 2 server yang terpilih. Khusus untuk Outbound `Gaming` saya rekomendasikan memilih 2 server region indo, jika tidak yah gunakan 1 proxy itu sangat saya rekomendasikan.

## RoutingA

RoutingA memudahkan users untuk membuat rule untuk setiap traffic akan melalui proxy-groups `Outbound` yang mana saja.

- Silahkan Copy RoutingA yang telah kami sediakan di [RoutingA](https://raw.githubusercontent.com/malikshi/TutorialA/main/routingA.conf)
- Pastekan pada menu `Setting > RoutingA > Configure`

Kelebihan RoutingA yang kami Sediakan yakni, semua traffic UDP akan melalui WAN/modem reguler jadi ping saat bermain game dan videocall serta voice chat akan berjalan dengan sangat bagus.