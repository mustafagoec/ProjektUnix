# Video 01: Solaris, OpenIndiana und Rocky Linux in VirtualBox

Leitfaden für Test, Installation und Aufnahme mit OBS Studio.
Systeme: **Oracle Solaris 11.4**, **OpenIndiana Hipster** (illumos) und **Rocky Linux 9/10** (RHEL-Klon).

---

## 1. Vorbereitung auf dem Windows-Host

- **VirtualBox 7.x** aktuell halten, das passende **Extension Pack** und die **Guest Additions ISO** sind dabei.
- **Hyper-V / Speicher-Integrität prüfen:** Ist Hyper-V, WSL2 oder „Kernisolierung → Speicher-Integrität“ aktiv, läuft VirtualBox nur über die Windows-Hypervisor-Schnittstelle. Erkennbar an der **grünen Schildkröte** unten rechts im VM-Fenster. Dann ist alles spürbar langsamer. Für den Drehtag ggf. abschalten (`bcdedit /set hypervisorlaunchtype off` als Admin, danach Neustart; rückgängig mit `... auto`). Achtung: WSL2/Docker Desktop laufen dann nicht.
- **Speicherplatz:** ca. 100 GB frei für drei VMs plus Aufnahmen.
- **ISOs prüfen** (Prüfsumme, falls angegeben): `certutil -hashfile datei.iso SHA256`

### Wichtig beim „Neue VM“-Assistenten
VirtualBox 7 bietet eine **unbeaufsichtigte Installation** an. Für das Video unbedingt
**„Unbeaufsichtigte Installation überspringen“ anhaken**, sonst installiert VirtualBox im Hintergrund und man sieht den Installer nicht.

---

## 2. Empfohlene VM-Einstellungen

| Einstellung | Oracle Solaris 11.4 | OpenIndiana Hipster | Rocky Linux 9 / 10 |
|---|---|---|---|
| Typ / Version | Solaris / Oracle Solaris 11 (64-bit) | Solaris / OpenSolaris / Illumos / OpenIndiana (64-bit) | Linux / Red Hat (64-bit) |
| RAM | 4 GB (min. 2 GB) | 4 GB (min. 2 GB, Live-GUI braucht mehr) | 4 GB (min. 2 GB) |
| CPUs | 2 | 2 | 2 |
| I/O-APIC | an | an | an |
| EFI | **aus** (BIOS, am wenigsten Ärger) | **aus** | an oder aus, beides geht |
| Festplatte | 30 GB VDI, dynamisch | 30 GB VDI, dynamisch | 25 GB VDI, dynamisch |
| Controller | SATA (AHCI) | SATA (AHCI) | SATA (AHCI) |
| Grafik | **VBoxVGA**, 128 MB, 3D aus | **VBoxVGA**, 128 MB, 3D aus | **VMSVGA**, 128 MB, 3D aus |
| Netzwerk | NAT, Intel PRO/1000 MT Desktop | NAT, Intel PRO/1000 MT Desktop | NAT, Intel PRO/1000 oder virtio-net |
| Audio | aus | aus (oder Intel HD Audio) | an, falls benötigt |
| Gasterweiterungen | `VBoxSolarisAdditions.pkg` von der Guest-Additions-ISO | oft schon im Image, sonst wie Solaris | `VBoxLinuxAdditions.run` von der ISO |

Hinweise:
- **Netzwerk Intel PRO/1000** ist bei Solaris/illumos der sichere Treiber (`e1000g`). Virtio wird dort nicht zuverlässig erkannt.
- **Rocky Linux 10** setzt eine CPU mit **x86-64-v3** (AVX2) voraus. Läuft VirtualBox im Hyper-V-Modus (Schildkröte), werden diese CPU-Features manchmal nicht durchgereicht und der Installer startet nicht. Dann Rocky 9 nehmen oder Hyper-V abschalten.
- Unter *Anzeige → Bildschirm* die **Skalierung auf 100 %** lassen, damit OBS ein scharfes Bild bekommt.

---

## 3. Sinnvolle Test-Reihenfolge (vor der Aufnahme)

