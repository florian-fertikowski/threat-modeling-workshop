# Schwachstellen & Angriffsvektoren – Übungsliste (Teil C)

---

1. **Path Traversal (Directory Traversal)**
   Über manipulierte Pfadangaben wie `../../etc/passwd` versucht ein Angreifer, aus dem vorgesehenen Verzeichnis auszubrechen und auf Dateien zuzugreifen, die er eigentlich nicht sehen dürfte.

2. **Credential Stuffing**
   Aus früheren Datenlecks bekannte Benutzername-/Passwort-Kombinationen werden automatisiert an der Login-Maske durchprobiert, in der Hoffnung, dass jemand dasselbe Passwort wiederverwendet hat.

3. **Zip Bomb**
   Eine winzige Archivdatei, die sich beim Entpacken auf ein Vielfaches ihrer Größe aufbläht (z. B. wenige KB → mehrere GB) und so Speicher oder Festplatte des verarbeitenden Systems überlastet.

4. **Cross-Site Scripting (XSS)**
   Schädlicher JavaScript-Code wird in eine Seite eingeschleust (z. B. über ein Eingabefeld) und im Browser anderer Nutzer ausgeführt, etwa um Sitzungsdaten abzugreifen oder Aktionen im Namen des Opfers auszuführen.

5. **Insecure Direct Object Reference (IDOR)**
   Eine ID in der URL oder im Request (z. B. `?order=1002`) lässt sich einfach hochzählen, um auf fremde Datensätze (Bestellungen, Profile, Dateien) zuzugreifen, weil keine Berechtigungsprüfung stattfindet.

6. **Man in the Middle (MitM)**
   Ein Angreifer schaltet sich in die Kommunikation zwischen Client und Server (z. B. im selben WLAN) und kann den Datenverkehr mitlesen oder manipulieren, sofern dieser nicht ausreichend verschlüsselt ist.

7. **SQL Injection**
   Über Eingabefelder wird zusätzlicher SQL-Code in eine Datenbankabfrage eingeschleust, um Login-Prüfungen zu umgehen, Daten auszulesen oder zu verändern.

8. **Fehlende / unzureichende Protokollierung (Insufficient Logging)**
   Sicherheitsrelevante Aktionen (fehlgeschlagene Logins, Änderungen, Zugriffe) werden gar nicht oder nur lückenhaft aufgezeichnet, sodass Angriffe später nicht nachvollzogen oder einer Person zugeordnet werden können.

9. **Server-Side Request Forgery (SSRF)**
   Der Angreifer bringt den Server dazu, in seinem Namen Anfragen an interne oder externe Adressen zu stellen (z. B. an interne Dienste oder Cloud-Metadaten-Endpunkte), die von außen normalerweise nicht erreichbar wären.

10. **Brute Force / fehlendes Rate Limiting**
    Passwörter, Tokens oder Coupon-Codes werden in sehr hoher Geschwindigkeit durchprobiert, weil das System die Anzahl der Versuche nicht begrenzt.

11. **Cross-Site Request Forgery (CSRF)**
    Ein eingeloggter Nutzer wird (z. B. über einen präparierten Link oder eine Webseite) dazu gebracht, ungewollt eine Aktion in der Anwendung auszulösen, weil der Request automatisch mit seiner gültigen Sitzung mitgeschickt wird.

12. **Sensitive Data Exposure / zu gesprächige Fehlermeldungen**
    Vertrauliche Informationen (Stacktraces, interne Pfade, DB-Strukturen, Schlüssel, personenbezogene Daten) landen in Fehlermeldungen, Quelltext oder API-Antworten und sind so für Angreifer einsehbar.

13. **Session Fixation / Session Hijacking**
    Ein Angreifer übernimmt eine fremde Sitzung – entweder indem er dem Opfer eine bekannte Session-ID unterschiebt oder indem er eine gültige Session-ID abgreift und damit als das Opfer auftritt.

14. **Resource Exhaustion (Ressourcenerschöpfung)**
    Durch besonders viele oder besonders aufwändige Anfragen werden CPU, Arbeitsspeicher, Datenbankverbindungen oder Bandbreite so weit verbraucht, dass die Anwendung für legitime Nutzer langsam oder nicht mehr erreichbar ist.

15. **Insecure Deserialization**
    Vom Nutzer kontrollierte serialisierte Daten (z. B. ein Objekt aus einem Cookie oder Token) werden ungeprüft wieder in ein Objekt umgewandelt, wodurch sich im Extremfall Logik manipulieren oder Code ausführen lässt.

16. **Regular Expression Denial of Service (ReDoS)**
    Eine speziell konstruierte Eingabe bringt einen ineffizienten regulären Ausdruck dazu, extrem lange zu rechnen, und blockiert damit den verarbeitenden Prozess.

17. **XML External Entity (XXE)**
    Ein XML-Parser, der externe Entitäten verarbeitet, wird über manipuliertes XML dazu gebracht, lokale Dateien einzulesen oder interne Netzwerkanfragen auszulösen.

18. **Open Redirect**
    Eine Weiterleitungsfunktion akzeptiert ein beliebiges Ziel als Parameter, sodass ein vertrauenswürdig aussehender Link der Anwendung das Opfer auf eine vom Angreifer kontrollierte Seite leitet (z. B. für Phishing).

19. **Command Injection / Remote Code Execution (RCE)**
    Nutzereingaben fließen ungefiltert in einen System- oder Shell-Befehl ein, sodass ein Angreifer eigene Befehle auf dem Server ausführen kann.

20. **JWT-Manipulation / schwache Token-Signatur**
    Ein JSON Web Token wird verändert (z. B. die Rolle von `customer` auf `admin`), und weil die Signatur fehlt, falsch geprüft oder mit einem schwachen Schlüssel erzeugt wurde, akzeptiert der Server das manipulierte Token.

21. **Unrestricted File Upload**
    Es können beliebige Dateitypen hochgeladen werden (z. B. ausführbare Skripte statt Bilder), die der Server anschließend speichert oder ausführt.

22. **Directory Listing / Information Leakage**
    Ein falsch konfigurierter Server zeigt den Inhalt von Verzeichnissen oder gibt versteckte Dateien preis (z. B. Backups, Konfigurationen), die Rückschlüsse auf den Aufbau der Anwendung erlauben.

23. **Broken Access Control / Privilege Escalation**
    Funktionen oder Endpunkte, die eigentlich nur für bestimmte Rollen gedacht sind (z. B. ein Admin-Bereich), sind ohne ausreichende Prüfung auch für normale Nutzer erreichbar.

24. **Log Injection / Log Forging**
    Durch eingeschmuggelte Zeilenumbrüche oder gefälschte Einträge in Eingaben, die später ins Log geschrieben werden, manipuliert ein Angreifer die Protokolle, um Spuren zu verschleiern oder falsche Einträge unterzuschieben.

25. **Hardcoded / Default-Zugangsdaten**
    Fest im Code hinterlegte oder unveränderte Standard-Zugangsdaten (z. B. `admin/admin`) ermöglichen unbefugten Zugriff, wenn sie nicht entfernt bzw. geändert wurden.