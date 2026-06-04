# Rapport — LAB 1 : Mise en place du lab Mobexler + snapshot CLEAN

## 1. Informations générales

| Élément | Valeur |
|---|---|
| Nom du lab | LAB 1 — Mise en place du lab |
| Environnement | Mobexler dans VirtualBox |
| Cible Android | Android Studio Emulator |
| Objectif principal | Préparer un environnement d’analyse mobile reproductible |


---

## 2. Objectifs du TP

Ce TP a pour objectif de mettre en place un environnement de laboratoire pour l’analyse mobile Android.

À la fin du TP, l’environnement doit permettre :

- de démarrer Mobexler sans erreur ;
- d’avoir un accès Internet dans Mobexler via l’interface NAT ;
- de communiquer avec une cible Android via un réseau de laboratoire Host-Only ;
- de vérifier la communication avec ADB ;
- de créer un snapshot propre appelé `CLEAN_BASELINE_TP1` ;
- de documenter les informations critiques pour garantir la reproductibilité.

---

## 3. Préparation de la machine virtuelle Mobexler

La machine virtuelle Mobexler a été importée à partir d’un fichier au format OVA.

Le format OVA permet d’importer une machine virtuelle déjà préparée, avec son système, ses outils et sa configuration de base.

### Vérification recommandée du fichier OVA

Commande PowerShell utilisée pour vérifier l’intégrité du fichier :

```powershell
Get-FileHash .\Mobexler.ova -Algorithm SHA256
```

Cette étape permet de vérifier que le fichier téléchargé n’est pas corrompu ou modifié.


## 4. Configuration réseau de Mobexler

Deux cartes réseau ont été configurées pour la machine virtuelle Mobexler.

| Adaptateur | Mode réseau | Rôle |
|---|---|---|
| Adapter 1 | NAT | Accès Internet depuis Mobexler |
| Adapter 2 | Host-Only Adapter | Communication avec l’hôte Windows et les cibles du lab |

### Explication

L’interface **NAT** permet à Mobexler d’accéder à Internet via la machine hôte.

L’interface **Host-Only** permet de créer un réseau privé entre Windows et Mobexler. Ce réseau est utilisé pour les communications de laboratoire, notamment avec ADB ou un proxy.

<img width="1022" height="613" alt="Capture d&#39;écran 2026-06-04 200556" src="https://github.com/user-attachments/assets/cbf0d18e-2fe8-4397-8e62-96e3b6260b67" />
<img width="1012" height="588" alt="Capture d&#39;écran 2026-06-04 200551" src="https://github.com/user-attachments/assets/94b1d45c-c71b-43c4-9161-1742164cacd0" />


## 5. Vérification des interfaces réseau dans Mobexler

La commande suivante a été utilisée dans Mobexler :

```bash
ip a
```

Résultat observé :

```text
enp0s8  : 192.168.56.103/24
enp0s17 : 10.0.2.15/24
docker0 : 172.17.0.1/16
```

### Analyse des interfaces

| Interface | Adresse IP | Rôle |
|---|---|---|
| `enp0s8` | `192.168.56.103/24` | Interface Host-Only |
| `enp0s17` | `10.0.2.15/24` | Interface NAT |
| `docker0` | `172.17.0.1/16` | Interface Docker interne |

L’interface `enp0s17` correspond au réseau NAT et permet l’accès à Internet.

L’interface `enp0s8` correspond au réseau Host-Only et permet la communication avec la machine Windows hôte.

<img width="1457" height="355" alt="Capture d&#39;écran 2026-06-04 200648" src="https://github.com/user-attachments/assets/65c1e119-d480-4377-96b1-6e999b8b77ac" />
<img width="1445" height="212" alt="Capture d&#39;écran 2026-06-04 200636" src="https://github.com/user-attachments/assets/4ac28e25-e23c-445c-97a1-c2042c5c093e" />


## 6. Vérification de l’accès Internet

La connectivité Internet a été testée avec la commande suivante :

```bash
ping 8.8.8.8
```

Résultat obtenu :

```text
2 packets transmitted, 2 received, 0% packet loss
```

Cela confirme que la machine Mobexler a bien accès à Internet via l’interface NAT.

### Résultat

| Test | Résultat |
|---|---|
| Ping vers `8.8.8.8` | OK |
| Perte de paquets | 0% |
| Internet via NAT | Validé |

<img width="1163" height="462" alt="Capture d&#39;écran 2026-06-04 200707" src="https://github.com/user-attachments/assets/9edf67de-a29f-4159-98ed-2d399b59e51b" />
<img width="1487" height="412" alt="Capture d&#39;écran 2026-06-04 200700" src="https://github.com/user-attachments/assets/d5821d93-b1d7-4551-a971-bb7a44929142" />


## 7. Vérification du réseau Host-Only

L’adresse Host-Only de Windows a été obtenue avec la commande suivante :

```powershell
ipconfig
```

Résultat observé côté Windows :

```text
Carte Ethernet Ethernet 2 :
Adresse IPv4 : 192.168.56.1
Masque       : 255.255.255.0
```

