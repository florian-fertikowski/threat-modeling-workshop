# Block 2 · Praxisteil 2: Broken Access Control

**Zeit:** ca. 20 Minuten

**Alternativ:** Verifiziert eure eigenen BAC-Hypothesen aus Block 1 frei.

**Hinweis zu den Lösungen:** Jede Challenge hat ein bis zwei einklappbare Hinweis-Blöcke und einen Lösungs-Block. Probiert es erst selbst — wer direkt die Lösung aufklappt, lernt am wenigsten.

---

# Block 2 · Praxisteil 2: Broken Access Control

**Zeit:** ca. 20 Minuten · **Werkzeug:** Browser-Devtools (Network-Tab, Application/Storage, Console), optional Burp Suite oder OWASP ZAP

---

## Vorgehen

Drei Pflicht-Challenges, dazu zwei Bonus-Challenges für die, die schnell durch sind.

**Pflicht:** Admin Section + 5-Sterne-Reviews löschen, View + Manipulate Basket, Payback Time
**Bonus:** Forged Review, Deluxe Fraud

**Alternativ:** Verifiziert eure eigenen BAC-Hypothesen aus Block 1 frei.

**Hinweis zu den Lösungen:** Jede Challenge hat ein bis zwei einklappbare Hinweis-Blöcke und einen Lösungs-Block. Probiert es erst selbst — wer direkt die Lösung aufklappt, lernt am wenigsten.

**Hinweis zur dritten Challenge:** Payback Time ist eine Business-Logic-Schwachstelle und damit deutlich anspruchsvoller als die ersten beiden. Es ist okay, wenn ihr sie nicht selbst löst — wir schauen sie am Ende kurz gemeinsam an.

---

### Lösche alle 5-Sterne-Reviews über die Admin-Übersicht

Verschaffe dir Zugang zum Admin-Bereich und lösche dort alle 5-Sterne-Reviews.

> **Verkettung:** Diese Challenge baut auf eurer "Login Admin"-Lösung aus dem Injection-Block auf. Ihr braucht zuerst einen Weg, um als Administrator eingeloggt zu sein.

<details>
<summary>Hinweis 1 — Wo ist der Admin-Bereich überhaupt?</summary>

Im normalen UI gibt es keinen "Admin"-Link für Customer-Accounts. Die Route existiert aber trotzdem — sie ist im Angular-Frontend definiert, das ihr als JavaScript-Bundle ausgeliefert bekommt.

Tipp: Öffne die Devtools → Sources (Chrome) bzw. Debugger (Firefox) → suche nach der `main.js`-Datei. Drücke `Strg+F` und such nach dem Begriff `admin`. Du findest dort einen Route-Eintrag mit `path:`.

</details>

<details>
<summary>Hinweis 2 — Ich kenne die URL, aber bekomme 403</summary>

Wenn du die URL aufrufst und einen "403 — You are not allowed to access this page!" bekommst — das ist normal. Die App prüft, ob du als Admin eingeloggt bist. Du bist es nicht.

Erinnerst du dich an die SQL Injection im Login-Formular? Damit landest du als erster User der Datenbank — und das ist der Admin.

</details>

<details>
<summary>Hinweis 3 — Ich bin auf der Admin-Seite, was jetzt?</summary>

Wenn du auf der Admin-Übersicht bist, siehst du eine Tabelle mit Customer Feedback. Daneben sind Trashcan-Symbole. Klicke darauf bei allen Einträgen mit 5 Sternen.

</details>

<details>
<summary>Lösung</summary>

**Schritt 1 — Admin-Route finden:** Öffne die Devtools (F12) → Sources (Chrome) bzw. Debugger (Firefox). Such die `main.js`-Datei im Network-Tab oder direkt im Source-Tree.

Mit `Strg+F` (Suchen-in-Datei) nach dem Begriff `admin` suchen. Du findest mehrere Treffer — einer davon sieht ungefähr so aus:

```javascript
{path:"administration", ...}
```

Damit weißt du: Die Admin-URL ist `http://localhost:3000/#/administration`.

**Schritt 2 — URL aufrufen (das schlägt fehl):** Geh als nicht-eingeloggter User oder als normaler Customer zu `http://localhost:3000/#/administration`. Du bekommst ein **403 — You are not allowed to access this page!**. Das ist eine **defense-in-depth**-Maßnahme, die in neueren Juice-Shop-Versionen nachgerüstet wurde — aber sie löst das Grundproblem nicht.

**Schritt 3 — Als Admin einloggen:** Geh zum Login-Formular und nutze die SQL Injection aus dem Injection-Block:

- E-Mail: `' OR 1=1--` (Leerzeichen nach dem Doppelminus!)
- Passwort: irgendwas Beliebiges

