# Installation OWASP Juice Shop

Diese Anleitung beschreibt, wie ihr den OWASP Juice Shop für den Workshop installiert. Bitte macht das **vor** dem Workshop — wenn etwas nicht funktioniert, gebt mir vorher Bescheid.

Es gibt zwei Hauptvarianten und zwei Fallback-Möglichkeiten. Wählt die, die für eure Umgebung am besten passt.

---

## Variante 1: Docker (empfohlen)

Die schnellste und sauberste Variante. Voraussetzung: Docker Engine auf eurem System installiert.

**Docker installieren**, falls noch nicht vorhanden:
[https://docs.docker.com/get-docker/](https://docs.docker.com/get-docker/)

**Juice Shop starten:**

```bash
docker pull bkimminich/juice-shop
docker run --rm -e "NODE_ENV=unsafe" -p 3000:3000 bkimminich/juice-shop
```

Sobald der Server gestartet ist, ist Juice Shop unter [http://localhost:3000](http://localhost:3000) erreichbar.

### Warum `NODE_ENV=unsafe`?

In Docker-Containern sind standardmäßig einige Challenges deaktiviert, weil sie potenziell den Container beschädigen können — z.B. alle Stored XSS, XXE und einige NoSQL-Injection-Challenges. Für unseren Workshop brauchen wir diese aber, weil sie zu den Themen in Block 2 gehören. Mit `NODE_ENV=unsafe` werden alle Challenges aktiviert.

Das ist nur deshalb sicher, weil ihr den Container lokal auf eurem eigenen Rechner laufen lasst. **Setzt `unsafe` niemals in einer öffentlich erreichbaren Instanz.**

### Container stoppen

`Strg + C` im Terminal, in dem der Container läuft.

---

## Variante 2: Node.js (Quellcode)

Wenn ihr Docker nicht installieren wollt oder könnt, könnt ihr Juice Shop auch direkt aus dem Quellcode laufen lassen.

**Voraussetzung:** Node.js Version 20 oder neuer. Prüfen mit `node --version`.

**Repository klonen und starten:**

```bash
git clone https://github.com/juice-shop/juice-shop.git --depth 1
cd juice-shop
npm install
npm start
```

Sobald der Server gestartet ist, ist Juice Shop unter [http://localhost:3000](http://localhost:3000) erreichbar.

### Muss ich hier auch den Safe Mode deaktivieren?

**Nein.** Bei der Quellcode-Variante läuft Juice Shop standardmäßig im Development-Modus, und alle Challenges sind aktiv. Die Safe-Mode-Beschränkungen greifen nur in Container- und Cloud-Umgebungen, nicht bei lokalem Node.js.

**Wichtig:** Setzt also `NODE_ENV` *nicht* auf `production` — dann wäre es ähnlich beschränkt wie der Docker-Container.

### Bekannte Stolperstellen

- Beim ersten `npm install` werden Dependencies geladen. Das kann mehrere Minuten dauern und produziert je nach Setup einige Warnings (z.B. zu deprecated packages). Das ist normal, solange am Ende der Server tatsächlich startet.
- Auf Windows kann es helfen, das Terminal als Administrator zu öffnen, falls `npm install` mit Rechtefehlern abbricht.
- Falls `git clone` einen langen Verlauf herunterlädt: Das `--depth 1` sorgt dafür, dass nur der aktuelle Stand und nicht die komplette History geklont wird (spart ca. 700 MB).

---

## Fallback 1: Zusammentun (Pair-Hacking)

Falls weder Docker noch Node.js bei euch funktionieren — kein Drama. Schließt euch im Workshop einfach mit jemandem zusammen, bei dem es läuft.

---

## Fallback 2: Offizielle Demo-Instanz

Es gibt eine offizielle Online-Demo unter [https://demo.owasp-juice.shop/](https://demo.owasp-juice.shop/).

**Nachteile, die ihr kennen müsst:**

Die offizielle Juice-Shop-Dokumentation sagt sehr klar: Eine Juice-Shop-Instanz ist für **einen einzelnen Nutzer** gedacht. Das hat zwei konkrete Konsequenzen:

- **Fortschritts-Tracking vermischt sich.** Wenn ihr eine Challenge löst, sehen das alle anderen Nutzer der Instanz. Umgekehrt seht ihr Challenges, die andere weltweit gerade lösen.
- **Self-Healing kann jederzeit zurücksetzen.** Juice Shop hat einen Mechanismus, der den Datenbank-Zustand regelmäßig zurücksetzt. Auf der geteilten Demo-Instanz kann das mitten in eurer Arbeit passieren.

---

## Funktionscheck

Egal welche Variante ihr gewählt habt — so wisst ihr, dass alles funktioniert:

- Beim Aufruf von [http://localhost:3000](http://localhost:3000) seht ihr die Startseite des Juice Shop (Produkt-Galerie mit Saft-Symbol oben links).
- Ihr könnt euch einen Test-Account anlegen: oben rechts auf "Account" → "Login" → "Not yet a customer?". Eure E-Mail muss nicht echt sein — `test@example.com` reicht.
- Nach dem Login könnt ihr Produkte in den Warenkorb legen.
- Wenn ihr in der URL `/#/score-board` aufruft, kommt eine Meldung wegen einer "easter egg challenge" (das ist die erste Challenge, die ihr im Workshop nicht lösen müsst — die zeigt nur, dass alles läuft).

Wenn diese vier Punkte funktionieren, seid ihr bereit für den Workshop.
