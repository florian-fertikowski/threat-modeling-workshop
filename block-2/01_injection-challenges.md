# Block 2 · Praxisteil 1: Injection

**Zeit:** ca. 15 Minuten

**Alternativ:** Verifiziert eure eigenen BAC-Hypothesen aus Block 1 frei.

**Hinweis zu den Lösungen:** Jede Challenge hat zwei oder drei einklappbare Blöcke — *Hinweise* sind sanfte Schubse in die richtige Richtung, *Lösung* zeigt den konkreten Weg.

---

### Login Admin

Logge dich als Administrator ein, ohne dessen Passwort zu kennen.

<details>
<summary>Hinweis 1 — Wo ansetzen?</summary>

Schau dir das Login-Formular genau an. Was passiert, wenn du etwas Ungewöhnliches ins E-Mail-Feld eingibst, z.B. ein einfaches Anführungszeichen `'`?

Versuche es und beobachte die Antwort im Network-Tab der Devtools. Was sagt die Fehlermeldung über die Backend-Verarbeitung aus?

</details>

<details>
<summary>Hinweis 2 — Welche Art von Schwachstelle?</summary>

Die Fehlermeldung verrät, dass die App eine SQL-Query baut, in die deine Eingabe direkt eingefügt wird. Damit ist klar: Hier kannst du SQL Injection probieren.

Überlege: Wie würde die SQL-Query intern aussehen? Etwa so:

```sql
SELECT * FROM Users WHERE email = 'DEINE_EINGABE' AND password = '...'
```

Wie kannst du deine Eingabe so gestalten, dass die Bedingung immer wahr ist?

</details>

<details>
<summary>Lösung</summary>

**Payload:** Im E-Mail-Feld eingeben:

```
' OR 1=1--
```

(Wichtig: Leerzeichen nach dem Doppelminus, sonst wird der Kommentar nicht erkannt.)

Im Passwort-Feld: irgendwas Beliebiges, z.B. `egal`.

**Was passiert technisch:**

Die App baut intern die Query unsicher aus String-Konkatenation:

```sql
SELECT * FROM Users 
WHERE email = '' OR 1=1--' 
AND password = '...'
```

Der Payload macht drei Dinge auf einmal:
1. Das einfache Anführungszeichen am Anfang schließt das `email`-Feld vorzeitig.
2. `OR 1=1` ist eine Bedingung, die immer wahr ist — der Filter ist damit faktisch deaktiviert.
3. `--` (mit Leerzeichen!) kommentiert den Rest der Query aus, inklusive der Passwortprüfung.

Die Datenbank gibt alle Nutzer zurück, und die App loggt dich als den ersten ein — und das ist der Administrator (User-ID 1).

**Verifikation:** Du landest auf der Startseite. Oben rechts steht jetzt `admin@juice-sh.op`. Damit ist die Challenge gelöst — du siehst eine Notification rechts unten.

</details>

---

### DOM XSS

Führe einen DOM-basierten XSS-Angriff mit dem Payload `<iframe src="javascript:alert(\`xss\`)">` durch.

<details>
<summary>Hinweis 1 — Wo könnte das funktionieren?</summary>

DOM-XSS entsteht, wenn JavaScript Eingaben des Users in das DOM einfügt, ohne sie zu sanitisieren. Wo gibt der User Text ein, der direkt auf der Seite angezeigt wird?

Tipp: Schau dir die Suchfunktion an (Lupensymbol oben rechts).

</details>

<details>
<summary>Lösung</summary>

**Payload in das Suchfeld eingeben:**

```html
<iframe src="javascript:alert(`xss`)">
```

Drücke Enter. Es erscheint ein Alert-Popup mit dem Text "xss". Challenge gelöst.

**Was passiert technisch:**