Du bist jetzt als `admin@juice-sh.op` eingeloggt.

**Schritt 4 — Admin-Bereich betreten:** Rufe erneut `http://localhost:3000/#/administration` auf. Jetzt funktioniert es. Damit ist die Challenge **"Admin Section"** gelöst.

**Schritt 5 — 5-Sterne-Reviews löschen:** Auf der Admin-Seite siehst du eine Tabelle "Customer Feedback". Filtere oder scrolle zu den Einträgen mit 5 Sternen. Klicke beim ersten 5-Sterne-Eintrag auf das Trashcan-Symbol. Wiederhole für alle anderen 5-Sterne-Reviews.

Sobald alle 5-Sterne-Reviews gelöscht sind, ist die Challenge **"Five-Star Feedback"** gelöst.

**Was passiert technisch:**

Diese Challenge demonstriert ein zentrales Konzept der Web-Security: **Schwachstellen verketten sich**. Eine einzelne Lücke ist oft begrenzt, aber zwei oder drei verkettete Lücken ergeben einen schweren Vorfall.

Hier sind es drei verkettete Schwächen:

1. **Information Disclosure im Frontend-Bundle** — Die Existenz und der Pfad der Admin-Route stehen für jeden lesbar in der `main.js`. Eine sichere Anwendung würde Admin-Funktionen nicht in das Standard-Bundle für Customer ausliefern, sondern als separates Bundle nach Rolle laden. Das spart Bandbreite und versteckt gleichzeitig die Existenz der Routen.

2. **SQL Injection im Login** — gibt euch unautorisierten Admin-Zugang.

3. **Schwache Autorisierung im Backend** — als (über SQL Injection legitimierter) Admin könnt ihr beliebige Reviews löschen, was die Bewertungs-Integrität des Shops zerstört.

</details>

---

### Öffne den Warenkorb eines anderen Nutzers und füge einen Artikel hinzu

Sieh dir den Warenkorb eines anderen Users an. Anschließend füge dort einen Artikel hinzu.

<details>
<summary>Hinweis 1 — Wie wird der Warenkorb identifiziert?</summary>

Wenn du auf `/#/basket` gehst, woher weiß die App, *welcher* Warenkorb gezeigt werden soll? Schau in den Devtools nach — speziell in **Application → Session Storage** (Chrome) bzw. **Storage → Session Storage** (Firefox).

Du findest dort einen `bid`-Wert. Was passiert, wenn du den änderst?

</details>

<details>
<summary>Hinweis 2 — View geht, wie füge ich aber was hinzu?</summary>

Wenn du den fremden Warenkorb angezeigt bekommst, ist View Basket schon gelöst. Für Manipulate Basket musst du einen Artikel in den *fremden* Warenkorb legen. Das macht das Frontend nicht automatisch — du musst den POST-Request manuell senden.

Tipp: Wenn du normal etwas in deinen eigenen Warenkorb legst, siehst du im Network-Tab den Request. Modifiziere ihn so, dass die `BasketId` auf einen anderen User zeigt.

Achtung: Das Backend prüft, ob die `BasketId` zu dir gehört. Du musst diese Validierung umgehen — eine Möglichkeit ist HTTP Parameter Pollution.

</details>

<details>
<summary>Lösung</summary>

#### Teil A: View Basket

**Schritt 1:** Eingeloggt sein, irgendwas in den Warenkorb legen.

**Schritt 2:** Devtools öffnen (F12) → **Application** (Chrome) bzw. **Storage** (Firefox) → **Session Storage** → `http://localhost:3000`.

**Schritt 3:** Du findest den Wert `bid` — das ist deine Basket-ID. Sie ist meist eine kleine Zahl (1, 2, 3...).

**Schritt 4:** Doppelklick auf den Wert, ändere ihn auf eine andere Zahl (z.B. von `1` auf `2`).

**Schritt 5:** Navigiere zu `http://localhost:3000/#/basket` und drücke F5 (Reload).

Du siehst nun den Warenkorb eines anderen Users. Challenge **"View Basket"** gelöst.

#### Teil B: Manipulate Basket

**Schritt 1:** Notiere dir deine eigene Basket-ID (z.B. `1`) und die des fremden Users (z.B. `2`).

**Schritt 2:** In den Devtools → Network-Tab. Lege ein Produkt in deinen eigenen Warenkorb. Du siehst einen POST-Request an `/api/BasketItems` mit folgendem Body:

```json
{ "ProductId": 1, "BasketId": "1", "quantity": 1 }
```

**Schritt 3:** Rechtsklick auf den Request → "Edit and Resend" (Firefox) oder als curl/fetch in Chrome.