L’adresse Host-Only de Mobexler est :

```text
192.168.56.103
```

Les deux machines appartiennent donc au même réseau :

```text
192.168.56.0/24
```

### Test de connectivité

Depuis Windows, le ping vers Mobexler a fonctionné :

```powershell
ping 192.168.56.103
```

Résultat :

```text
Paquets : envoyés = 2, reçus = 2, perdus = 0
```

Cela valide la communication dans le sens :

```text
Windows → Mobexler
```

Le ping dans le sens inverse :

```text
Mobexler → Windows
```

était bloqué au départ, probablement à cause du pare-feu Windows. Des règles de pare-feu ont donc été ajoutées pour autoriser ICMP et le port ADB.

### Commandes utilisées pour le pare-feu Windows

```powershell
New-NetFirewallRule -DisplayName "Allow ICMPv4 HostOnly" -Protocol ICMPv4 -IcmpType 8 -Direction Inbound -Action Allow -Profile Private
```

```powershell
New-NetFirewallRule -DisplayName "ADB Lab 5037" -Direction Inbound -Protocol TCP -LocalPort 5037 -Action Allow -Profile Private
```

```powershell
New-NetFirewallRule -DisplayName "ADB Lab 5037 Any" -Direction Inbound -Protocol TCP -LocalPort 5037 -Action Allow -Profile Any
```

```powershell
New-NetFirewallRule -DisplayName "ADB Lab 5038 Any" -Direction Inbound -Protocol TCP -LocalPort 5038 -Action Allow -Profile Any
```


## 8. Création du snapshot CLEAN

Après validation du démarrage de Mobexler, de l’accès Internet et de la configuration réseau Host-Only, un snapshot propre doit être créé.

### Nom du snapshot

```text
CLEAN_BASELINE_TP1
```

### Description du snapshot

```text
Import OK, NAT + Host-Only OK, Internet OK, prêt ADB.
```

### Rôle du snapshot

Le snapshot permet de revenir rapidement à un état stable et propre avant les prochains travaux pratiques. Il est utile si une mauvaise configuration, un certificat, un proxy ou un outil modifie l’environnement.

<img width="506" height="442" alt="Capture d&#39;écran 2026-06-04 200900" src="https://github.com/user-attachments/assets/0a7da51c-bb32-4576-88bd-b62af83fbc17" />


## 9. Préparation de la cible Android

Pour ce TP, l’option choisie est :

```text
Option B — Android Studio Emulator
```

L’émulateur Android Studio a été lancé sur la machine hôte Windows.

### Vérification de l’émulateur côté Windows

Dans PowerShell Windows, depuis le dossier `platform-tools`, la commande suivante a été exécutée :

```powershell
.\adb.exe devices
```

Résultat obtenu :

```text
List of devices attached
emulator-5554   device
```

Cela confirme que Windows détecte correctement l’émulateur Android Studio.

<img width="782" height="151" alt="image" src="https://github.com/user-attachments/assets/49219cc1-f723-4735-8dd9-b6a923de854d" />


## 10. Problème rencontré : conflit de versions ADB

Un problème de compatibilité a été rencontré entre ADB dans Mobexler et ADB dans Windows.

### Erreur observée

```text
adb server version (41) doesn't match this client (39)
ADB server didn't ACK
```

### Cause

Mobexler utilisait initialement une ancienne version ADB :

```text
Android Debug Bridge version 1.0.39
```

Alors que Windows utilisait une version plus récente :

```text
Android Debug Bridge version 1.0.41
```

Cette différence de version empêchait Mobexler d’utiliser correctement le serveur ADB Windows.

---

## 11. Correction : mise à jour de ADB dans Mobexler

Les platform-tools Android récents ont été installés dans Mobexler.

### Commandes utilisées

```bash
cd ~
mkdir -p ~/android-sdk
wget -O platform-tools-latest-linux.zip https://dl.google.com/android/repository/platform-tools-latest-linux.zip
unzip -o platform-tools-latest-linux.zip -d ~/android-sdk
```

Puis le chemin du nouveau ADB a été ajouté au `PATH` :

```bash
export PATH=$HOME/android-sdk/platform-tools:$PATH
hash -r
```

Pour rendre le changement permanent :

```bash
echo 'export PATH=$HOME/android-sdk/platform-tools:$PATH' >> ~/.bashrc
source ~/.bashrc
```

### Vérification

Commandes utilisées :

```bash
adb version
which adb
```

Résultat obtenu :

```text
Android Debug Bridge version 1.0.41
Version 37.0.0-14910828
Installed as /home/mobexler/android-sdk/platform-tools/adb
Running on Linux 4.19.0-21-amd64 (x86_64)

/home/mobexler/android-sdk/platform-tools/adb
```

La version ADB de Mobexler est donc maintenant compatible avec celle de Windows.
<img width="946" height="143" alt="image" src="https://github.com/user-attachments/assets/0d5f2862-4950-4a3c-b1ed-c45cbef0cdf4" />



