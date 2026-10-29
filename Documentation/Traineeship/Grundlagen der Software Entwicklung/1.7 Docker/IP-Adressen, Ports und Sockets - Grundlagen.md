Begleitnotiz zu [[1.7 Docker - Vertiefung|1.7 Docker – Vertiefung]] (Stichwort *Networks*). Entstanden aus einer Frage beim Port-Mapping: Woher kommt eigentlich `127.0.0.1`, was ist `0.0.0.0`, und warum darf man einfach so darauf lauschen?

Das Thema liegt **unter** dem, was in [[1.5 REST APIs & Layered Architecture - Vertiefung|1.5]] behandelt wurde: Dort ging es um HTTP — also darum, was *über* eine bestehende Verbindung gesprochen wird. Hier geht es darum, wie die Verbindung überhaupt zustande kommt.

# Inhaltsverzeichnis

- [[#1. Die zwei Fragen jeder Verbindung]]
- [[#2. IP-Adressen]]
- [[#3. Die Sonderadressen — und woher `127.0.0.1` kommt]]
- [[#4. Ports]]
- [[#5. Der Socket: warum ein Server-Port nicht „belegt" wird]]
- [[#6. Lauschen vs. Verbinden — hier sitzt das Missverständnis]]
- [[#7. Namen statt Zahlen: `/etc/hosts` und DNS]]
- [[#7b. Bridges, `veth`-Paare und Dockers DNS]]
- [[#8. Und jetzt alles zusammen: der Container]]
- [[#9. Befehle zum Selbstansehen]]

---

# 1. Die zwei Fragen jeder Verbindung

Wenn ein Programm ein anderes über ein Netzwerk erreichen will, müssen genau **zwei** Dinge geklärt sein:

| Frage | Antwort |
|---|---|
| **Welche Maschine?** | die **IP-Adresse** |
| **Welches Programm auf dieser Maschine?** | der **Port** |

Die klassische Analogie ist das Bürogebäude: Die **Straßenadresse** bringt den Brief zum Gebäude (IP), die **Zimmernummer** zum richtigen Schreibtisch (Port). Ohne Zimmernummer landet er in der Poststelle und niemand weiß, wohin damit.

Deshalb schreibt man beides zusammen, getrennt durch einen Doppelpunkt:

```
127.0.0.1 : 8000
└── wer ──┘ └─ was ─┘
```

# 2. IP-Adressen

## Der Aufbau

Eine **IPv4**-Adresse besteht aus **4 Bytes** — vier Zahlen von 0 bis 255, durch Punkte getrennt:

```
192 . 168 . 178 . 42
└┬┘   └┬┘   └┬┘   └┬┘
 je ein Byte (0–255)
```

Vier Bytes sind 32 Bit, also gut **4,3 Milliarden** mögliche Adressen. Das klang 1981 nach unendlich viel und ist seit den 2010ern aufgebraucht — daher **IPv6** mit 128 Bit (`2001:db8::1`), das im Alltag aber selten stört, weil beides parallel läuft.

## Der Punkt, den man verinnerlichen muss

> [!important] Eine IP-Adresse gehört nicht dem Rechner, sondern einer **Netzwerkschnittstelle**
> Und ein Rechner hat **mehrere** Schnittstellen. Dein Laptop hat mindestens:
>
> | Schnittstelle | Typische Adresse | Was sie ist |
> |---|---|---|
> | `lo` | `127.0.0.1` | Loopback — rein virtuell, existiert immer |
> | `wlan0` | `192.168.178.42` | die WLAN-Karte, Adresse vom Router per DHCP |
> | `eth0` | `192.168.178.43` | Kabel-Netzwerkkarte |
> | `docker0` | `172.17.0.1` | die Bridge, die Docker angelegt hat |
> | `veth…` | — | je ein Kabelende pro laufendem Container |
>
> „Die IP meines Rechners" gibt es also gar nicht. Es gibt nur „die IP dieser Schnittstelle". Genau daraus folgt fast alles Weitere in dieser Notiz.

Ansehen kannst du dir das mit:

```bash
ip -brief addr
```

# 3. Die Sonderadressen — und woher `127.0.0.1` kommt

Nicht jede Adresse ist eine echte Adresse im Internet. Einige Bereiche sind reserviert:

| Bereich | Name | Bedeutung |
|---|---|---|
| `127.0.0.0` – `127.255.255.255` | **Loopback** | „diese Maschine selbst" |
| `10.x.x.x`, `172.16–31.x.x`, `192.168.x.x` | **private Bereiche** | nur im lokalen Netz gültig, nicht im Internet routbar |
| `0.0.0.0` | **unspezifiziert** | Platzhalter, keine echte Adresse |
| `255.255.255.255` | Broadcast | „an alle im lokalen Netz" |

## Das Loopback-Netz

`127.0.0.1` ist keine willkürliche Zahl, sondern ein **Standard**: Das gesamte Netz `127.0.0.0/8` — immerhin 16,7 Millionen Adressen — ist seit den frühen Internet-Spezifikationen für **Loopback** reserviert.

Loopback bedeutet: Pakete an diese Adressen **verlassen die Maschine nie**. Der Kernel nimmt sie entgegen und reicht sie sofort wieder nach oben, ohne dass eine Netzwerkkarte, ein Kabel oder ein Router beteiligt wäre.

Dafür stellt der Kernel eine **virtuelle Schnittstelle** namens `lo` bereit. Und damit ist auch deine eigentliche Frage beantwortet:

> [!question] Warum kann ich einfach so auf `127.0.0.1` lauschen?
> Weil `lo` **immer existiert und immer aktiv ist** — ohne Netzwerkkarte, ohne Kabel, ohne WLAN, ohne DHCP, ohne Router. Sie ist Teil des Betriebssystems, nicht der Hardware.
>
> Deshalb funktioniert `uvicorn --host 127.0.0.1` auch im Zug ohne Empfang. Und deshalb ist es die sichere Standardwahl in der Entwicklung: Was auf `lo` lauscht, ist von außen **prinzipiell** nicht erreichbar — nicht, weil eine Firewall es blockiert, sondern weil es physisch keinen Weg dorthin gibt.

Dass gleich 16,7 Millionen Adressen dafür reserviert sind, ist historische Großzügigkeit. Praktisch nutzt man `127.0.0.1`. Auf deinem System ist aber auch eine zweite in Gebrauch — schau in `/etc/hosts`:

```
127.0.0.1	localhost
127.0.1.1	thinkpad
```

Der eigene Rechnername `thinkpad` zeigt bei Debian per Konvention auf `127.0.1.1`, damit er auflösbar bleibt, auch wenn gerade kein Netz da ist. Zwei verschiedene Adressen, dieselbe `lo`-Schnittstelle.

## `0.0.0.0` ist keine Adresse

Das ist der Punkt, der am meisten verwirrt:

> [!warning] `0.0.0.0` ist ein **Platzhalter**, kein Ziel
> Man verbindet sich **nie zu** `0.0.0.0`. Die Adresse taucht ausschließlich beim **Lauschen** auf und bedeutet dort: *„nimm Verbindungen auf allen Schnittstellen an, die diese Maschine hat"* — also auf `lo`, auf `wlan0`, auf `docker0`, auf allem.
>
> Deshalb ist die Frage „Ist `0.0.0.0` dasselbe wie localhost?" so schwer zu beantworten: `127.0.0.1` ist **eine** Adresse, `0.0.0.0` ist **alle**. Sie überschneiden sich, sind aber nicht dasselbe — so wie „mein Zuhause" und „überall" sich überschneiden.

# 4. Ports

Ein Port ist eine Zahl mit **16 Bit**, also von 0 bis **65535**. Er identifiziert nicht das Programm selbst, sondern den *Endpunkt*, den ein Programm sich beim Betriebssystem reserviert hat.

Die Bereiche sind in drei Gruppen eingeteilt:

| Bereich | Name | Anmerkung |
|---|---|---|
| 1 – 1023 | **Well-known** | HTTP 80, HTTPS 443, SSH 22, PostgreSQL 5432¹ — **Binden erfordert root** |
| 1024 – 49151 | **Registered** | 8000, 8080, 3000, 5432 … frei nutzbar |
| ab 49152 | **Dynamic / Ephemeral** | vergibt das System selbst an *Clients* |

¹ 5432 liegt tatsächlich im registrierten Bereich — nur zur Erinnerung, dass „bekannter Port" und „Well-known-Bereich" nicht dasselbe sind.

> [!tip] Warum uvicorn auf 8000 läuft und nicht auf 80
> Ports unter 1024 darf unter Linux nur root binden. Ein Entwicklungsserver soll aber nicht als root laufen — also nimmt man einen darüber. 8000 und 8080 haben sich als „HTTP, aber unprivilegiert" eingebürgert.
>
> Das erklärt auch eine Einschränkung aus Stichwort 2 der Vertiefung: Im **Rootless Mode** kann Docker ohne Zusatzkonfiguration keine Ports unter 1024 veröffentlichen — genau aus diesem Grund.

## Der Client-Port, an den niemand denkt

Wenn du `curl http://127.0.0.1:8000` aufrufst, kennst du den Ziel-Port. Aber auch **curl selbst** bekommt einen Port — einen zufälligen aus dem Ephemeral-Bereich, etwa 51834. Ohne den wüsste der Server nicht, wohin er antworten soll.

Das ist die Voraussetzung für den nächsten Abschnitt.

# 5. Der Socket: warum ein Server-Port nicht „belegt" wird

Eine der hartnäckigsten Fehlvorstellungen ist, ein Server könne pro Port nur eine Verbindung haben. Tatsächlich wird eine TCP-Verbindung durch **vier** Werte eindeutig identifiziert:

```
(Quell-IP, Quell-Port, Ziel-IP, Ziel-Port)
```

Drei gleichzeitige Browser-Tabs auf deiner API ergeben also:

```
(127.0.0.1, 51834, 127.0.0.1, 8000)
(127.0.0.1, 51835, 127.0.0.1, 8000)
(127.0.0.1, 51836, 127.0.0.1, 8000)
       └── nur DAS unterscheidet sie
```

Alle drei landen bei demselben lauschenden Programm auf Port 8000, und der Kernel kann sie trotzdem sauber auseinanderhalten. **Ein Server-Port wird durch Verbindungen nicht verbraucht** — er ist keine Leitung, sondern eine Adresse.

Was tatsächlich exklusiv ist, ist das **Lauschen**: Das Paar (IP, Port) darf nur ein Programm für sich beanspruchen. Genau darum geht es im nächsten Abschnitt.

# 6. Lauschen vs. Verbinden — hier sitzt das Missverständnis

Ein Programm kann mit einem Socket zwei völlig verschiedene Dinge tun:

| Operation | Wer | Frage | `0.0.0.0` sinnvoll? |
|---|---|---|---|
| **binden + lauschen** | Server | „Auf welcher Schnittstelle nehme ich Verbindungen an?" | ✅ ja — „auf allen" |
| **verbinden** | Client | „Zu welcher Adresse will ich?" | ❌ nein — „alle" ist kein Ziel |

`--host` bei uvicorn ist **kein Ziel**, sondern die Bind-Adresse. Deshalb ergibt `0.0.0.0` dort Sinn und in `curl http://0.0.0.0:8000` streng genommen nicht (es funktioniert meist trotzdem, weil viele Systeme es hilfsweise als `127.0.0.1` interpretieren).

## Was daraus für die Erreichbarkeit folgt

```
uvicorn --host 127.0.0.1     →  erreichbar NUR über lo
                                (nur Prozesse auf derselben Maschine)

uvicorn --host 192.168.178.42 → erreichbar NUR über wlan0
                                (nur aus dem WLAN, nicht mal über localhost!)

uvicorn --host 0.0.0.0        → erreichbar über ALLE Schnittstellen
```

Die mittlere Zeile ist der Aha-Moment: Ein Server, der an eine konkrete WLAN-Adresse gebunden ist, ist über `127.0.0.1` **nicht** erreichbar — obwohl es dieselbe Maschine ist. Bindung heißt wirklich Bindung an *eine Schnittstelle*.

## „address already in use"

Damit lässt sich auch dein Fehler von vorhin genau erklären. Kollisionen entstehen nur bei **überlappenden** (IP, Port)-Paaren:

| Programm A | Programm B | Kollision? |
|---|---|---|
| `127.0.0.1:8000` | `127.0.0.1:8000` | ✅ ja |
| `0.0.0.0:8000` | `127.0.0.1:8000` | ✅ ja — `0.0.0.0` schließt `127.0.0.1` ein |
| `127.0.0.1:8000` | `192.168.178.42:8000` | ❌ nein — verschiedene Schnittstellen |
| `127.0.0.1:8000` | `127.0.0.1:8001` | ❌ nein — verschiedene Ports |

`docker run -p 8000:8000` bindet auf dem Host an `0.0.0.0:8000`. Lief dort schon dein uvicorn auf `127.0.0.1:8000`, überlappen die beiden — daher `address already in use`.

# 7. Namen statt Zahlen: `/etc/hosts` und DNS

Menschen tippen ungern Zahlen. Die Übersetzung von Namen zu IP-Adressen läuft in dieser Reihenfolge:

1. **`/etc/hosts`** — eine simple Textdatei auf deinem Rechner. Was hier steht, gewinnt.
2. **DNS** — ein Netzwerkdienst, der befragt wird, wenn Schritt 1 nichts liefert.

Deshalb ist `localhost` **kein magisches Schlüsselwort**, sondern ein ganz normaler Eintrag in einer Textdatei:

```bash
$ cat /etc/hosts
127.0.0.1	localhost
127.0.1.1	thinkpad
::1         localhost ip6-localhost ip6-loopback
```

Man könnte die Zeile löschen, und `localhost` funktionierte nicht mehr. Man könnte auch `1.2.3.4 localhost` eintragen — dann zeigte der Name dorthin.

> [!note] Der Bezug zu Docker
> Genau dieser Mechanismus ist es, den Compose ausnutzt. Im Container steht in `/etc/resolv.conf` ein **eingebauter DNS-Server unter `127.0.0.11`**, den Docker betreibt. Er kennt die Dienstnamen des Compose-Netzes und löst `db` zur aktuellen Container-IP auf.
>
> Deshalb ist `db` ein gültiger Hostname — und deshalb funktioniert das nur in **selbst angelegten** Netzwerken, wie in Stichwort 3 beschrieben. Nachsehen:
> ```bash
> docker compose exec api cat /etc/resolv.conf
> docker compose exec api getent hosts db
> ```

# 7b. Bridges, `veth`-Paare und Dockers DNS

Bevor es an den Container geht, drei Begriffe, die vorausgesetzt werden — alle drei sind Software-Nachbauten von etwas Physischem.

## Die Bridge ist ein virtueller Switch

Ein **Switch** ist ein Kasten mit Buchsen: Man steckt Kabel hinein, und was an Buchse 3 hereinkommt und für Buchse 5 bestimmt ist, wird dorthin weitergeschoben. Er übersetzt nichts und routet nicht — er arbeitet eine Ebene unter IP, mit **MAC-Adressen** (der Hardware-Kennung jeder Netzwerkkarte).

Daraus folgen zwei Dinge, die den Kern ausmachen:

- **Alles am selben Switch kann sich direkt erreichen.**
- **Was an einem anderen Switch hängt, nicht** — solange niemand die Switches verbindet.

Der Linux-Kernel kann das in Software: Ein **Bridge** ist ein virtueller Switch. `docker0` und die `br-<netzwerk-id>`-Interfaces sind genau das. (Der Name ist historisch: Ein Bridge verband ursprünglich zwei Netzsegmente; ein Switch ist im Grunde ein Bridge mit vielen Anschlüssen.)

**Jedes Docker-Netzwerk ist ein eigener Switch.** Deshalb sehen sich Container in verschiedenen Netzen nicht — nicht weil etwas blockiert, sondern weil kein Kabel dazwischen liegt.

```bash
ip -brief link show type bridge
```

## Das `veth`-Paar ist das Kabel

Ein Container hat durch den `net`-Namespace einen **eigenen, anfangs leeren** Netzwerk-Stack — nur `lo` ist da. Um ihn an einen Switch anzuschließen, braucht es ein Kabel, und auch das gibt es in Software.

Ein **`veth`-Paar** (virtual ethernet) entsteht immer als **Paar**: zwei Enden, und was in das eine hineingeht, kommt aus dem anderen heraus. Genau wie ein Patchkabel.

```
   Host-Namespace                    Container-Namespace
   ┌────────────────────┐            ┌──────────────────┐
   │  br-f8cdcaa3e337   │            │                  │
   │  (der Switch)      │            │                  │
   │       │            │            │                  │
   │   veth7a3f ════════╪════════════╪══► eth0          │
   │   (Ende A,         │  das Kabel │    (Ende B,      │
   │    eingesteckt)    │            │     umbenannt)   │
   └────────────────────┘            └──────────────────┘
```

Docker legt das Paar an, steckt Ende A in die Bridge und **verschiebt** Ende B in den Namespace des Containers, wo es `eth0` heißt. Deshalb erscheint auf dem Host pro laufendem Container ein `veth…`-Interface:

```bash
ip -brief link | grep -E 'br-|veth'
```

## Die Adressvergabe

Jedes Docker-Netzwerk hat ein **Subnetz** (typischerweise `172.17.0.0/16` für `docker0`, `172.18.0.0/16` für das erste selbst angelegte). Docker vergibt daraus der Reihe nach — es spielt nebenbei DHCP-Server:

| Adresse | Wer |
|---|---|
| `172.18.0.1` | die **Bridge selbst** — zugleich das Gateway nach draußen |
| `172.18.0.2` | erster angeschlossener Container |
| `172.18.0.3` | zweiter Container |

```bash
docker network inspect <netz> | jq '.[0].IPAM.Config, .[0].Containers'
```

## Was „im DNS registriert" bedeutet

Container könnten sich damit schon über IP-Adressen erreichen — nur sind die **nicht stabil**: Bei jedem Neustart kann ein Container je nach Startreihenfolge eine andere bekommen. Fest hinschreiben kann man sie nirgends.

> **Docker betreibt deshalb in jedem Container-Namespace einen winzigen DNS-Server unter `127.0.0.11`.**

Das ist eine Loopback-Adresse *des Containers* (aus dem reservierten `127.0.0.0/8`-Bereich), und dort horcht ein Docker-Prozess:

```bash
docker run --rm --network <netz> alpine cat /etc/resolv.conf
# → nameserver 127.0.0.11
```

Er führt pro Netzwerk eine Tabelle Name → aktuelle IP:

| Name | IP |
|---|---|
| `db`, `pricing-db` | 172.18.0.2 |
| `adminer`, `pricing-adminer` | 172.18.0.4 |

**„Im DNS registriert" heißt also schlicht: Docker hat den Container beim Anschließen mit seinem Namen in diese Tabelle eingetragen.** Fragt ein Container nach `db`, geht die Anfrage an `127.0.0.11` und wird mit der *momentan gültigen* Adresse beantwortet. Unbekannte Namen (`google.com`) reicht Docker an die DNS-Server des Hosts weiter.

Im **vorgegebenen `bridge`-Netz gibt es diesen Dienst nicht** — ein Überbleibsel aus der Zeit vor selbst angelegten Netzwerken. Das ist der ganze Inhalt der Regel, dass Namensauflösung nur in eigenen Netzen funktioniert.

## `--network <netz>` in vier Schritten

1. **Kabel gebaut** — ein `veth`-Paar erzeugt
2. **Eingesteckt** — Ende A in die Bridge, Ende B als `eth0` in den Container
3. **Adresse vergeben** — die nächste freie aus dem Subnetz
4. **Namen eingetragen** — Containername in die DNS-Tabelle dieses Netzes

Ohne 1–2 hinge der Container am falschen Switch; ohne 4 wäre `db` ein unbekannter Name.

# 8. Und jetzt alles zusammen: der Container

Aus Stichwort 1 der Vertiefung weißt du: Der `net`-Namespace gibt jedem Container einen **eigenen Netzwerk-Stack**. Das heißt konkret — eigene Schnittstellen, eigene Adressen, eigene Ports.

```
   ┌──────────────────── HOST (thinkpad) ─────────────────────┐
   │  lo       127.0.0.1                                      │
   │  wlan0    192.168.178.42                                 │
   │  docker0  172.17.0.1     ← Bridge, von Docker angelegt   │
   │  veth1a2b ─────────────┐                                 │
   │                        │  virtuelles "Kabel"             │
   │  iptables-DNAT:        │                                 │
   │  0.0.0.0:8000 → 172.17.0.2:8000                          │
   └────────────────────────┼─────────────────────────────────┘
                            │
   ┌────────────────────────┼──── CONTAINER ──────────────────┐
   │  lo    127.0.0.1       │   ← eigenes Loopback!           │
   │  eth0  172.17.0.2  ◄───┘                                 │
   │                                                          │
   │  uvicorn --host 0.0.0.0 --port 8000                      │
   │     lauscht auf lo UND eth0  ✅                          │
   └──────────────────────────────────────────────────────────┘
```

Damit lösen sich drei Fragen auf einmal:

**Warum kollidieren Container-Ports nicht mit Host-Ports?**
Weil es verschiedene Netzwerk-Stacks sind. Zehn Container dürfen alle auf Port 8000 lauschen — sie tun es auf zehn verschiedenen `eth0`-Schnittstellen mit zehn verschiedenen IPs. Nur beim **Veröffentlichen** auf dem Host wird es eng, weil sich dort alle denselben Stack teilen.

**Warum muss uvicorn im Container auf `0.0.0.0` lauschen?**
Der Verkehr vom Host kommt über die Bridge auf **`eth0`** an, nicht auf `lo`. Ein Server auf `127.0.0.1` würde ihn nicht annehmen — und `127.0.0.1` bedeutet im Container ohnehin *den Container selbst*, nicht deinen Laptop.

**Was macht `-p 8000:8000` eigentlich?**
Es weist den Daemon an, auf dem **Host** an `0.0.0.0:8000` zu lauschen und ankommende Pakete per **DNAT** auf `172.17.0.2:8000` umzuschreiben. Die linke Zahl ist eine Host-Adresse, die rechte eine Container-Adresse — sie haben nichts miteinander zu tun außer der Regel, die Docker dazwischenschreibt.

## Der Rückweg — „lauscht der da nicht nur?"

Naheliegende Frage: Wenn Docker auf dem Host-Port nur *lauscht* und nach innen weiterleitet — wie kommt die Antwort zurück?

**Erstens: „Lauschen" betrifft nur den Verbindungsaufbau.** Ein Server tut drei getrennte Dinge:

```c
bind()     // "ich bin unter dieser Adresse:Port erreichbar"
listen()   // "neue Anrufer bitte durchstellen"
accept()   // "ich nehme diesen einen Anruf an"
```

`accept()` liefert einen **neuen, eigenen Socket** für *diese eine Unterhaltung*; der lauschende Socket wartet unverändert auf den nächsten Anrufer.

| Socket | Zweck | Richtung |
|---|---|---|
| Listening-Socket (`0.0.0.0:8000`) | neue Verbindungen annehmen | nur rein |
| Verbindungs-Socket (pro Client) | Daten übertragen | **rein und raus** |

Die Antwort geht also nicht „an einen Port zurück", sondern durch die **bestehende, beidseitige Verbindung**. Wie beim Telefon: Die Durchwahl ist die Nummer, unter der man dich *erreicht* — sobald das Gespräch steht, reden beide.

**Zweitens: NAT ist zustandsbehaftet.** Der Kernel merkt sich jede übersetzte Verbindung in der *conntrack*-Tabelle:

```
Original:    127.0.0.1:51834 → 127.0.0.1:8000
Übersetzt:   127.0.0.1:51834 → 172.17.0.2:8000
```

Antwortet uvicorn mit `172.17.0.2:8000 → 127.0.0.1:51834`, erkennt der Kernel die bekannte Verbindung und **dreht die Umschreibung automatisch um** zu `127.0.0.1:8000 → 127.0.0.1:51834`.

Das muss so sein: Käme die Antwort sichtbar von `172.17.0.2:8000`, würde der TCP-Stack des Clients sie verwerfen — sie gehörte zu keiner ihm bekannten Verbindung. Und deshalb gibt es nur **eine** DNAT-Regel und keine zweite für den Rückweg.

```bash
sudo conntrack -L | grep 8000
sudo cat /proc/net/nf_conntrack | grep 8000
```

**Und tatsächlich lauscht dort auch ein Prozess.** In der Standardkonfiguration startet Docker pro veröffentlichtem Port ein Hilfsprogramm namens `docker-proxy`:

```bash
ss -tlnp | grep 8000     # → users:(("docker-proxy",...))
```

Es fängt Fälle ab, die `iptables` nicht sauber abdeckt (Loopback- und Hairpin-Verkehr), und sorgt dafür, dass eine Portkollision ein sauberes `address already in use` ergibt statt einer stillen Fehlfunktion. Ein Proxy leitet naturgemäß in **beide** Richtungen weiter.

> [!important] „localhost" ist immer relativ
> | Wo du es tippst | Was es bedeutet |
> |---|---|
> | in deinem Terminal | dein Laptop |
> | in einem Container | **dieser Container** |
> | in `DB_HOST` der API im Container | der API-Container — **nicht** die Datenbank |
>
> Das ist der ganze Grund, warum `DB_HOST` von `localhost` auf `db` wechseln muss.

# 9. Befehle zum Selbstansehen

```bash
# Welche Schnittstellen und Adressen hat diese Maschine?
ip -brief addr

# Wer lauscht gerade auf welchem Port? (t=tcp l=listening n=numerisch p=prozess)
ss -tlnp

# Bestehende Verbindungen — hier siehst du das 4-Tupel aus Abschnitt 5
ss -tnp

# Wie löst sich ein Name auf?
getent hosts localhost
getent hosts thinkpad

# Dasselbe im Container
docker compose exec api ip -brief addr
docker compose exec api cat /etc/resolv.conf     # → nameserver 127.0.0.11
docker compose exec api getent hosts db

# Was hat Docker im Kernel eingetragen?
docker network inspect pricing_default
sudo iptables -t nat -L DOCKER -n
```

Der lehrreichste ist `ss -tlnp`. In der Spalte `Local Address:Port` steht bei jedem Dienst genau die Bind-Adresse — dort siehst du unmittelbar, ob ein Programm auf `127.0.0.1`, auf `0.0.0.0` oder auf einer konkreten Schnittstelle lauscht.

# Merksätze

1. **IP = welche Schnittstelle, Port = welches Programm.**
2. Eine IP gehört einer **Schnittstelle**, nicht einem Rechner. Jeder Rechner hat mehrere.
3. **`127.0.0.1` ist eine Adresse** (die virtuelle Schnittstelle `lo`, immer da). **`0.0.0.0` ist keine** — es heißt beim Lauschen „auf allen Schnittstellen".
4. **Lauschen** darf `0.0.0.0` benutzen, **Verbinden** nicht.
5. Ein Server-Port wird durch Verbindungen **nicht verbraucht** — eine Verbindung ist ein 4-Tupel.
6. **Lauschen betrifft nur den Verbindungsaufbau.** Eine bestehende TCP-Verbindung ist beidseitig; `accept()` erzeugt dafür einen eigenen Socket. Und **NAT ist zustandsbehaftet** — der Rückweg wird automatisch zurückübersetzt, ohne zweite Regel.
7. `address already in use` heißt: Zwei Programme wollen dasselbe **(IP, Port)-Paar**. `0.0.0.0` kollidiert dabei mit allem.
8. **`localhost` ist relativ** — im Container ist es der Container.
9. Ports **unter 1024** brauchen root. Darum 8000.

# Selbstcheck

1. Warum gibt es „die IP-Adresse meines Rechners" streng genommen nicht?
2. Warum funktioniert `127.0.0.1` ohne Netzwerkkarte und ohne Internet?
3. Was passiert, wenn du in `/etc/hosts` die Zeile mit `localhost` löschst?
4. Ein Server lauscht auf `192.168.178.42:8000`. Erreichst du ihn über `http://127.0.0.1:8000`? Warum (nicht)?
5. Drei Browser-Tabs sind gleichzeitig mit deiner API auf Port 8000 verbunden. Warum kommen sie sich nicht ins Gehege?
6. Warum kollidieren `0.0.0.0:8000` und `127.0.0.1:8000`, aber nicht `127.0.0.1:8000` und `192.168.178.42:8000`?
7. Warum läuft uvicorn auf 8000 und nicht auf 80?
8. Warum muss ein Server im Container auf `0.0.0.0` lauschen, auf deinem Laptop aber nicht?
9. Was bedeutet `localhost` innerhalb eines Containers?
10. Woher weiß ein Container, dass `db` die Adresse eines anderen Containers ist?
11. Docker lauscht auf dem Host-Port und leitet nach innen weiter — wie kommt die Antwort zurück? Nenne beide Gründe.