**Schritt 4:** Ändere `BasketId` auf die fremde ID (z.B. `2`) und sende:

```json
{ "ProductId": 14, "BasketId": "2", "quantity": 1 }
```

Du bekommst vermutlich eine Fehlermeldung wie `{"error": "Invalid BasketId"}` — das Backend prüft, ob die Basket dem aufrufenden User gehört. Aber: Diese Prüfung lässt sich mit HTTP Parameter Pollution umgehen.

**Schritt 5:** Sende den Request mit *zwei* `BasketId`-Feldern:

```json
{ "ProductId": 14, "BasketId": "1", "quantity": 1, "BasketId": "2" }
```

Der Validierungs-Code im Backend liest den ersten Wert (deine eigene ID, akzeptiert), aber der Insert in die Datenbank nutzt den zweiten Wert (fremde ID). Das Produkt landet im fremden Warenkorb. Challenge **"Manipulate Basket"** gelöst.

**Was passiert technisch:**

Beides sind klassische **IDOR**-Schwachstellen (*Insecure Direct Object Reference*):

1. Bei "View Basket" speichert die App die Basket-ID im Session Storage und vertraut diesem clientseitigen Wert. Sie prüft serverseitig nicht, ob die angefragte Basket-ID zum aktuellen User gehört.

2. Bei "Manipulate Basket" zeigt sich eine andere Variante: Die Validierung existiert, aber sie liest nicht denselben Parameter wie der Insert. **HTTP Parameter Pollution** ist eine subtile Klasse von Schwachstellen — wenn verschiedene Code-Pfade unterschiedliche Werte für denselben Parameter sehen, lassen sich Validierungen umgehen.

</details>

---

### Werde reich (Payback Time)

Platziere eine Bestellung, die dich reich macht — die App soll dir Geld gutschreiben, statt dir welches abzubuchen.

<details>
<summary>Hinweis 1 — Wo könnte ich ansetzen?</summary>

Schau dir die Berechnung des Gesamtpreises im Warenkorb genau an. `Gesamtpreis = Stückpreis × Menge`. Was passiert, wenn einer der beiden Werte ungewöhnlich ist?

Tipp: Das Frontend lässt dich nur positive Mengen eingeben. Aber das Backend prüft das nicht zwingend mit.

</details>

<details>
<summary>Hinweis 2 — Wie sende ich eine negative Menge?</summary>

Das UI hat einen `+`- und `−`-Button bei der Mengenanpassung im Warenkorb. Bei jeder Änderung wird ein PUT-Request an `/api/BasketItems/{id}` gesendet, mit `quantity` im Body.

Fang diesen Request ab und ändere die Menge auf eine negative Zahl.

</details>

<details>
<summary>Lösung</summary>

**Schritt 1:** Lege ein Produkt in deinen Warenkorb (irgendeins — am besten ein teures, dann wirst du richtig reich).

**Schritt 2:** Geh zu `/#/basket`. Devtools öffnen → Network-Tab.

**Schritt 3:** Klick auf den `+`- oder `−`-Button bei dem Produkt, um die Menge zu ändern. Du siehst im Network-Tab einen PUT-Request an `/api/BasketItems/{id}` mit folgendem Body:

```json
{ "quantity": 2 }
```

**Schritt 4:** Rechtsklick auf den Request → "Edit and Resend" (Firefox) bzw. Copy-as-fetch in Chrome. Ändere die Menge auf eine deutlich negative Zahl:

```json
{ "quantity": -100 }
```

Sende den Request. Du bekommst eine `200 OK`-Antwort.

**Schritt 5:** Reload der Basket-Seite. Du siehst jetzt eine negative Menge und einen entsprechend negativen Gesamtpreis.

**Schritt 6:** Klick auf Checkout. Wähle eine Lieferadresse (falls nötig, eine anlegen). Wähle "Zahlung mit Wallet" als Zahlungsmethode — auch wenn dein Wallet leer ist.

**Schritt 7:** Klick auf "Place your order and pay". Die Challenge ist gelöst — dein Wallet hat jetzt einen positiven Saldo, weil die App dir das (negative) Bestellgeld gutgeschrieben hat.

**Was passiert technisch:**

Das ist eine **Business-Logic-Schwachstelle**, die sich von den klassischen BAC-Mustern unterscheidet:

- Es gibt keine klassische "fehlende Autorisierung" — du *darfst* deinen eigenen Warenkorb ändern.
- Aber: Es gibt keine **plausibilitätsbasierte Validierung** der Geschäftslogik. Mengen sollten positiv sein. Gesamtpreise sollten positiv sein. Der Code hat diese impliziten Annahmen, prüft sie aber nicht.

</details>