1. **Rocky Linux zuerst.** Bekanntes Terrain, prüft, ob VirtualBox, Netzwerk und Grafik auf dem Host sauber laufen. Dauer ca. 15 Minuten.
2. **OpenIndiana.** Live-Desktop (MATE) mit grafischem Installer, gut zum Kennenlernen der Solaris-Welt.
3. **Oracle Solaris 11.4.** Text-Installer, am meisten Stolpersteine, daher zuletzt, wenn der Rest läuft.
4. Nach jeder erfolgreichen Installation: **Snapshot „frisch installiert“** anlegen. Für die Aufnahme kann man so später ohne Neuinstallation jede Szene wiederholen.
5. Optional: Vor der Installation zusätzlich einen **Snapshot „leer“** anlegen, damit man die Installation selbst mehrfach drehen kann.

### Test-Checkliste je System
- [ ] Bootet die ISO, Installer erscheint
- [ ] Tastatur auf Deutsch eingestellt
- [ ] Installation läuft durch, Neustart ohne ISO
- [ ] Netzwerk: `ping -c 3 8.8.8.8` bzw. unter Solaris `ping 8.8.8.8`
- [ ] Gasterweiterungen installiert, Auflösung 1920×1080 möglich
- [ ] Demo-Befehle für das Video funktionieren (siehe Abschnitt 6)

---

## 4. Typische Stolpersteine

### Oracle Solaris 11.4
- **ISO-Wahl:** Für VirtualBox die **Text-Installer-ISO** (`sol-11_4-text-x86.iso`, bei der CBE-Version entsprechend benannt) nehmen. Die *Automated-Install*-ISO braucht einen AI-Server. Eine Live-Desktop-ISO gibt es für 11.4 nicht mehr.
- **Funktionstasten:** Der Text-Installer arbeitet mit F2 (Weiter), F3 (Zurück) usw. Fängt Windows oder das Notebook die F-Tasten ab, funktioniert **Esc und dann 2** (bzw. Esc 3, …) genauso.
- **Tastatur:** Die Tastaturbelegung wird beim Booten der ISO abgefragt (Liste mit Nummern, „German“ wählen). Später: `kbd -s`.
- **Netzwerk:** im Installer „Automatisch“ wählen, das ist am einfachsten mit NAT.
- **Benutzer und root:** root ist eine **Rolle**, direktes Login als root geht nicht. Mit dem angelegten Benutzer anmelden und `su -` (root-Passwort) oder `sudo` verwenden.
- **Nach der Installation:** ISO auswerfen bzw. Bootreihenfolge prüfen, sonst startet der Installer erneut.
- **Kein Desktop nach der Installation:** Der Text-Installer installiert nur ein Server-System. Desktop nachinstallieren mit
  `sudo pkg install solaris-desktop` und neu starten. Das lädt rund 1 GB aus dem Oracle-Repository (bei der CBE-Version ist `pkg.oracle.com/solaris/release` schon eingetragen). **Vor dem Drehtag erledigen**, das dauert.
- **Gasterweiterungen:** Geräte → Gasterweiterungen einlegen, dann als root:
  `pkgadd -d /media/VBOXADDITIONS_*/VBoxSolarisAdditions.pkg all` und neu starten.
- **Langsamer erster Boot:** SMF importiert beim ersten Start alle Service-Manifeste, das kann einige Minuten dauern. Nicht abbrechen.

### OpenIndiana Hipster
- **ISO-Wahl:** Die **GUI-ISO (Live, MATE)** ist für das Video am schönsten. Die Text- oder Minimal-ISO geht auch, ist aber weniger anschaulich.
- **Beim Booten** fragt der Live-Modus nach Tastaturbelegung und Sprache in einer Textabfrage. Nicht verpassen, sonst bleibt US-Layout.
- **Grafik:** Bei schwarzem Bildschirm oder Flackern auf **VBoxVGA** umstellen und 3D aus lassen.
- **Installation:** Auf dem Live-Desktop „Install OpenIndiana“ doppelklicken. Ganze Platte wählen, das legt einen ZFS-Pool `rpool` an.
- **Benutzer und root:** Wie bei Solaris ist root eine Rolle. Der erste Benutzer bekommt Admin-Rechte (`sudo`/`pfexec`).
- **Gasterweiterungen:** Prüfen mit `pkg list | grep -i virtualbox`. Fehlen sie, wie bei Solaris über `VBoxSolarisAdditions.pkg` installieren.
- **Updates:** `sudo pkg update` legt automatisch eine neue **Boot-Umgebung** an. Schöner Effekt für das Video (siehe `beadm list`).