Die Suchfunktion nimmt deinen Eingabetext und fügt ihn direkt in das DOM ein — vermutlich über `innerHTML` oder eine ähnliche unsichere Methode. Statt deinen Text als String anzuzeigen, interpretiert der Browser das `<iframe>`-Element als echten HTML-Tag und führt das eingebettete JavaScript aus.

In sicheren Frameworks (z.B. modernes React, Vue, Angular ohne `bypassSecurityTrust*`) wird so etwas standardmäßig escaped — `<` würde zu `&lt;` werden und als reiner Text angezeigt. Der Juice Shop tut das hier nicht.

</details>

---

### Reflected XSS

Führe einen Reflected-XSS-Angriff aus.

<details>
<summary>Hinweis 1 — Welche URLs nehmen Parameter entgegen?</summary>

Reflected XSS funktioniert, wenn ein URL-Parameter direkt in der Seite gerendert wird, ohne sanitisiert zu werden. Welche Funktion im Juice Shop nimmt einen User-Input und zeigt ihn auf einer eigenen Ergebnisseite an?

Tipp: Bestell etwas. Im Bestellverlauf gibt es eine Funktion "Order tracken".

</details>

<details>
<summary>Hinweis 2 — Wo genau zeigt sich der Parameter?</summary>

Wenn du eine Bestellung trackst, landest du auf einer URL wie `/#/track-result?id=<order-id>`. Probiere, statt der echten Order-ID einen Test-String einzugeben, z.B. `<h1>test</h1>`. Wird der HTML interpretiert oder als Text angezeigt?

</details>

<details>
<summary>Lösung</summary>

**URL manuell anpassen:**

```
http://localhost:3000/#/track-result?id=<iframe src="javascript:alert(`xss`)">
```

Oder, falls die URL-Encoding stört, den Payload im UI-Eingabefeld der Track-Order-Funktion eingeben.

**Was passiert technisch:**

Der `id`-Parameter wird direkt auf der Track-Result-Seite in das DOM eingefügt. Da keine Sanitization stattfindet, wird das eingebettete `<iframe>` als echtes HTML-Element interpretiert und das `javascript:`-URL-Schema im `src`-Attribut führt zur Code-Ausführung.

**Unterschied zu DOM XSS:** Bei DOM XSS findet die Manipulation client-seitig im JavaScript der App statt. Bei Reflected XSS wird der Payload an den Server geschickt, von dort zurückreflektiert und beim Rendern ausgeführt. In Single-Page-Apps wie Juice Shop verschwimmt diese Trennung — technisch ist es bei diesem Juice-Shop-Beispiel eigentlich auch eine Form von DOM XSS, weil alles client-side passiert.

</details>

---

### Login Jim (Bonus)

Logge dich als der Nutzer "Jim" ein, ohne sein Passwort zu kennen.

<details>
<summary>Hinweis 1 — Wie heißt Jim mit E-Mail?</summary>

Du brauchst Jims E-Mail-Adresse, um ihn gezielt anzugreifen. Schau dir Produkt-Bewertungen an — manchmal stehen dort Namen oder Hinweise auf E-Mail-Adressen. Die typische E-Mail-Struktur im Juice Shop ist `<vorname>@juice-sh.op`.

</details>

<details>
<summary>Hinweis 2 — Wie nutze ich SQL Injection für einen bestimmten User?</summary>

Bei `Login Admin` hast du den ersten User der Tabelle bekommen. Jetzt willst du gezielt Jim. Statt `' OR 1=1--` versuche, das `email`-Feld so zu manipulieren, dass die Bedingung auf Jim zutrifft, aber die Passwortprüfung umgangen wird.

Tipp: Was passiert, wenn du nach der E-Mail einen SQL-Kommentar einfügst?

</details>

<details>
<summary>Lösung</summary>

**Payload im E-Mail-Feld:**

```
jim@juice-sh.op'--
```

Im Passwort-Feld: irgendwas Beliebiges.

**Was passiert technisch:**

Die Query wird so:

