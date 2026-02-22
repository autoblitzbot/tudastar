# Raspberry Pi 5 VPN Router — Tailscale Exit Node-on keresztül

Házi VPN router építése Raspberry Pi 5-ből, OpenWrt-vel és Tailscale-lel. Az összes LAN kliens forgalma egy távoli Tailscale exit node-on megy ki — VPN kliens nélkül a klienseken.

## Alkatrészlista

| Alkatrész | Ajánlás | Megjegyzés |
|-----------|---------|------------|
| **Raspberry Pi 5** (4GB vagy 8GB) | [Hivatalos forgalmazók](https://www.raspberrypi.com/products/raspberry-pi-5/) | 4GB bőven elég routernek |
| **USB 3.0 Gigabit Ethernet adapter** | RTL8153 chipset-es adapter | Lásd lent |
| **microSD kártya** (16GB+) | Samsung EVO Plus, SanDisk Extreme | Class 10 / A1 minimum |
| **Tápegység** | Hivatalos Pi 5 USB-C 27W (5.1V/5A) | A USB GbE adapter extra áramot húz, ne spórolj a tápon |
| **Hűtő** | [Official Active Cooler](https://www.raspberrypi.com/products/active-cooler/) vagy fém ház ventilátorral | Router folyamatosan megy, hűtés kell |
| **Ethernet kábel** (2 db) | Cat5e vagy Cat6 | WAN + LAN |

### USB Ethernet adapter — melyiket?

**RTL8153 chipset-et keress**, ne AX88179-et. A Realtek driver jobb Linux/OpenWrt támogatással bír, stabilabb és gyorsabb a Pi-n.

Konkrét ajánlások:
- **Ugreen USB 3.0 Gigabit Ethernet Adapter** — RTL8153, széles körben elérhető, olcsó
- **Cable Matters USB 3.0 to Gigabit Ethernet** — RTL8153, megbízható
- **TP-Link UE300** — RTL8153, de néha boot-kompatibilitási gondok lehetnek Pi-vel

> **Tipp:** Az adaptert kapcsold be a Pi bekapcsolása *előtt* — egyes adapterek újraindítást okozhatnak, ha menet közben dugod be.

## Hálózati topológia

```
Internet ←→ ISP Router ←→ [eth0 WAN] RPi5 [eth1 LAN] ←→ Switch/AP ←→ Kliensek
                                          ↕
                                    Tailscale tunnel
                                          ↕
                                    Távoli Exit Node
```

- **eth0** (beépített) → WAN (ISP router felé, DHCP kliens)
- **eth1** (USB adapter) → LAN (belső hálózat, DHCP szerver)
- A Pi NAT-olja a LAN forgalmat, és a Tailscale tunnel-en keresztül egy távoli exit node-ra irányítja

## 1. OpenWrt telepítés

### Image letöltés

1. Nyisd meg az [OpenWrt Firmware Selector](https://firmware-selector.openwrt.org/?version=24.10.1&target=bcm27xx/bcm2712&id=rpi-5) oldalt
2. Válaszd a **24.10.1** stabil verziót (vagy újabbat, ha elérhető)
3. Töltsd le a **Factory (SQUASHFS)** image-et (`.img.gz`)

> **Miért SquashFS és nem EXT4?** A SquashFS read-only root + írható overlay-t használ. Előnye: `firstboot` paranccsal bármikor visszaállítható a gyári állapot (hasznos, ha kizárod magad a tűzfallal), és kevesebb SD kártya írást jelent → hosszabb élettartam. Az EXT4 image-nél nincs factory reset lehetőség.

### SD kártyára írás

Linux/macOS:
```bash
# Kicsomagolás és írás (a /dev/sdX helyett a kártya tényleges eszköze)
gunzip openwrt-*.img.gz
sudo dd if=openwrt-*.img of=/dev/sdX bs=4M status=progress
sync
```

Vagy használj [Raspberry Pi Imager](https://www.raspberrypi.com/software/)-t: "Use custom" → válaszd ki az `.img` fájlt.

### Első indítás

1. Helyezd be a microSD-t a Pi-be
2. **Előbb** dugd be a USB Ethernet adaptert
3. Kösd be az **eth0**-t (beépített) a laptopodba vagy switchbe
4. Tápot rá
5. Várj ~1 percet, amíg bootol
6. Böngészőben: **http://192.168.1.1** → LuCI web felület
7. Első lépés: állíts be root jelszót: **System → Administration → Router Password**

## 2. Hálózat konfigurálás

### Interfészek beazonosítása

SSH-val (`ssh root@192.168.1.1`):

```bash
# Interfészek listázása
ip link show
# Általában: eth0 = beépített, eth1 = USB adapter
# Ellenőrzés:
dmesg | grep -i eth
```

### WAN interfész (eth0 → ISP felé)

**Network → Interfaces → Add new interface:**

| Mező | Érték |
|------|-------|
| Name | `wan` |
| Protocol | DHCP client |
| Device | `eth0` |
| Firewall zone | `wan` |

### LAN interfész (eth1 → belső hálózat)

Az alapértelmezett `lan` interfész a `br-lan` bridge-en van. Módosítsd:

**Network → Interfaces → LAN → Edit:**

| Mező | Érték |
|------|-------|
| Protocol | Static address |
| IPv4 address | `192.168.2.1` |
| Netmask | `255.255.255.0` |
| Device | `eth1` (USB adapter) |
| DHCP | Enabled (alapértelmezett tartomány OK) |
| Firewall zone | `lan` |

> **Fontos:** Ha az ISP routered is 192.168.1.x-et használ, a LAN-nak más subnetet adj (pl. 192.168.2.0/24), különben ütközés lesz.

### Tűzfal

**Network → Firewall:** Az alapértelmezett szabályok jók:
- `lan → wan`: **ACCEPT** (LAN kliensek kimennek az internetre)
- `wan → lan`: **REJECT** (kívülről nem jön be semmi)
- Masquerading (NAT): **ON** a `wan` zónánál

Mentés: **Save & Apply**.

### Teszt

A LAN portra kötött gépnek DHCP-vel kapnia kell 192.168.2.x címet, és el kell érnie az internetet az ISP routeren keresztül.

```bash
# A Pi-ről:
ping -c 3 1.1.1.1
# A LAN kliensről:
ping -c 3 8.8.8.8
```

## 3. Tailscale telepítés

### Csomag telepítés

```bash
opkg update
opkg install tailscale
```

> **Megjegyzés:** A Tailscale csomag ~22 MB helyet foglal. 16GB-os SD-vel nincs gond.

### Indítás és bejelentkezés

```bash
# Service engedélyezés és indítás
/etc/init.d/tailscale enable
/etc/init.d/tailscale start

# Bejelentkezés
tailscale up --accept-routes
```

Ez ad egy linket — nyisd meg böngészőben és jelentkezz be a Tailscale fiókoddal. A Pi megjelenik az admin konzolon: [https://login.tailscale.com/admin/machines](https://login.tailscale.com/admin/machines)

### Tűzfal zóna Tailscale-nek

**Network → Firewall → Add zone:**

| Mező | Érték |
|------|-------|
| Name | `tailscale` |
| Input | ACCEPT |
| Output | ACCEPT |
| Forward | ACCEPT |
| Covered devices | `tailscale0` |
| Allow forward from | `lan` |
| Allow forward to | `lan`, `wan` |

**Save & Apply.**

## 4. Csatlakozás távoli Exit Node-hoz

### Előfeltétel: Exit Node a Tailscale hálózatodban

Kell egy másik gép a tailnet-edben, ami exit node-ként fut. Ez lehet:
- Egy VPS (pl. Hetzner, DigitalOcean) valahol a világban
- Egy másik otthoni gép más országban
- Bármilyen Tailscale-t futtató eszköz

Azon a gépen:
```bash
tailscale up --advertise-exit-node
```

Majd a [Tailscale Admin Console](https://login.tailscale.com/admin/machines)-ban engedélyezd az exit node-ot:
**Gép → "..." menü → Edit route settings → Use as exit node ✓**

### A Pi-t ráállítani az exit node-ra

```bash
# Exit node-ok listázása
tailscale exit-node list

# Csatlakozás (a <HOSTNAME> helyére az exit node neve)
tailscale set --exit-node=<HOSTNAME>

# Ellenőrzés — a Pi IP-je most az exit node IP-je kell legyen
curl -s https://ifconfig.me
```

### Miért működik a LAN klienseknek is?

A LAN kliensek a Pi-n NAT-olva mennek ki. Amikor a Pi forgalma az exit node-on megy keresztül, **automatikusan az összes LAN kliens forgalma is oda megy** — nem kell Tailscale klienst telepíteni a kliensekre.

### Exit node kikapcsolás

```bash
# Visszaállás normál routing-ra (ISP-n ki)
tailscale set --exit-node=
```

## 5. Automatikus indítás és optimalizálás

### CPU Performance governor

Router folyamatos terhelés alatt van, a powersave governor lassítja a crypto műveleteket:

```bash
# Azonnali beállítás
echo performance > /sys/devices/system/cpu/cpufreq/policy0/scaling_governor

# Boot-kor automatikusan — add hozzá az /etc/rc.local-hoz (exit 0 elé):
echo performance > /sys/devices/system/cpu/cpufreq/policy0/scaling_governor
```

### Tailscale exit node beállítás újraindítás után

A `tailscale set --exit-node=<HOSTNAME>` **megmarad újraindítás után** — a Tailscale a beállítást a state fájlban tárolja (`/var/lib/tailscale/`), és a daemon újraindításkor automatikusan alkalmazza. Nem kell külön boot script.

### Watchdog

Ha a Tailscale tunnel leáll, a LAN kliensek elveszítik a netet. Egyszerű watchdog cron job:

```bash
cat << 'EOF' > /etc/tailscale-watchdog.sh
#!/bin/sh
# Ha nem elérhető az internet az exit node-on, újracsatlakozás
if ! ping -c 2 -W 5 1.1.1.1 > /dev/null 2>&1; then
    logger -t tailscale-watchdog "Connection lost, reconnecting..."
    tailscale set --exit-node=
    sleep 5
    tailscale set --exit-node=<HOSTNAME>
fi
EOF
chmod +x /etc/tailscale-watchdog.sh
```

Crontab-ba (5 percenként):

```bash
echo '*/5 * * * * /etc/tailscale-watchdog.sh' >> /etc/crontabs/root
/etc/init.d/cron restart
```

## Teljesítmény

| Mérés | Várható érték |
|-------|---------------|
| WireGuard (Tailscale) throughput | ~800–900 Mbps |
| Egy irányba max (CPU korlát) | ~1 Gbps |
| Latency overhead | +1–3 ms (LAN-on belül) |

A Pi 5 Cortex-A76 magjai ARMv8 crypto extension-nel rendelkeznek, ami a WireGuard-ot hardveresen gyorsítja. 1 Gbps-es internetnél a CPU a szűk keresztmetszet, nem a NIC.

## Hibaelhárítás

### USB adapter nem jelenik meg

```bash
dmesg | grep -i eth
lsusb
ip link show
```

Ha nem látszik `eth1`, próbáld ki kikapcsolt Pi-vel bedugni az adaptert, majd újra boot.

### Tailscale nem csatlakozik

```bash
tailscale status
# Ha "Logged out":
tailscale up --accept-routes
```

### LAN kliensek nem kapnak IP-t

```bash
# DHCP szerver fut?
/etc/init.d/dnsmasq status
# Log:
logread | grep dhcp
```

### Internet nem megy exit node-on

```bash
# Exit node státusz
tailscale exit-node list
# Ping teszt közvetlenül a Pi-ről
ping -c 3 1.1.1.1
# Ha nem megy, ideiglenes kikapcsolás:
tailscale set --exit-node=
# Ha így megy, az exit node-dal van gond
```

## Források

- [OpenWrt Firmware Selector — RPi 5](https://firmware-selector.openwrt.org/?version=24.10.1&target=bcm27xx/bcm2712&id=rpi-5)
- [Tailscale Exit Nodes dokumentáció](https://tailscale.com/kb/1103/exit-nodes)
- [Tailscale Subnet Routers dokumentáció](https://tailscale.com/kb/1019/subnets)
- [Tailscale OpenWrt telepítés — WunderTech](https://www.wundertech.net/how-to-set-up-tailscale-on-openwrt/)
- [OpenWrt Forum — Tailscale exit node 24.10-en](https://forum.openwrt.org/t/how-to-configure-tailscale-18-02-on-24-10-to-access-remote-exit-node/226561)