### Rocky Linux
- **Anaconda-Installer:** Zuerst „Installationsziel“ anklicken und bestätigen, sonst ist „Installation beginnen“ ausgegraut. Root-Passwort oder Benutzer mit Admin-Rechten anlegen.
- **Software-Auswahl:** „Server mit GUI“ für Desktop, „Minimal“ für ein schlankes Terminal-System.
- **Gasterweiterungen** brauchen Build-Werkzeuge:
  ```
  sudo dnf install -y gcc make perl bzip2 kernel-devel-$(uname -r) kernel-headers elfutils-libelf-devel
  sudo sh /run/media/$USER/VBox_GAs_*/VBoxLinuxAdditions.run
  ```
  Vorher `sudo dnf update -y` und neu starten, damit `kernel-devel` zur laufenden Kernel-Version passt.

---

## 5. Kurzvergleich für das Video

| | Oracle Solaris 11.4 | OpenIndiana Hipster | Rocky Linux |
|---|---|---|---|
| Herkunft | Sun Microsystems (SunOS 1982, Solaris ab 1992), seit 2010 Oracle | Community, Nachfolger von OpenSolaris über illumos | Community-Rebuild von Red Hat Enterprise Linux |
| Kern | SunOS-Kernel (System V Release 4) | illumos-Kernel (Fork des OpenSolaris-Kernels) | Linux-Kernel |
| Unix-Zertifikat | Ja, UNIX 03 zertifiziert | Nein, aber technisch Solaris-Familie | Nein, „Unix-artig“ |
| Lizenz | proprietär (CBE-Version kostenlos für nicht-produktive Nutzung) | Open Source (CDDL u. a.) | Open Source (GPL u. a.) |
| Paketverwaltung | IPS: `pkg` | IPS: `pkg` | RPM/DNF: `dnf` |
| Dienste | SMF: `svcs`, `svcadm` | SMF: `svcs`, `svcadm` | systemd: `systemctl` |
| Dateisystem | ZFS (Standard) | ZFS (Standard) | XFS (Standard), ZFS nur extern |
| Container | Zones | Zones | Podman / Container |
| Tracing | DTrace | DTrace | eBPF/bpftrace (DTrace-ähnlich) |
| Netzwerk | `ipadm`, `dladm` | `ipadm`, `dladm` | `ip`, `nmcli` |
| Boot-Umgebungen | `beadm` | `beadm` | nicht standardmäßig |
| Release-Modell | Continuous Delivery mit SRUs | Rolling Release (Hipster) | feste Major-Versionen mit 10 Jahren Support |

---

## 6. Demo-Befehle nach dem ersten Start

Gleiche Aufgabe, drei Systeme. Das ist der Kern des Vergleichs.

| Aufgabe | Solaris / OpenIndiana | Rocky Linux |
|---|---|---|
| System anzeigen | `uname -a`, `cat /etc/release` | `uname -a`, `cat /etc/os-release` |
| Dienste auflisten | `svcs` | `systemctl list-units --type=service` |
| Dienst neu starten | `svcadm restart ssh` | `sudo systemctl restart sshd` |
| Kaputte Dienste | `svcs -xv` | `systemctl --failed` |
| Paket suchen | `pkg search -r nano` | `dnf search nano` |
| Paket installieren | `sudo pkg install nano` | `sudo dnf install nano` |
| Datenträger | `zpool status`, `zfs list` | `lsblk`, `df -h` |
| Snapshot | `sudo zfs snapshot rpool/export/home@demo` | (mit XFS nicht direkt, LVM nötig) |
| Boot-Umgebungen | `beadm list` | – |
| Netzwerk | `ipadm show-addr`, `dladm show-link` | `ip a`, `nmcli device` |
| Prozesse | `prstat` | `top` |
| Tracing-Einzeiler | `sudo dtrace -n 'syscall:::entry { @[execname] = count(); }'` | `sudo bpftrace -e 'tracepoint:raw_syscalls:sys_enter { @[comm] = count(); }'` (Paket `bpftrace`) |

Tipp: Terminal-Schrift für die Aufnahme auf **16–18 pt** stellen und ein helles oder sehr kontrastreiches Farbschema wählen.

---

## 7. Drehbuch mit Erzähltexten

Gesamtlänge YouTube: ca. 15–20 Minuten. Installationen im Schnitt beschleunigen (Zeitraffer 4–8×).

