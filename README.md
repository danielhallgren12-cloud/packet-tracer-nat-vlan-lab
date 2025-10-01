# Packet-Tracer-NAT-VLAN-Lab

Segmenterat nätverk med NAT och VLAN-routing

Projektbeskrivning
Detta projekt är en avancerad Packet Tracer-labb där jag byggde en segmenterad nätverkstopologi med två VLAN — VLAN 10 och VLAN 20 — fördelade över två routrar. Alla PC-enheter har identiska IP-adresser inom sina VLAN, vilket krävde en lösning med Static NAT för att möjliggöra full kommunikation mellan enheter på olika sidor av nätverket.

---

Topologiöversikt

| VLAN | Router | PC-enheter        | Subnet             | Gateway IP         |
|------|--------|-------------------|--------------------|--------------------|
| 10   | R1     | PC1, PC2          | 192.168.10.0/24    | 192.168.10.254     |
| 10   | R2     | PC5, PC6          | 192.168.10.0/24    | 192.168.10.254     |
| 20   | R1     | PC3, PC4          | 192.168.20.0/24    | 192.168.20.254     |
| 20   | R2     | PC7, PC8          | 192.168.20.0/24    | 192.168.20.254     |

---

VLAN-portstruktur

- Varje router är kopplad till sin närmaste switch via en **trunkport** för att möjliggöra VLAN-routing med subinterfaces.
- Under varje trunk-switch finns en **access-switch** per VLAN:
  - PC1 och PC2 är kopplade till en switch med **access VLAN 10**
  - PC3 och PC4 är kopplade till en switch med **access VLAN 20**
  - PC5 och PC6 är kopplade till en switch med **access VLAN 10**
  - PC7 och PC8 är kopplade till en switch med **access VLAN 20**
- Ingen trunk används mellan routrarna — VLAN-trafik är lokalt segmenterad per sida.

---

Tekniker och lösningar

- Static NAT för att översätta identiska IP-adresser till unika externa adresser
- Subinterfaces med dot1Q för VLAN-routing
- Statiska routes för att styra trafik mellan routrar
- Gateway-justering för att matcha rätt VLAN-interface
- Rensning av gamla konfigurationer för att undvika routingkonflikter

---

NAT-tabell

| Lokal IP        | Översatt IP     | VLAN | Router | Ruttas via |
|-----------------|----------------|------|--------|------------|
| 192.168.10.1    | 10.0.10.1       | 10   | R1     | 10.0.0.1   |
| 192.168.10.2    | 10.0.10.2       | 10   | R1     | 10.0.0.1   |
| 192.168.10.1    | 10.0.10.1       | 10   | R2     | 10.0.0.2   |
| 192.168.10.2    | 10.0.10.2       | 10   | R2     | 10.0.0.2   |
| 192.168.20.1    | 10.0.20.1       | 20   | R1     | 10.0.0.1   |
| 192.168.20.2    | 10.0.20.2       | 20   | R1     | 10.0.0.1   |
| 192.168.20.1    | 10.0.20.1       | 20   | R2     | 10.0.0.2   |
| 192.168.20.2    | 10.0.20.2       | 20   | R2     | 10.0.0.2   |

> 🔹 Även om översatta IP-adresser är identiska på båda sidor, fungerar kommunikationen eftersom varje router hanterar sin NAT-tabell lokalt och trafiken ruttas via separata gränssnitt (`10.0.0.1` och `10.0.0.2`).

---

Konfigurationsutdrag – Router R1

```bash
interface Gig0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.254 255.255.255.0
 ip nat inside

interface Gig0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.254 255.255.255.0
 ip nat inside

interface Gig0/1
 ip address 10.0.0.1 255.255.255.252
 ip nat outside

ip nat inside source static 192.168.10.1 10.0.10.1
ip nat inside source static 192.168.10.2 10.0.10.2
ip nat inside source static 192.168.20.1 10.0.20.1
ip nat inside source static 192.168.20.2 10.0.20.2

ip route 10.0.10.1 255.255.255.255 10.0.0.2
ip route 10.0.10.2 255.255.255.255 10.0.0.2
ip route 10.0.20.1 255.255.255.255 10.0.0.2
ip route 10.0.20.2 255.255.255.255 10.0.0.2

---

Konfigurationsutdrag – Router R2

```bash
interface Gig0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.254 255.255.255.0
 ip nat inside

interface Gig0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.254 255.255.255.0
 ip nat inside

interface Gig0/1
 ip address 10.0.0.2 255.255.255.252
 ip nat outside

ip nat inside source static 192.168.10.1 10.0.10.1
ip nat inside source static 192.168.10.2 10.0.10.2
ip nat inside source static 192.168.20.1 10.0.20.1
ip nat inside source static 192.168.20.2 10.0.20.2

ip route 10.0.10.1 255.255.255.255 10.0.0.1
ip route 10.0.10.2 255.255.255.255 10.0.0.1
ip route 10.0.20.1 255.255.255.255 10.0.0.1
ip route 10.0.20.2 255.255.255.255 10.0.0.1

---

Testresultat
Alla PC-enheter i VLAN 10 och VLAN 20 kan kommunicera över routrar via statisk NAT
Gateway-adresser är korrekt inställda per VLAN (192.168.10.254, 192.168.20.254)
Ping och traceroute fungerar mellan översatta adresser
Simulation Mode i Packet Tracer visar inte alltid ICMP-paketens väg, men manuell ping bekräftar funktion

---

Reflektion
Detta projekt gav mig djupare förståelse för:
IP-konflikter och hur man löser dem med NAT
VLAN-routing och subinterface-design
Felsökning i Packet Tracer och skillnaden mellan visuell simulering och faktisk funktion
Vikten av konfigurationshygien och dokumentation
Hur trunk- och accessportar samverkar för korrekt VLAN-segmentering

---

Filer i detta repo
topologi.png – Skärmdump av nätverket
konfig-router1.txt – Fullständig konfiguration för Router R1
konfig-router2.txt – Fullständig konfiguration för Router R2
NAT-VLAN-Lab.pdf – Dokumentation i PDF-format
