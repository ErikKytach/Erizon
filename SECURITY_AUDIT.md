# Security Audit & Hardening Report: Erizon E-Commerce Platform

* **Autor:** Erik Kytach
* **Projekt:** Erizon (Full-Stack Django E-Commerce)
* **Ziel:** Überprüfung der Anwendung auf Schwachstellen nach OWASP Top 10 und Implementierung von Hardening-Maßnahmen für den Produktivbetrieb.
* **Umgebung:** Isolierte Staging-Umgebung / Code-Analyse (White-Box).

---

## 1. Konfigurations- und Transport-Sicherheit (OWASP A05:2021)

Die Produktionskonfiguration der Anwendung wurde durch den Befehl `manage.py check --deploy` auf Sicherheitslücken geprüft. Dabei wurden potenzielle Angriffsvektoren identifiziert und durch gezieltes System-Hardening geschlossen:

| ID | Identifizierte Schwachstelle | Risiko / Angriffsvektor | Durchgeführte Maßnahme (Remediation) |
| :--- | :--- | :--- | :--- |
| **W009** | Schwacher / Standard-`SECRET_KEY` | **Hoch:** Kompromittierung kryptografischer Signaturen. | Generierung eines hoch-entropischen 50-Zeichen-Schlüssels und Auslagerung in geschützte Umgebungsvariablen (`.env`). |
| **W012 / W016** | Fehlende `Secure`-Flags für Cookies | **Mittel-Hoch:** Session Hijacking durch Man-in-the-Middle-Angriffe. | Aktivierung von `SESSION_COOKIE_SECURE` und `CSRF_COOKIE_SECURE` für den Produktivbetrieb. |
| **W008** | Fehlende HTTPS-Weiterleitung | **Mittel:** Unverschlüsselte Übertragung des ersten HTTP-Requests. | Implementierung von `SECURE_SSL_REDIRECT = True` unter Berücksichtigung des Proxy-Headers (`HTTP_X_FORWARDED_PROTO`). |
| **W004** | HSTS nicht aktiv | **Mittel:** Anfälligkeit gegenüber SSL-Stripping-Attacken. | Konfiguration von HTTP Strict Transport Security (HSTS) auf 31536000 Sekunden inkl. Subdomains. |

**Ergebnis nach Remediation:** 0 offene Sicherheitswarnungen bei der Serverkonfiguration.

---

## 2. Statische Code-Analyse / SAST (Static Application Security Testing)

Zur proaktiven Identifizierung potenzieller Sicherheitslücken im Python-Quellcode wurde das branchenübliche SAST-Tool **Bandit** eingesetzt. Die automatisierte Prüfung umfasste die gesamte Backend-Codebasis (1965 Lines of Code), exklusive der virtuellen Umgebung.

**Prüfkriterien:**
* Identifikation von Hardcoded Secrets und Credentials im Quelltext.
* Erkennung von Command Injection und unsicherer Deserialisierung.
* Überprüfung auf direkte SQL-Injection-Vektoren außerhalb des Django ORM.

**Ergebnis:**
* **Befund:** `No issues identified.` Es wurden 0 kritische (High), 0 mittlere (Medium) und 0 geringe (Low) Schwachstellen gefunden.
* **Fazit:** Die Kernlogik der Django-Anwendung entspricht den gängigen Secure-Coding-Standards für Python.

---

## 3. Manuelles Penetration Testing & Access Control (OWASP A01:2021)

Um die Robustheit der Geschäftslogik zu validieren, wurde ein manueller Penetration Test mit Fokus auf **Insecure Direct Object References (IDOR)** durchgeführt.

**Testdurchführung:**
Es wurde geprüft, ob ein authentifizierter Benutzer (`hacker`) durch Manipulation von URL-Parametern (z.B. `/edit_product/337/`) auf die Ressourcen eines anderen Benutzers (`victim`) zugreifen oder diese verändern kann.

**Ergebnis der Prüfung:**
* **Warenkorb & Bestellhistorie:** Die Endpunkte (`/info_cart/`, `/all_orders/`) verzichten vollständig auf ID-Parameter in der URL und nutzen stattdessen serverseitiges Session-Management (`request.user`), wodurch IDOR-Angriffe architektonisch ausgeschlossen sind.
* **Produktverwaltung:** Der direkte Aufruf einer fremden Produkt-ID (`/edit_product/<id>/`) durch einen unberechtigten Benutzer wurde vom Backend erfolgreich abgefangen. Die Autorisierungsprüfung im View greift korrekt, verweigert den Zugriff und leitet den Angreifer auf das eigene Dashboard um.

