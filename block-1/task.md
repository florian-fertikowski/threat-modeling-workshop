# Block 1: Bedrohungsmodellierung — Aufgabe

**Gesamtzeit:** 30 Minuten · **Werkzeug:** OWASP Threat Dragon

---

## Phase A: Trust Boundaries einzeichnen (5 min)

**Aufgabe:** Findet im Diagramm die Stellen, an denen Daten eine Vertrauensgrenze überqueren, z.B. zwischen Browser und Backend, zwischen App und Datenbank, zwischen App und externen Diensten.

**Tipp:** Trust Boundaries sind die Linien, an denen ihr euch fragen müsst: *"Kann ich dem, was hier ankommt, vertrauen?"* Alles, was von "außen" kommt, ist erst einmal nicht vertrauenswürdig.

**Faustregel:** Wo immer ein Pfeil in eurem Diagramm eine solche Grenze überquert, gehört dorthin eine Trust Boundary.

---

## Phase B: Eigene Bedrohungen finden (12 min)

**Aufgabe:** Geht **die sechs STRIDE-Kategorien einzeln durch**. Versucht für jede Kategorie selbst eine konkrete Bedrohung zu finden. Tragt sie in Threat Dragon ein.

| Kategorie                  | Kernfrage                                                            |
|----------------------------|----------------------------------------------------------------------|
| **S**poofing               | Wer könnte sich für jemand anderen ausgeben?                         |
| **T**ampering              | Welche Daten könnte jemand manipulieren?                             |
| **R**epudiation            | Welche Aktionen könnte jemand bestreiten?                            |
| **I**nformation Disclosure | Welche Informationen könnten nach außen gelangen, die nicht sollten? |
| **D**enial of Service      | Wie könnte jemand das System lahmlegen?                              |
| **E**levation of Privilege | Wie könnte jemand mehr Rechte bekommen, als er haben sollte?         |

**Wichtig:**

- Denkt aus Angreifersicht: *Wenn ich böse wäre, was würde ich versuchen?*
- Schreibt konkrete Bedrohungen, keine abstrakten Kategorien: Statt "Tampering" z.B. "Ein Angreifer ändert den Preis im Warenkorb, bevor die Bestellung abgeschickt wird"
- **Öffnet noch NICHT** die Datei `inspiration-bedrohungen.md`. Erst denken, dann nachschauen.

---

## Phase C: Auswahl aus der Inspirationsliste (12 min)

Jetzt öffnet ihr `inspiration-bedrohungen.md`. Diese enthält eine Liste verschiedener Schwachstellen-Klassen und Angriffsvektoren — bewusst ohne STRIDE-Sortierung und ohne Verortung im System.

**Aufgabe:**

1. **Auswählen:** Geht die Liste durch und **pickt euch fünf bis acht Bedrohungen heraus**, die euch für den OWASP Juice Shop am plausibelsten erscheinen. Ihr müsst nicht alle bearbeiten — eine bewusste Auswahl ist Teil der Aufgabe.
2. **Verorten:** Überlegt für jede ausgewählte Bedrohung: *Wo* im System würde sie greifen? Tragt sie an der entsprechenden Stelle in Threat Dragon ein.
3. **Kategorisieren:** Welcher STRIDE-Kategorie ordnet ihr sie zu? Manche Einträge passen zu mehreren Kategorien — das ist gewollt.
4. Optional: Notiert euch in einem Satz, warum ihr genau diese Bedrohungen ausgewählt habt.

**Tipp:** Verschiedene Teams werden unterschiedliche Auswahl treffen. Genau das macht den Debrief am Ende interessant — schaut ruhig auch über den Tellerrand und wählt Bedrohungen aus, die euch fachlich reizen.

---

## Was am Ende stehen sollte

In eurem Threat-Dragon-Modell sind:

- Trust Boundaries an den richtigen Stellen
- Gefundene Bedrohungen aus Phase B (mindestens eine pro STRIDE-Kategorie)
- Fünf bis acht ausgewählte Bedrohungen aus der Inspirationsliste (aus Phase C)
- Jede Bedrohung verortet (welche Stelle im Diagramm) und kategorisiert (welche STRIDE-Kategorie)

---

## Wenn ihr nicht weiterkommt

- Schaut auf den Leitsatz: *"Traue dem Client niemals."* Was bedeutet das hier konkret?
- Fragt euch: Was wäre für **mich als Nutzer** schlimm, wenn es passiert? Was wäre für **den Shop-Betreiber** schlimm?
- Wenn alle Stricke reißen, schaut euch die Lösung im Ordner `solution` an.