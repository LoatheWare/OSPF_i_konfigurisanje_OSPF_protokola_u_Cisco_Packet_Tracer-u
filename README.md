# OSPF i konfigurisanje OSPF protokola u Cisco Packet Tracer-u
## Šta je ovo?
Ovo je vežba za administratore računarskih mreža koju sam radio tokom mog školovanja. Nalazi se jedan .pkt fajl koji se preuzme i to će Vam biti početno stanje. Zadak Vam je dat, kao i uputstvo za njegovo rešavanje. Ukoliko Vam nešto nije jasno tokom rešavanja ovih zadataka obratite mi se direktnom porukom na aplikaciji LinkedIn, www.linkedin.com/in/nikola-karanović-397185390

## Pitanja i odgovori — OSPF (Open Shortest Path First)

### 1. Šta je OSPF?
**OSPF** (Open Shortest Path First) je **link-state** protokol za dinamičko rutiranje, koji se koristi unutar jednog autonomnog sistema (**IGP** — Interior Gateway Protocol). OSPF je **otvoren standard** (definisan u **RFC 2328** za OSPFv2/IPv4), za razliku od proizvođački-zavisnih protokola kao što je Cisco-ov EIGRP. Za izračunavanje najkraće (najbolje) putanje do svake mreže koristi **Dijkstra-in algoritam** (SPF — Shortest Path First).

---

### 2. Koje su osnovne prednosti OSPF protokola?
OSPF nam omogućava:
1. **Brzu konvergenciju** — promene u mreži se brzo detektuju i propagiraju, jer svaki ruter ima kompletnu topološku mapu mreže (ili dela mreže — area).
2. **Skalabilnost** — podelom mreže na **area-e** (oblasti) smanjuje se veličina tabela i opterećenje rutera u velikim mrežama.
3. **Efikasno korišćenje propusnog opsega** — ažuriranja se šalju samo kada dođe do promene u topologiji (ne periodično u celosti), a koriste se i multicast adrese umesto brodkasta.
4. **Podršku za VLSM i CIDR** — OSPF šalje informaciju o mreži **i** masci, pa podržava rutiranje bez klasa (classless routing).
5. **Load balansiranje** — automatski balansira saobraćaj preko više putanja istog troška (*equal-cost multipath*).

---

### 3. Koji su osnovni koncepti/komponente OSPF-a?
- **Router ID (RID)** — jedinstveni identifikator rutera u OSPF domenu (po formatu liči na IP adresu, ali to nije obavezno IP adresa interfejsa).
- **Area (oblast)** — logička grupa rutera i mreža; OSPF mreže se obavezno organizuju oko **Area 0** (backbone area), na koju se moraju direktno ili indirektno povezivati sve ostale area-e.
- **Neighbor (sused)** — ruter direktno povezan i u OSPF odnosu sa drugim ruterom, sa kojim razmenjuje rutirajuće informacije.
- **LSA (Link-State Advertisement)** — poruka kojom ruter oglašava informacije o svojim linkovima (mrežama) ostalim ruterima.
- **LSDB (Link-State Database)** — baza svih LSA poruka koje ruter poseduje; svi ruteri u istoj area-i imaju **identičnu** LSDB.
- **Cost (metrika)** — OSPF metrika, izračunava se na osnovu propusnog opsega interfejsa (manji cost = bolja, brža putanja).

---

### 4. Koja su OSPF stanja (states) prilikom formiranja susedstva?
Ruteri prolaze kroz sledeća stanja dok formiraju susedstvo (neighbor adjacency):
1. **Down** — nije primljen nijedan Hello paket.
2. **Init** — primljen je Hello paket, ali bez potvrde dvosmerne komunikacije.
3. **2-Way** — ruteri se međusobno "vide" (svaki vidi sebe u Hello paketu drugog), formira se susedstvo (*neighbor*).
4. **ExStart** — pregovara se ko će biti master/slave u razmeni baza.
5. **Exchange** — razmenjuju se DBD (Database Description) paketi sa sažetkom LSA zapisa.
6. **Loading** — razmenjuju se kompletni LSA zapisi koji nedostaju.
7. **Full** — ruteri imaju identičnu LSDB; susedstvo je potpuno formirano (*adjacency*).

---

