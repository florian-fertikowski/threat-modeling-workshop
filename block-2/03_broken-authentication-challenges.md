# Block 2 · Praxisteil 3: Authentication Failures

**Zeit:** ca. 10 Minuten

**Alternativ:** Verifiziert eure eigenen Auth-Hypothesen aus Block 1 frei.

**Hinweis zu den Lösungen:** Wieder klappbare Hinweise und Lösungen. Probiert es erst selbst.

---

## Pflicht 1: Password Strength

Logge dich als Administrator ein **ohne SQL Injection** — also mit dem echten Passwort.

<details>
<summary>Hinweis 1 — Wie komme ich an das Passwort?</summary>

Es gibt mehrere Wege:

1. **Erraten** — Welche Passwörter sind im Real-World immer wieder die häufigsten? Denk nach: Was nimmt jemand, der einen Admin-Account schnell aufsetzt? Versuche typische Default-Passwörter aus.

2. **Brute-Force über die Browser-Console** — Du kannst das auch automatisieren. Mit JavaScript in der Devtools-Console kannst du eine Wortliste runterladen und jedes Passwort gegen den Login-Endpoint testen. Das ist ein realistischer Angriffs-Workflow.

</details>

<details>
<summary>Lösung</summary>

### Weg 1: Manuell raten

**Schritt 1:** Geh zum Login-Formular: `http://localhost:3000/#/login`.

**Schritt 2:** Login mit:

- E-Mail: `admin@juice-sh.op`
- Passwort: `admin123`

Das war's. Die Challenge ist gelöst.

### Weg 2: Brute-Force über die Browser-Console

Wer das Passwort nicht erraten will (oder einen realistischeren Angriff demonstrieren möchte), kann es über ein kurzes JavaScript-Skript in der Devtools-Console automatisieren.

**Schritt 1:** Devtools öffnen (F12) → Console-Tab.

**Schritt 2:** Folgendes Skript einfügen und mit Enter ausführen:

```javascript
const res = await fetch('https://raw.githubusercontent.com/danielmiessler/SecLists/master/Passwords/Default-Credentials/default-passwords.txt');
const text = await res.text();
const passwords = text.trim().split('\n');

for (const pw of passwords) {
    const login = await fetch('/rest/user/login', {
        method: 'POST',
        headers: {'Content-Type': 'application/json'},
        body: JSON.stringify({email: 'admin@juice-sh.op', password: pw})
    });

    if (login.status === 200) {
        console.log(`Eingeloggt: ${pw}`);
        break;
    }
}
```

