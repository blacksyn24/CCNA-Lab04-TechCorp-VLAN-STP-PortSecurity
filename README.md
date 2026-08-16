# 🏢 Lab 4 — TechCorp Bénin : VLAN, STP, Port Security & DHCP

![Cisco](https://img.shields.io/badge/Cisco-CCNA2-blue?style=for-the-badge&logo=cisco&logoColor=white)
![PacketTracer](https://img.shields.io/badge/Packet%20Tracer-8.x-orange?style=for-the-badge&logo=cisco)
![Status](https://img.shields.io/badge/Status-✅%20Completed-brightgreen?style=for-the-badge)
![Lab](https://img.shields.io/badge/Lab%20Series-Lab%204-purple?style=for-the-badge)
![Topic](https://img.shields.io/badge/Topic-VLAN%20%2F%20STP%20%2F%20Security-red?style=for-the-badge)

---

## 📋 Description

Quatrième lab de ma série CCNA1/CCNA2. Scénario d'entreprise complet pour
**TechCorp Bénin** à Cotonou : segmentation en 4 VLAN, routage inter-VLAN
via Router-on-a-Stick, sécurisation des ports d'accès, protection contre
les boucles réseau (STP), et distribution automatique d'adresses IP (DHCP).

### Objectifs
- ✅ Segmenter le réseau en **4 VLAN** (Direction, Comptabilité, Commercial, IT)
- ✅ Configurer le **routage inter-VLAN** (Router-on-a-Stick)
- ✅ Activer **Rapid PVST+** avec Root Bridge défini
- ✅ Sécuriser les ports d'accès avec **Port Security** (sticky MAC)
- ✅ Configurer **DHCP** pour chaque VLAN
- ✅ Vérifier la connectivité intra et inter-VLAN

---

## 📖 Théorie

### Port Security — pourquoi c'est important

Sans port security
→ N'importe quel appareil branché sur un port access est accepté
→ Risque : intrusion physique, appareil non autorisé

Avec port security (sticky MAC)
→ Le switch apprend automatiquement la 1ère adresse MAC vue
→ Toute autre MAC sur ce port = violation
→ Mode "restrict" : bloque le trafic mais garde le port actif (log généré)


### Root Bridge STP

Le Root Bridge est le point de référence du réseau pour STP.
→ spanning-tree vlan X root primary : force ce switch comme racine
→ spanning-tree vlan X root secondary : backup si le primary tombe
→ Sans ça, l'élection se fait automatiquement (souvent pas optimal)


---

## 🖥️ Équipements

| Équipement | Modèle | Nom | Rôle |
|-----------|--------|-----|------|
| 🔀 Routeur | 2911 | Router0 | Router-on-a-Stick + DHCP |
| 🔀 Switch | 2960-24TT | S2 | Distribution + Root Bridge |
| 🔀 Switch | 2960-24TT | S1 | Accès Direction/Comptabilité |
| 🔀 Switch | 2960-24TT | S3 | Accès Commercial/IT |
| 💻 PC | PC-PT | PC1-PC8 | Utilisateurs (2 par VLAN) |

---

## 🗺️ Topologie
                   Router0
                      |
                   Gi0/0
                      |
                   Gi0/1
                     S2 ← Root Bridge STP
                Fa0/1     Fa0/2
                /             \
              S1               S3
          /       \        /       \
      VLAN10    VLAN20  VLAN30   VLAN40
      PC1,2     PC3,4   PC5,6    PC7,8

<img width="1920" height="1080" alt="Topologie" src="https://github.com/user-attachments/assets/1f367391-a62d-49fb-8f1c-4c06656cc98f" />


---

## 📊 Plan d'adressage

| VLAN | Nom | Réseau | Passerelle | Pool DHCP | Switch |
|------|-----|--------|------------|-----------|--------|
| 10 | DIRECTION | 192.168.10.0/24 | 192.168.10.1 | .11→.50 | S1 |
| 20 | COMPTABILITE | 192.168.20.0/24 | 192.168.20.1 | .11→.50 | S1 |
| 30 | COMMERCIAL | 192.168.30.0/24 | 192.168.30.1 | .11→.50 | S3 |
| 40 | IT | 192.168.40.0/24 | 192.168.40.1 | .11→.50 | S3 |
| 99 | MANAGEMENT | — | — | — | tous |

---

## 📡 Messages VLAN Packet Tracer

### VLAN 10 — DIRECTION

Switch : S1
Ports : Fa0/2, Fa0/3
PC connectés : PC1, PC2
Réseau : 192.168.10.0/24
Passerelle : 192.168.10.1


### VLAN 20 — COMPTABILITE

Switch : S1
Ports : Fa0/4, Fa0/5
PC connectés : PC3, PC4
Réseau : 192.168.20.0/24
Passerelle : 192.168.20.1


### VLAN 30 — COMMERCIAL

Switch : S3
Ports : Fa0/2, Fa0/3
PC connectés : PC5, PC6
Réseau : 192.168.30.0/24
Passerelle : 192.168.30.1


### VLAN 40 — IT

Switch : S3
Ports : Fa0/4, Fa0/5
PC connectés : PC7, PC8
Réseau : 192.168.40.0/24
Passerelle : 192.168.40.1


### Router0 — Sous-interfaces

Interface physique : Gi0/0 (trunk vers S2)

Gi0/0.10 (VLAN 10) → IP 192.168.10.1/24 → dot1Q 10
Gi0/0.20 (VLAN 20) → IP 192.168.20.1/24 → dot1Q 20
Gi0/0.30 (VLAN 30) → IP 192.168.30.1/24 → dot1Q 30
Gi0/0.40 (VLAN 40) → IP 192.168.40.1/24 → dot1Q 40
Gi0/0.99 (VLAN 99) → aucune IP → dot1Q 99 native


---

## ⚙️ Configuration complète

### 🔧 Task 1 — Router0 (Router-on-a-Stick + DHCP)

```cisco
enable
configure terminal
hostname Router0
interface gigabitEthernet0/0
no shutdown
exit
interface gigabitEthernet0/0.10
encapsulation dot1Q 10
ip address 192.168.10.1 255.255.255.0
exit
interface gigabitEthernet0/0.20
encapsulation dot1Q 20
ip address 192.168.20.1 255.255.255.0
exit
interface gigabitEthernet0/0.30
encapsulation dot1Q 30
ip address 192.168.30.1 255.255.255.0
exit
interface gigabitEthernet0/0.40
encapsulation dot1Q 40
ip address 192.168.40.1 255.255.255.0
exit
interface gigabitEthernet0/0.99
encapsulation dot1Q 99 native
exit
ip dhcp excluded-address 192.168.10.1 192.168.10.10
ip dhcp pool DIRECTION
network 192.168.10.0 255.255.255.0
default-router 192.168.10.1
dns-server 8.8.8.8
exit
ip dhcp excluded-address 192.168.20.1 192.168.20.10
ip dhcp pool COMPTABILITE
network 192.168.20.0 255.255.255.0
default-router 192.168.20.1
dns-server 8.8.8.8
exit
ip dhcp excluded-address 192.168.30.1 192.168.30.10
ip dhcp pool COMMERCIAL
network 192.168.30.0 255.255.255.0
default-router 192.168.30.1
dns-server 8.8.8.8
exit
ip dhcp excluded-address 192.168.40.1 192.168.40.10
ip dhcp pool IT
network 192.168.40.0 255.255.255.0
default-router 192.168.40.1
dns-server 8.8.8.8
exit
end
write
```

---

### 🔧 Task 2 — S2 (Distribution + Root Bridge)

```cisco
enable
configure terminal
hostname S2
vlan 10
name DIRECTION
exit
vlan 20
name COMPTABILITE
exit
vlan 30
name COMMERCIAL
exit
vlan 40
name IT
exit
vlan 99
name MANAGEMENT
exit
spanning-tree mode rapid-pvst
spanning-tree vlan 10,20,30,40,99 root primary
interface gigabitEthernet0/1
switchport mode trunk
switchport trunk native vlan 99
switchport trunk allowed vlan 10,20,30,40,99
exit
interface fastEthernet0/1
switchport mode trunk
switchport trunk native vlan 99
switchport trunk allowed vlan 10,20,99
exit
interface fastEthernet0/2
switchport mode trunk
switchport trunk native vlan 99
switchport trunk allowed vlan 30,40,99
exit
end
write
```

---

### 🔧 Task 3 — S1 (Accès Direction/Comptabilité)

```cisco
enable
configure terminal
hostname S1
vlan 10
name DIRECTION
exit
vlan 20
name COMPTABILITE
exit
vlan 99
name MANAGEMENT
exit
spanning-tree mode rapid-pvst
spanning-tree vlan 10,20,30,40,99 root secondary
interface fastEthernet0/1
switchport mode trunk
switchport trunk native vlan 99
switchport trunk allowed vlan 10,20,99
exit
interface fastEthernet0/2
switchport mode access
switchport access vlan 10
switchport port-security
switchport port-security maximum 1
switchport port-security violation restrict
switchport port-security mac-address sticky
spanning-tree portfast
exit
interface fastEthernet0/3
switchport mode access
switchport access vlan 10
switchport port-security
switchport port-security maximum 1
switchport port-security violation restrict
switchport port-security mac-address sticky
spanning-tree portfast
exit
interface fastEthernet0/4
switchport mode access
switchport access vlan 20
switchport port-security
switchport port-security maximum 1
switchport port-security violation restrict
switchport port-security mac-address sticky
spanning-tree portfast
exit
interface fastEthernet0/5
switchport mode access
switchport access vlan 20
switchport port-security
switchport port-security maximum 1
switchport port-security violation restrict
switchport port-security mac-address sticky
spanning-tree portfast
exit
end
write
```

---

### 🔧 Task 4 — S3 (Accès Commercial/IT)

```cisco
enable
configure terminal
hostname S3
vlan 30
name COMMERCIAL
exit
vlan 40
name IT
exit
vlan 99
name MANAGEMENT
exit
spanning-tree mode rapid-pvst
interface fastEthernet0/1
switchport mode trunk
switchport trunk native vlan 99
switchport trunk allowed vlan 30,40,99
exit
interface fastEthernet0/2
switchport mode access
switchport access vlan 30
switchport port-security
switchport port-security maximum 1
switchport port-security violation restrict
switchport port-security mac-address sticky
spanning-tree portfast
exit
interface fastEthernet0/3
switchport mode access
switchport access vlan 30
switchport port-security
switchport port-security maximum 1
switchport port-security violation restrict
switchport port-security mac-address sticky
spanning-tree portfast
exit
interface fastEthernet0/4
switchport mode access
switchport access vlan 40
switchport port-security
switchport port-security maximum 1
switchport port-security violation restrict
switchport port-security mac-address sticky
spanning-tree portfast
exit
interface fastEthernet0/5
switchport mode access
switchport access vlan 40
switchport port-security
switchport port-security maximum 1
switchport port-security violation restrict
switchport port-security mac-address sticky
spanning-tree portfast
exit
end
write
```

---

## 🧪 Tests finaux

```cisco
S2# show vlan brief                       ✅ 4 VLAN créés
S2# show interfaces trunk                 ✅ 3 trunks actifs
S2# show spanning-tree                    ✅ S2 = Root Bridge
S1# show port-security interface fa0/2    ✅ sticky MAC appris
Router0# show ip dhcp binding             ✅ baux distribués
PC1 → ping 192.168.10.1                   ✅ passerelle DIRECTION
PC1 → ping 192.168.10.12                  ✅ PC1 → PC2 (même VLAN)
PC1 → ping 192.168.30.11                  ✅ PC1 → PC5 (inter-VLAN)
```

---

## 💡 Points clés

| 🔑 Commande | 📖 Rôle |
|-------------|---------|
| `encapsulation dot1Q 10` | Associe une sous-interface à un VLAN |
| `spanning-tree vlan X root primary` | Force ce switch comme racine STP |
| `switchport port-security mac-address sticky` | Apprend et verrouille la 1ère MAC vue |
| `switchport port-security violation restrict` | Bloque le trafic sans désactiver le port |
| `switchport trunk native vlan 99` | Définit le VLAN natif du lien trunk |

---

## 📊 Comparatif Labs

| | Lab 2 (VLSM) | Lab 4 (TechCorp) |
|---|---|---|
| **Focus** | Adressage IP | VLAN + Sécurité + STP |
| **Niveau** | CCNA1 | CCNA2 |
| **Switches** | 2 | 3 |
| **VLAN** | Aucun | 4 + management |
| **Sécurité** | Aucune | Port Security |

---

## 👨‍💻 Auteur

**Urbain Sedami Landjidé**
🎓 Étudiant en 2ème année — Licence Professionnelle
📡 Réseaux Informatique Mobilité Sécurité (RMS)
🏫 Cisco Networking Academy (en partenariat avec Pigier Bénin)
📍 Cotonou, Bénin 🇧🇯

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connecter-blue?style=flat-square&logo=linkedin)](https://www.linkedin.com/in/urbain-sedami-landjide-9b49043a8/)

---

## 📄 Licence

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