### 5. Koji su uslovi da bi dva rutera postala OSPF susedi?
Da bi se formiralo susedstvo, sledeći parametri moraju biti identični na oba rutera:
- **Area broj** na zajedničkom interfejsu
- **Hello i Dead interval** (vreme slanja Hello paketa i vreme čekanja pre nego što se sused proglasi nedostupnim)
- **Subnet maska** (na istoj mreži)
- **Autentifikacija** (ako je konfigurisana, mora biti ista na oba rutera)
- **OSPF tip mreže** (npr. broadcast, point-to-point) na interfejsu

---

### 6. Kako se konfiguriše osnovni OSPF na Cisco ruteru?
Osnovna konfiguracija OSPF procesa i oglašavanje mreža:

"Router(config)# router ospf 1"

"Router(config-router)# network 192.168.1.0 0.0.0.255 area 0"

"Router(config-router)# network 192.168.2.0 0.0.0.255 area 0"

*(`1` je proces ID, koji je lokalno značajan i ne mora biti isti na svim ruterima; `0.0.0.255` je wildcard maska, a `area 0` je backbone area.)*

---

### 7. Kako se ručno definiše Router ID?
Router ID se po default-u bira automatski (najveća IP adresa loopback interfejsa, ili najveća IP adresa aktivnog interfejsa ako loopback ne postoji), ali se preporučuje da se definiše ručno:

"Router(config)# router ospf 1"

"Router(config-router)# router-id 1.1.1.1"

*(Nakon promene Router ID-a, potrebno je restartovati OSPF proces komandom `clear ip ospf process` da bi promena stupila na snagu.)*

---

### 8. Kako se podešava passive interfejs u OSPF-u?
Kada interfejs treba da bude **u OSPF mreži (oglašen)**, ali na njemu **ne treba slati Hello pakete** (npr. interfejs prema krajnjim korisnicima, a ne prema drugom ruteru), koristi se:

"Router(config)# router ospf 1"

"Router(config-router)# passive-interface GigabitEthernet0/0"

Ovo povećava sigurnost (sprečava neovlašćeno formiranje OSPF susedstva na tom interfejsu) i smanjuje nepotreban saobraćaj.

---

### 9. Koje su komande za proveru OSPF konfiguracije i stanja?
Najvažnije komande za verifikaciju OSPF-a:

"Router# show ip ospf neighbor"

"Router# show ip ospf interface"

"Router# show ip protocols"

"Router# show ip route ospf"

"Router# show ip ospf database"

- `show ip ospf neighbor` — prikazuje listu OSPF suseda, njihovo stanje (state) i Router ID.
- `show ip ospf interface` — prikazuje detalje o OSPF konfiguraciji na pojedinačnom interfejsu (area, cost, hello/dead interval).
- `show ip protocols` — prikazuje opšte informacije o svim aktivnim protokolima rutiranja, uključujući OSPF.
- `show ip route ospf` — prikazuje samo rute naučene putem OSPF-a u routing tabeli (oznaka `O` u tabeli rutiranja).

---

### 10. Kako OSPF bira Designated Router (DR) i Backup Designated Router (BDR)?
Na **broadcast** i **non-broadcast multi-access** mrežama (npr. Ethernet segment sa više rutera), OSPF bira **DR** i **BDR** da bi se smanjio broj adjacency veza (svi ruteri formiraju Full susedstvo samo sa DR i BDR, a međusobno samo 2-Way). Izbor se vrši na osnovu:
1. **OSPF prioriteta** interfejsa (veći prioritet pobeđuje; default je 1, a prioritet 0 znači da ruter **ne može** postati DR/BDR).
2. Ako su prioriteti isti, pobeđuje **veći Router ID**.

---

### 11. Šta je OSPF cost i kako se računa?
**Cost** je OSPF metrika i računa se po formuli:

"Cost = Referentni propusni opseg (default 100 000 Kbps) / Propusni opseg interfejsa (Kbps)"

Ukupan cost putanje do neke mreže je **zbir cost-ova** svih izlaznih interfejsa na putu do odredišta. Ruta sa **manjim** ukupnim cost-om se bira kao bolja. Cost se može i ručno podesiti na interfejsu:

"Router(config-if)# ip ospf cost 50"

---

### 12. Koja je razlika između OSPFv2 i OSPFv3?
- **OSPFv2** — koristi se za rutiranje **IPv4** mreža (RFC 2328).
- **OSPFv3** — koristi se za rutiranje **IPv6** mreža (RFC 5340), sa izmenjenim formatom paketa i konfiguracijom (npr. `ipv6 router ospf` umesto `router ospf`), ali sa istom osnovnom logikom rada (link-state, Dijkstra algoritam, area-e, DR/BDR izbor).

---