Das Skript:
1. Lädt die [SecLists Top 100](https://github.com/danielmiessler/SecLists/blob/master/Passwords/Common-Credentials/10-million-password-list-top-100.txt) Passwortliste von GitHub herunter
2. Iteriert durch jedes Passwort
3. Versucht den Login per POST an `/rest/user/login`
4. Bricht beim ersten erfolgreichen Login ab und gibt das Passwort aus

**Was passiert technisch — und warum das ein Problem ist:**

Das Passwort `admin123` ist eines der häufigsten schwachen Passwörter weltweit. Es steht auf praktisch jeder Brute-Force-Wortliste ziemlich weit oben. Wie ihr im Skript seht: Ein automatisierter Brute-Force-Angriff findet es in unter einer Sekunde.

Verschärft wird das durch zwei zusätzliche Probleme im Juice Shop:

1. **Kein Rate Limiting auf dem Login-Endpoint.** Ihr habt eben gerade 100 Login-Versuche in einer Sekunde geschickt — und die App hat keinen einzigen davon blockiert. In produktiven Systemen sollte nach 5–10 Fehlversuchen entweder eine Sperre, ein CAPTCHA oder ein Delay einsetzen.

2. **Passwörter werden als unsalted MD5 gespeichert.** Selbst wenn das Passwort komplexer wäre, würde ein Datenbankleck es trivial knackbar machen — MD5-Hashes sind in Rainbow Tables erfasst, und ohne Salt funktionieren die Tables direkt. Moderne Verfahren wären Argon2id oder bcrypt mit Salt.

</details>

---

## Bjoern's Favorite Pet

Setze das Passwort von Bjoerns OWASP-Account zurück, indem du die Antwort auf seine Security Question findest.

<details>
<summary>Hinweis 1 — Welche E-Mail-Adresse soll ich nutzen?</summary>

Bjoern hat im Juice Shop **mehrere Accounts**. Die E-Mail-Adressen findet ihr im **Admin-Panel** unter `/#/administration` — wenn ihr nach Password Strength (oder über die Admin-Challenge aus dem BAC-Block) als Admin eingeloggt seid. Dort listet die App alle registrierten User auf.

Bjoern taucht mit drei E-Mails auf:

- `bjoern.kimminich@gmail.com`
- `bjoern@juice-sh.op`
- `bjoern@owasp.org`

Welche ist die richtige? Jede Adresse hat eine andere Security Question hinterlegt. Probiert sie über `/#/forgot-password` durch — diese Challenge fragt nach *"Name of your favorite pet?"*.

</details>

<details>
<summary>Hinweis 2 — Wer ist Bjoern überhaupt?</summary>

Bjoern ist eine echte Person. Eine kurze Google-Suche nach *"Bjoern OWASP Juice Shop"* führt sofort zu **Bjoern Kimminich**, dem Maintainer des Projekts. Über seinen Namen findet ihr sein Twitter-Profil unter `@bkimminich`, seinen GitHub-Account `bkimminich`, YouTube-Vorträge etc.

Das ist genau der Trick der Challenge: Wenn ihr den echten Namen einer Person habt, ist ihre Online-Präsenz meist mit ein paar Klicks zu finden. Geht durch sein Twitter oder seine Vorträge und sucht nach Hinweisen auf sein Haustier.

</details>

<details>
<summary>Lösung</summary>

**Schritt 1 — E-Mail-Adresse finden:** Logge dich als Admin ein (Password Strength oder SQL Injection aus dem Injection-Block) und navigiere zu `http://localhost:3000/#/administration`. Du siehst eine Tabelle mit allen registrierten Usern. Bjoern hat drei E-Mail-Adressen — wir brauchen seinen OWASP-Account.

**Schritt 2 — Richtige Security Question finden:** Geh zu `http://localhost:3000/#/forgot-password` und probiere die drei E-Mails durch:

- `bjoern.kimminich@gmail.com` → andere Frage
- `bjoern@juice-sh.op` → andere Frage (ZIP-Code als Teenager)
- `bjoern@owasp.org` → *"Name of your favorite pet?"* ✓

**Schritt 3 — Bjoern identifizieren:** Eine Google-Suche nach `"Bjoern" OWASP Juice Shop` führt sofort zu **Bjoern Kimminich**, dem Maintainer des OWASP-Juice-Shop-Projekts. Er ist eine reale, öffentliche Person mit Twitter-Account, GitHub-Profil, YouTube-Vorträgen.

**Schritt 4 — OSINT zur Haustier-Antwort:**

- Bjoerns Twitter-Profil: `@bkimminich` — geh durch seine Tweets und Bilder. Auf vielen Fotos siehst du eine Katze, und in mehreren Tweets steht ihr Name.
- Alternativ: Schau einen seiner Juice-Shop-Vorträge auf YouTube — er erwähnt den Namen sogar live im Demo (siehe seinen [BeNeLux Day 2018 Talk](https://youtu.be/Lu0-kDdtVf4?t=239) bei Minute 3:59, wo er einen Account mit der echten Antwort registriert).

Antwort: **Zaya** (Name seiner Familienkatze).

**Schritt 5 — Passwort zurücksetzen:** Im Forgot-Password-Formular `Zaya` als Antwort eingeben, ein neues Passwort setzen, bestätigen. Challenge gelöst.

**Was passiert technisch — und warum das ein wichtiges Lernmoment ist:**

Diese Challenge ist **nicht** eine technische Schwachstelle im klassischen Sinne. Der Code des Juice Shop arbeitet exakt so, wie er sollte: Security Question prüfen, neues Passwort setzen, fertig. Das Problem liegt **im Konzept der Security Question selbst**.

Security Questions sollten Antworten haben, die:

- nur der Nutzer kennt
- sich nicht ändern
- nicht öffentlich auffindbar sind
- nicht erraten werden können

Das Problem: Die meisten typischen Fragen ("Name deines ersten Haustieres", "Mädchenname deiner Mutter", "Stadt, in der du geboren bist") verletzen mindestens eine dieser Anforderungen. Solche Informationen sind oft öffentlich — auf Social Media, in Online-Profilen, in Interviews, manchmal sogar im Telefonbuch.

</details>

---