---

### Forged Review (Bonus)

Editiere die Review eines anderen Users — oder poste eine Review unter fremdem Namen.

<details>
<summary>Hinweis</summary>

Wenn du als eingeloggter User eine Review schreibst oder editierst, wird ein Request mit Author-Information gesendet. Schau im Network-Tab nach. Was passiert, wenn du die Author-Information manipulierst?

</details>

<details>
<summary>Lösung</summary>

**Schritt 1:** Logge dich als beliebiger User ein.

**Schritt 2:** Öffne ein Produkt, das eine bestehende Review von dir selbst hat (notfalls schnell eine schreiben).

**Schritt 3:** Klicke auf den "Edit"-Button bei deiner Review. Devtools → Network-Tab.

**Schritt 4:** Speichere die Review. Im Network-Tab siehst du einen PUT-Request an `/rest/products/reviews` mit folgendem Body:

```json
{ "id": "<review-id>", "message": "...", "author": "<deine-email>" }
```

**Schritt 5:** Rechtsklick auf den Request → "Edit and Resend". Ändere das `author`-Feld auf eine andere E-Mail, z.B. `admin@juice-sh.op`:

```json
{ "id": "<review-id>", "message": "...", "author": "admin@juice-sh.op" }
```

Sende. Die Review erscheint jetzt unter dem Namen des anderen Users. Challenge gelöst.

**Was passiert technisch:**

Klassische **IDOR mit Identity-Spoofing**: Das Backend übernimmt das `author`-Feld aus dem Request, ohne zu prüfen, ob es mit dem authentifizierten User übereinstimmt. Eine Review sollte immer dem eingeloggten User zugeordnet werden — die Identität sollte serverseitig aus dem Session-Token kommen, nicht aus dem Request-Body.

</details>

---

### Deluxe Fraud (Bonus)

Erhalte eine Deluxe-Mitgliedschaft, ohne dafür zu bezahlen.

<details>
<summary>Hinweis 1 — Wo ist der Upgrade-Endpoint?</summary>

Unter `/#/payment/deluxe` kannst du dich auf Deluxe upgraden. Es gibt mehrere Zahlungsmethoden. Schau dir an, wie die App die Zahlung verarbeitet — welcher Endpoint wird beim Bezahlen aufgerufen?

</details>

<details>
<summary>Hinweis 2 — Wallet ist leer, was nun?</summary>

Wenn dein Wallet leer ist, kannst du den "Pay with Wallet"-Button gar nicht klicken (er ist disabled). Aktiviere ihn über die Devtools — dann kannst du den Request triggern und im Network-Tab inspizieren. Anschließend kannst du den Request modifizieren.

</details>

<details>
<summary>Lösung</summary>

**Schritt 1:** Erstelle einen frischen Account (damit der Wallet-Saldo garantiert leer ist).

**Schritt 2:** Gehe zu `http://localhost:3000/#/payment/deluxe`.

**Schritt 3:** Devtools öffnen. Im Elements/Inspector-Tab den "Pay with Wallet"-Button finden. Das `disabled`-Attribut entfernen (Rechtsklick → Edit attribute oder direkt aus dem DOM löschen).

**Schritt 4:** Network-Tab öffnen. Klick auf den nun aktivierten Wallet-Pay-Button. Du siehst einen POST-Request — typischerweise an `/rest/deluxe-membership` mit folgendem Body:

```json
{ "paymentMode": "wallet" }
```

Die Antwort ist ein Fehler ("Insufficient funds" o.ä.).

**Schritt 5:** Rechtsklick auf den Request → "Edit and Resend". Ändere `paymentMode` auf einen leeren String:

```json
{ "paymentMode": "" }
```

Sende. Du bekommst eine erfolgreiche Antwort, und dein Account ist nun Deluxe-Mitglied — ohne dass jemals tatsächlich gezahlt wurde.

**Was passiert technisch:**

Das ist eine besonders schöne Logic-Flaw-Schwachstelle: Der Backend-Code prüft *welche* Zahlungsmethode genutzt wird (Wallet, Karte, etc.) und führt die jeweilige Abbuchungs-Logik aus. Wenn der `paymentMode` aber leer ist, gibt es keinen Switch-Case-Treffer — und die Abbuchungs-Logik wird komplett übersprungen. Trotzdem läuft der nachgelagerte Code weiter und gibt dem User die Deluxe-Rolle.

Der Fehler ist eigentlich offensichtlich, wenn man drauf schaut: "Wenn keine Zahlungsmethode angegeben ist, mache trotzdem das Upgrade" ist offensichtlich unsinnig. Aber im Code sieht das oft so aus, dass der Default-Pfad einfach übersehen wird.

</details>