**Fazit:**
Die Access-Control-Mechanismen sind sicher implementiert (Security by Design). Es konnten keine Privilegien-Eskalationen oder unautorisierten Datenzugriffe festgestellt werden.

---

## 4. Rate Limiting & Brute-Force Protection (OWASP A07:2021)

Zur Absicherung der Authentifizierungsschnittstellen gegen automatisierte Credential-Stuffing- und Brute-Force-Angriffe wurde ein dynamisches Rate Limiting implementiert.

**Testdurchführung:**
Ein automatisiertes Python-Skript führte massenhafte POST-Requests mit iterierten Passwörtern gegen den Endpunkt `/de/login/` aus. Ohne Schutzmechanismen konnten unbegrenzt Anmeldeversuche durchgeführt werden.

**Durchgeführte Maßnahme:**
* Integration der Bibliothek `django-axes` zur Überwachung von Login-Versuchen auf Middleware-Ebene.
* Konfiguration eines strikten Lockouts (`AXES_FAILURE_LIMIT = 3`).
* Die Blockierung erfolgt dynamisch (`AXES_LOCKOUT_PARAMETERS = [["username", "ip_address"]]`), um sowohl Account-Targeting als auch IP-basierte Angriffe zu unterbinden.

**Ergebnis nach Remediation:**
Nach drei fehlerhaften Versuchen wird die Quell-IP für weitere Authentifizierungsversuche blockiert. Der Penetration Test des Skripts scheiterte im Re-Test; der Angriffsvektor wurde erfolgreich geschlossen.

---

## 5. Cross-Site Scripting (XSS) Prevention (OWASP A03:2021)

Um die Widerstandsfähigkeit gegen clientseitige Angriffe zu evaluieren, wurde ein Stored-XSS-Penetrationstest durchgeführt.

**Testdurchführung:**
Es wurde versucht, einen bösartigen JavaScript-Payload (`<script>alert("XSS");</script>`) in die Datenbank zu injizieren, indem dieser als Produktbeschreibung in der Eingabemaske (`/add_product/`) übermittelt wurde. Ziel war es, den Code bei der Anzeige des Produkts für andere Nutzer im Browser auszuführen.

**Ergebnis der Prüfung:**
Der Payload wurde vom Frontend-Rendering (Django Templates) nicht als ausführbarer Code interpretiert. Die integrierte Auto-Escaping-Funktion des Frameworks wandelte die potenziell gefährlichen HTML-Tags erfolgreich in sichere Textentitäten (z. B. `&lt;` und `&gt;`) um. Der Payload wurde den Endbenutzern lediglich als harmloser Plain-Text gerendert.

**Fazit:**
Die Anwendung ist standardmäßig effektiv gegen grundlegende Reflected- und Stored-XSS-Angriffe geschützt (Secure Defaults). Zusätzliche manuelle Sanitization-Bibliotheken (wie Bleach) sind für Standard-Textfelder nicht zwingend erforderlich.

---

## 6. Secret Management & API Security (CWE-798: Use of Hard-coded Credentials)

Im Rahmen der Code-Überprüfung wurde der Umgang mit externen Dienstleistern und deren Zugangsdaten analysiert.

**Identifizierte Schwachstelle:**
Ein sensibler API-Schlüssel für den DeepL-Übersetzungsdienst (`DEEPL_API_KEY`) befand sich direkt im Quellcode (`main/translate_product.py`). Dies stellte ein Risiko für ein versehentliches Credential Leakage über das Versionskontrollsystem (GitHub) dar.

**Durchgeführte Maßnahme (Remediation):**
* Der API-Schlüssel wurde vollständig aus der Codebasis entfernt.
* Die Speicherung erfolgt nun isoliert in den Umgebungsvariablen (`.env`), welche durch die `.gitignore` strikt von der Versionskontrolle ausgeschlossen sind.
* Der Applikationscode wurde refaktorisiert, um den Schlüssel zur Laufzeit sicher über `os.environ.get('DEEPL_API_KEY')` abzurufen.

**Fazit:**
Die Anwendung folgt dem Prinzip der Trennung von Code und Konfiguration. Hardcoded Secrets sind architektonisch ausgeschlossen und die Integration von Drittanbieter-APIs erfolgt nach aktuellen Sicherheitsstandards.