### Szene 1: Hook (0:00–0:30)
*Bild: Drei VM-Fenster nebeneinander, alle drei zeigen `uname -a`.*
> „Drei Betriebssysteme, eine Frage: Was ist eigentlich echtes Unix? Heute installiere ich Oracle Solaris, sein freies Geschwister OpenIndiana und zum Vergleich Rocky Linux, und wir schauen, was gleich ist und was ganz anders.“

### Szene 2: Kurze Geschichte (0:30–2:30)
*Bild: Facecam oder Grafik mit Zeitstrahl.*
> „Unix entstand 1969 bei den Bell Labs von AT&T. Sun Microsystems baute darauf zuerst SunOS und ab 1992 Solaris, ein kommerzielles Unix für Server und Workstations. 2005 öffnete Sun den Quellcode als OpenSolaris. Als Oracle Sun 2010 übernahm, wurde der Code wieder geschlossen. Die Community nahm die letzte offene Version und machte daraus illumos, und darauf baut OpenIndiana auf. Linux dagegen ist kein Unix-Nachfahre im Code, sondern eine Neuentwicklung, die sich wie Unix verhält. Rocky Linux ist ein freier Nachbau von Red Hat Enterprise Linux, also das, was in vielen Rechenzentren läuft.“

### Szene 3: VirtualBox-Setup (2:30–4:00)
*Bild: VirtualBox-Fenster, neue VM anlegen.*
> „Für Solaris wähle ich Typ Solaris, Version Oracle Solaris 11 64-Bit, vier Gigabyte RAM, zwei Kerne und eine 30-Gigabyte-Platte. Wichtig: Grafik auf VBoxVGA und als Netzwerkkarte die Intel PRO/1000, die erkennt Solaris sicher. Und ganz wichtig: die unbeaufsichtigte Installation überspringen, wir wollen ja sehen, was passiert.“

### Szene 4: Solaris installieren (4:00–7:00)
*Bild: Text-Installer, Zeitraffer bei Dateikopie.*
> „Solaris startet mit einem klassischen Text-Installer. Navigation über die Funktionstasten, F2 heißt weiter. Ich wähle die ganze Platte, Solaris legt darauf automatisch einen ZFS-Pool an, das Dateisystem, das Sun 2005 vorgestellt hat und das bis heute als eines der robustesten gilt. Netzwerk automatisch, Zeitzone Europa/Berlin, dann Benutzer anlegen. Auffällig: root ist hier keine normale Anmeldung, sondern eine Rolle, die man über su oder sudo annimmt.“

### Szene 5: Erster Start Solaris (7:00–9:00)
*Bild: Login, Terminal, Befehle aus Abschnitt 6.*
> „Der erste Start dauert etwas, weil die Service Management Facility (SMF) alle Dienste einmal registriert. SMF ist Solaris' Antwort auf die Frage, die Linux später mit systemd beantwortet hat. Mit `svcs` sehe ich alle Dienste, mit `zfs list` die Dateisysteme, und mit `beadm list` meine Boot-Umgebungen. Das heißt: Vor jedem Update entsteht automatisch ein Snapshot des ganzen Systems, zu dem ich beim Booten zurückkehren kann.“

### Szene 6: OpenIndiana (9:00–12:00)
*Bild: Live-Desktop MATE, grafischer Installer, danach Terminal.*
> „OpenIndiana ist der freie Zweig der Familie. Es bootet direkt in einen Live-Desktop, installiert wird per Doppelklick. Unter der Haube ist fast alles wie bei Solaris: ZFS, SMF, Zones, DTrace und sogar der gleiche Paketmanager pkg. Die Befehle von eben funktionieren hier eins zu eins. Der Unterschied liegt vor allem in Lizenz und Community: OpenIndiana ist Open Source und wird als Rolling Release weiterentwickelt.“

### Szene 7: Rocky Linux zum Vergleich (12:00–14:30)
*Bild: Anaconda-Installer im Zeitraffer, danach Terminal.*
> „Und jetzt der Vergleich mit Linux. Rocky Linux installiert sich mit Anaconda, dem Installer von Red Hat. Nach dem Start mache ich dieselben Aufgaben: Statt `svcs` nutze ich `systemctl`, statt `pkg` nutze ich `dnf`, statt ZFS ist hier XFS das Standard-Dateisystem. Die Ideen sind sehr ähnlich, die Werkzeuge haben andere Namen.“