```sql
SELECT * FROM Users 
WHERE email = 'jim@juice-sh.op'--' 
AND password = '...'
```

Das einfache Anführungszeichen schließt den `email`-String sauber ab, und `--` kommentiert alles danach aus — inklusive der Passwortprüfung. Die Datenbank gibt also genau den User mit der E-Mail `jim@juice-sh.op` zurück, ohne dass das Passwort jemals verglichen wird.

**Verifikation:** Oben rechts steht jetzt Jims E-Mail. Die Challenge wird automatisch als gelöst markiert.

**Warum funktioniert das `OR 1=1--`-Pattern hier nicht?** Es würde funktionieren, aber nicht spezifisch Jim einloggen — sondern wieder den Admin (den ersten User der Tabelle).

</details>


---

### Persisted XSS via API (Bonus)

Speichere einen XSS-Payload dauerhaft am API-Endpoint, ohne über das Frontend zu gehen.

<details>
<summary>Hinweis 1 — Welche Tools brauche ich?</summary>

Du brauchst **kein** Burp Suite. Die Browser-Devtools reichen — speziell der Network-Tab mit der "Edit and Resend"-Funktion (Firefox) bzw. "Copy as fetch" (Chrome).

</details>

<details>
<summary>Hinweis 2 — Welcher Endpoint ist verwundbar?</summary>

Schau dir an, was beim Aufruf eines Produkts im Network-Tab passiert — du siehst einen `GET /api/Products/<id>`-Request. Was wäre, wenn dieselbe URL auch andere HTTP-Methoden akzeptiert? PUT zum Beispiel?

Das Frontend bietet keine Möglichkeit, ein Produkt zu bearbeiten — aber das Backend könnte den Endpoint trotzdem unauthorisiert offen haben.

</details>

<details>
<summary>Lösung</summary>

**Voraussetzung:** Du bist eingeloggt (egal als welcher User — auch als normaler Customer).

**Schritt 1:** Öffne die Devtools (F12) → Network-Tab.

**Schritt 2:** Klicke im Shop irgendein Produkt an, damit ein `GET /api/Products/<id>`-Request rausgeht. Den brauchst du als Vorlage, weil er bereits den Auth-Token im Header hat.

**Schritt 3:** Rechtsklick auf den Request → "Edit and Resend" (Firefox) bzw. in Chrome "Copy → Copy as fetch" und in die Console einfügen.

**Schritt 4:** Im Edit-Resend-Dialog folgendes anpassen:

    - **URL:** `http://localhost:3000/api/Products/5`
- **Methode:** `PUT` (statt `GET`)
- **Header hinzufügen:** `Content-Type: application/json`
- **Body:**

```json
{
  "description": "<iframe src=\"javascript:alert(`xss`)\">"
}
```

**Schritt 5:** Senden. Du solltest eine `200 OK`-Antwort mit dem aktualisierten Produkt zurückbekommen.

**Was passiert technisch:**

Das Frontend hat clientseitige Validierung, die HTML im `description`-Feld blockiert. Diese Validierung wird aber serverseitig nicht erzwungen — der API-Endpoint `PUT /api/Products/:id` akzeptiert beliebige Inhalte und speichert sie ohne Sanitization in der Datenbank. Wenn das Produkt danach gerendert wird, kommt der Schadcode aus der Datenbank — eine klassische **Stored XSS** (auch *Persisted XSS* genannt).

Im Gegensatz zu DOM- oder Reflected-XSS muss kein Opfer eine spezielle URL anklicken — die Schwachstelle wirkt für jeden, der die Produktseite aufruft. Das ist gleichzeitig ein Beispiel für **Broken Access Control**: Der `PUT`-Endpoint sollte nur für Administratoren erreichbar sein, ist es aber für jeden authentifizierten User. Eine einzelne Lücke, zwei OWASP-Top-10-Kategorien.

</details>