## 12. Utilisation du serveur ADB Windows sur le port 5038

Le port `5037` était déjà utilisé par Android Studio ou un processus ADB existant.

L’erreur observée était :

```text
cannot bind to 0.0.0.0:5037
Une seule utilisation de chaque adresse de socket est habituellement autorisée.
```

Pour éviter ce conflit, le serveur ADB Windows a été lancé sur le port `5038`.

### Commande utilisée côté Windows

Dans PowerShell administrateur :

```powershell
cd $env:LOCALAPPDATA\Android\Sdk\platform-tools
.\adb.exe -a -P 5038 nodaemon server
```

Cette fenêtre doit rester ouverte pendant l’utilisation d’ADB depuis Mobexler.

---

## 13. Connexion ADB depuis Mobexler vers l’émulateur Android Studio

Depuis Mobexler, la connexion au serveur ADB Windows a été testée avec :

```bash
adb -H 192.168.56.1 -P 5038 devices
```

Résultat obtenu :

```text
List of devices attached
emulator-5554    device
```

Cela valide la communication suivante :

```text
Mobexler → Windows Host-Only → ADB Windows → Android Studio Emulator
```
<img width="872" height="137" alt="image" src="https://github.com/user-attachments/assets/e582b4be-966e-49b1-ad97-d3d927d272f9" />



## 14. Vérification des propriétés Android

Depuis Mobexler, les propriétés de l’émulateur Android ont été récupérées avec les commandes suivantes :

```bash
adb -H 192.168.56.1 -P 5038 shell getprop ro.product.model
adb -H 192.168.56.1 -P 5038 shell getprop ro.build.version.release
```

Résultat obtenu :

```text
sdk_gphone16k_x86_64
17
```

### Interprétation

| Propriété | Valeur |
|---|---|
| Modèle Android | `sdk_gphone16k_x86_64` |
| Version Android | `17` |
| Device ADB | `emulator-5554` |
| État | `device` |

<img width="943" height="148" alt="image" src="https://github.com/user-attachments/assets/c4a2b7c9-33d4-406c-b01a-ef173ef03eca" />


## 15. Tableau récapitulatif final

| Élément | Valeur |
|---|---|
| VM utilisée | Mobexler |
| Hyperviseur | VirtualBox |
| Interface NAT | `enp0s17` |
| IP NAT Mobexler | `10.0.2.15/24` |
| Interface Host-Only | `enp0s8` |
| IP Host-Only Mobexler | `192.168.56.103/24` |
| IP Host-Only Windows | `192.168.56.1/24` |
| Réseau Host-Only | `192.168.56.0/24` |
| Accès Internet | OK |
| Ping `8.8.8.8` | OK, 0% perte |
| Cible Android | Android Studio Emulator |
| Identifiant ADB | `emulator-5554` |
| État ADB | `device` |
| Port ADB utilisé | `5038` |
| Version ADB Mobexler | `1.0.41` |
| Chemin ADB Mobexler | `/home/mobexler/android-sdk/platform-tools/adb` |
| Modèle Android | `sdk_gphone16k_x86_64` |
| Version Android | `17` |
| Snapshot | `CLEAN_BASELINE_TP1` |

---

## 16. Commandes importantes utilisées

### Dans Mobexler

```bash
ip a
```

```bash
ping 8.8.8.8
```

```bash
adb version
which adb
```

```bash
adb -H 192.168.56.1 -P 5038 devices
```

```bash
adb -H 192.168.56.1 -P 5038 shell getprop ro.product.model
adb -H 192.168.56.1 -P 5038 shell getprop ro.build.version.release
```

### Dans Windows PowerShell

```powershell
ipconfig
```

```powershell
ping 192.168.56.103
```

```powershell
cd $env:LOCALAPPDATA\Android\Sdk\platform-tools
.\adb.exe devices
```

```powershell
.\adb.exe -a -P 5038 nodaemon server
```

---



## 17. Conclusion

Dans ce TP, l’environnement Mobexler a été mis en place avec succès. La machine virtuelle a été configurée avec deux interfaces réseau : une interface NAT pour l’accès Internet et une interface Host-Only pour la communication avec l’hôte Windows et la cible Android.

Les tests réseau ont confirmé que Mobexler dispose d’une adresse NAT `10.0.2.15/24` et d’une adresse Host-Only `192.168.56.103/24`. L’accès Internet via NAT a été validé par un ping vers `8.8.8.8` avec 0% de perte.

La cible Android a été préparée avec Android Studio Emulator. Windows détecte correctement l’émulateur avec l’identifiant `emulator-5554`. Après correction d’un conflit de versions ADB et utilisation du port `5038`, Mobexler a pu communiquer avec l’émulateur via ADB.

La commande `adb -H 192.168.56.1 -P 5038 devices` a confirmé que la cible Android est bien détectée avec l’état `device`. Le modèle Android `sdk_gphone16k_x86_64` et la version Android `17` ont également été vérifiés.

Le lab est donc validé et l’environnement est prêt pour les prochains travaux pratiques d’analyse mobile.