### Szene 8: Vergleich und Fazit (14:30–17:00)
*Bild: Vergleichstabelle aus Abschnitt 5 als Einblendung, Facecam.*
> „Was nehme ich mit? Solaris ist zertifiziertes Unix mit Technik, die ihrer Zeit voraus war: ZFS, Zones, DTrace, Boot-Umgebungen. OpenIndiana bringt diese Technik frei und offen auf den Rechner. Linux hat vieles davon übernommen oder nachgebaut und ist heute der Standard im Rechenzentrum. Wer Linux kann, findet sich in Solaris schnell zurecht, nur die Befehlsnamen sind anders.“

### Szene 9: Outro (17:00–17:30)
> „Im nächsten Video schauen wir uns ZFS und Zones genauer an und vergleichen sie mit Linux-Containern. Schreibt in die Kommentare, welches System ihr selbst schon benutzt habt.“

---

## 8. Reels, Shorts und Stories (Hochformat, 30–60 s)

1. **„Echtes Unix in 60 Sekunden“**: Solaris bootet, `uname -a`, `cat /etc/release`, Satz: „Das ist zertifiziertes UNIX, kein Linux.“
2. **„Gleiche Aufgabe, anderer Befehl“**: Split-Screen `svcs` vs. `systemctl`, `pkg install` vs. `dnf install`.
3. **„Zeitmaschine fürs Betriebssystem“**: `beadm list`, Update, Neustart, zurück zur alten Boot-Umgebung.
4. **„Solaris-Familienstammbaum“**: Grafik Unix → SunOS → Solaris → OpenSolaris → illumos → OpenIndiana, daneben Linux als eigener Zweig.
5. LinkedIn: kurzer Text plus Vergleichstabelle als Bild, Fokus „Was Linux-Admins von Solaris lernen können“.

---

## 9. OBS-Tipps für VM-Aufnahmen

### Quelle
- **Fensteraufnahme** auf das VM-Fenster, Aufnahmemethode „Windows 10 (1903 und höher)“. Funktioniert auch, wenn das Fenster teilweise verdeckt ist.
- Alternativ **Bildschirmaufnahme** mit VM im Vollbild (Host-Taste + F). Dann sieht man keine VirtualBox-Leisten.
- Bei Fensteraufnahme mit **Zuschneiden (Alt + Ziehen)** Menü- und Statusleiste von VirtualBox entfernen.

### Auflösung
- Vor den Gasterweiterungen haben VMs oft nur 1024×768 oder 800×600. In OBS die Quelle mit **Skalierungsfilter „Punktabtastung“** (Rechtsklick → Skalierungsfilter) vergrößern, so bleibt Text scharf.
- Nach den Gasterweiterungen VM-Auflösung auf **1920×1080** stellen.

### Ausgabe
- Format **MKV** aufnehmen (übersteht Abstürze), danach in OBS über *Datei → Aufnahmen remuxen* in MP4 umwandeln.
- Encoder: Hardware (NVENC, AMD AMF oder Intel QuickSync), Qualität CQP/CRF ca. 18–20.
- **1080p, 30 fps** reichen für Terminal und Installer völlig.
- Mikrofon auf eigene **Audiospur** legen, Filter: Rauschunterdrückung und Kompressor.

### Szenen
- Szene „Facecam“, Szene „VM Vollbild“, Szene „VM + Facecam klein“, Szene „Split Screen“ (zwei Fensteraufnahmen nebeneinander für den Vergleich).
- Für Reels ein **zweites Profil mit Leinwand 1080×1920** oder das Plugin **Aitum Vertical** nutzen, dann entstehen Quer- und Hochformat gleichzeitig.

### Ablauf
- Installation in **Abschnitten** aufnehmen (Pause-Hotkey), Wartezeiten nicht mitschneiden oder im Schnitt beschleunigen.
- Vor jeder Szene zum passenden **Snapshot** zurückkehren, so lassen sich Versprecher einfach neu drehen.
- Benachrichtigungen unter Windows auf „Nicht stören“, Uhrzeit/Desktop-Symbole ausblenden.
- Kurze **Testaufnahme von 30 Sekunden** machen und Ton sowie Lesbarkeit prüfen, bevor die echte Aufnahme startet.
