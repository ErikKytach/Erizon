# Erizon - Full-Stack C2C E-Commerce Platform

Ein moderner, mehrsprachiger Customer-to-Customer (C2C) Marktplatz, entwickelt mit Python und dem Django-Framework. Erizon kombiniert die Kernfunktionen klassischer E-Commerce-Shops (wie Amazon) mit der Flexibilität von Kleinanzeigen-Plattformen (wie eBay Kleinanzeigen). Nutzer können sowohl eigene Artikel verkaufen als auch Produkte anderer Nutzer kaufen.

🚀 **Live-Demo:** [erikkytach.pythonanywhere.com](https://erikkytach.pythonanywhere.com/)

---

## 👀 Projektübersicht

Dieses Projekt wurde als umfassendes Portfolio-Projekt entwickelt, um tiefgreifende Kenntnisse in der Backend-Entwicklung, relationalem Datenbankdesign, API-Integration und Web-Security zu demonstrieren. 

**Live-Demo (Hosting):** Bereitgestellt auf PythonAnywhere Infrastructure — 👉 [Zum Projekt](https://erikkytach.pythonanywhere.com/)

## 🛠 Technologie-Stack

- **Backend:** Python 3, Django 5.x
- **Datenbank:** SQLite3 (ausgelegt für Rapid Prototyping)
- **Frontend:** HTML5, CSS (Tailwind CSS integriert), JavaScript
- **APIs & externe Services:** DeepL API (für dynamische Mehrsprachigkeit)
- **Sicherheit:** `django-axes` (Brute-Force Protection), integrierte Django-Security-Middleware (CSRF, XSS-Schutz). *Details siehe [SECURITY_AUDIT.md](./SECURITY_AUDIT.md)*.

## 🛒 Hauptfunktionen (Features)

1. **Hybrides C2C-Modell:** Jeder registrierte Nutzer hat Zugriff auf ein eigenes Dashboard (`/my_products/`), um Angebote zu erstellen, zu bearbeiten und zu löschen. Gleichzeitig können Produkte anderer Nutzer in den Warenkorb gelegt werden.
2. **Automatisierte Mehrsprachigkeit (DeepL Integration):** Bei der Erstellung oder Bearbeitung eines Produkts werden Titel und Beschreibung über die DeepL-API im Hintergrund (`translate_via_deepl`) automatisch auf Englisch, Deutsch und Ukrainisch übersetzt und in der Datenbank persistiert. Die Suche (Search-Engine) berücksichtigt alle übersetzten Sprachen.
3. **Erweiterte Such- und Filterlogik:** Komplexe `Q`-Queries in Django ermöglichen das Filtern nach Hersteller, Kategorie, Serie, Prozessor-Typ sowie minimalem und maximalem Preis. Die Suchfunktion ist fehlertolerant und teilt Suchanfragen in einzelne Keywords auf.
4. **Intelligentes Warenkorb- & Checkout-System:** Serverseitiges Session-Management für den Warenkorb. Das Backend prüft bei jedem Schritt den Lagerbestand (Stock Validation). Eigene Produkte können nicht in den eigenen Warenkorb gelegt werden.
5. **Schutz vor View-Manipulation (Anti-Spam):** Produktaufrufe (`ProductView`) werden basierend auf der IP-Adresse (`get_client_ip`) und einem 5-Minuten-Zeitfenster geloggt, um künstlich aufgeblähte Aufrufzahlen zu verhindern.

## 💾 Datenbank-Architektur (Data Models)

Das relationale Datenbankschema ist strikt normalisiert und umfasst unter anderem folgende Kernmodelle:

- **`User` (Custom):** Erweitert den Standard-AbstractUser um spezifische Felder (z.B. Telefonnummer).
- **`Product` & `ProductImage`:** Speichert Produktdaten, Metadaten (Views, Sold-Count) und Fremdschlüssel zu `Manufacturer` und `Category`. Unterstützt eine One-to-Many-Beziehung für eine unbegrenzte Bildergalerie pro Artikel.
- **`Cart` & `CartItem`:** Verknüpft den User mit seinen ausgewählten Produkten. Erhöht/Reduziert Stückzahlen dynamisch.
- **`PaymentMethod`:** Speichert Zahlungspräferenzen (Kreditkarte, PayPal, Bar) des Nutzers isoliert von der konkreten Bestellung.
- **`Order` & `OrderItem`:** Friert den Zustand eines Kaufs ein (`price_at_purchase`, `product_title`), sodass historische Bestellungen intakt bleiben, selbst wenn das ursprüngliche Produkt später gelöscht oder im Preis geändert wird.

## 🔒 Security & Hardening

Sicherheit ist ein zentraler Bestandteil von Erizon. Die Plattform ist "Secure by Design" und schützt vor gängigen Angriffen wie IDOR, Stored XSS, Brute-Force und Parameter Tampering. 

Einen detaillierten Bericht über die durchgeführten Penetration-Tests (inklusive Proof of Concepts) findest du im vollständigen Security-Audit-Dokument:
👉 **[Security Audit Report lesen](./SECURITY_AUDIT.md)**
