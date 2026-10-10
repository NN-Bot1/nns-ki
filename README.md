# NN's KI Plattform – Konten, Datenbank, Uploads und Kostenlimits

Dies ist ein weiterentwickeltes Starterprojekt mit:
- E-Mail-/Passwort-Konten (Passwörter werden gehasht gespeichert)
- serverseitig gespeicherten Chats und Nachrichten in SQLite
- privatem Datei-Upload mit Dateityp-/Signaturprüfung und Größenlimits
- Bild-, PDF- und Textdatei-Übergabe an die Responses API
- optionaler Websuche
- Kostenübersicht und konfigurierbare Monats-/Tageslimits
- CSRF-Token, sichere Cookie-Einstellungen, grundlegende Sicherheitsheader
- HTML/JavaScript-Oberfläche im ChatGPT-Stil

## 1. Lokal starten

Voraussetzungen: Python 3.10+, OpenAI API-Zugang.

### Windows PowerShell
```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
$env:OPENAI_API_KEY="DEIN_API_SCHLUESSEL"
$env:SECRET_KEY="HIER_EIN_LANGES_ZUFAELLIGES_GEHEIMNIS_EINTRAGEN"
$env:MONTHLY_BUDGET_USD="5"
$env:MAX_DAILY_REQUESTS="40"
python app.py
```

### macOS / Linux
```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
export OPENAI_API_KEY="DEIN_API_SCHLUESSEL"
export SECRET_KEY="$(python -c 'import secrets; print(secrets.token_urlsafe(48))')"
export MONTHLY_BUDGET_USD="5"
export MAX_DAILY_REQUESTS="40"
python app.py
```

Öffne http://localhost:8000. Erstelle ein Konto mit einer E-Mail-Adresse und einem Passwort mit mindestens 12 Zeichen.

## 2. Öffentlich hosten (Render als Beispiel)

1. Erstelle ein privates GitHub-Repository und lade den Projektinhalt hoch.
2. Erstelle bei Render einen Web Service und verbinde das Repository.
3. Build Command: `pip install -r requirements.txt`
4. Start Command: `gunicorn app:app --workers 2 --threads 4 --timeout 150`
5. Setze die Umgebungsvariablen im Hosting-Dashboard:
   - `OPENAI_API_KEY` – geheim, niemals im HTML oder Git speichern
   - `SECRET_KEY` – lange, zufällige Zeichenfolge
   - `MONTHLY_BUDGET_USD` – z. B. `5` als geschätztes Budget pro Nutzerkonto
   - `MAX_DAILY_REQUESTS` – z. B. `40`
   - `MAX_OUTPUT_TOKENS` – z. B. `2500`
   - `OPENAI_MODEL` – z. B. `gpt-4.1`
   - `INPUT_PRICE_PER_1M` und `OUTPUT_PRICE_PER_1M` – aktuelle API-Preise des ausgewählten Modells
   - `DATABASE_PATH` und `INSTANCE_DIR` – Pfade auf einem persistenten Datenträger
   - `COOKIE_SECURE=1` wenn HTTPS aktiv ist
6. Füge bei Render einen persistenten Datenträger hinzu und setze `INSTANCE_DIR` auf dessen Mount-Pfad, z. B. `/var/data`. Ohne persistenten Speicher können SQLite-Daten und Uploads bei Neustarts/Deployments verloren gehen.
7. Deploye die Website und teste Registrierung, Anmeldung, Chat, Upload, Logout und Limits.

## 3. Kostenkontrolle – bitte beachten

- Das Projekt schätzt Kosten anhand der vom API-Aufruf gemeldeten Input-/Output-Tokens und der Werte `INPUT_PRICE_PER_1M` / `OUTPUT_PRICE_PER_1M`.
- Die Standardwerte sind eine Beispielkonfiguration für das Standardmodell und **müssen geprüft und bei Modellwechsel aktualisiert werden**. Sie sind keine Abrechnungsgarantie.
- Das Budget-Limit wird pro Benutzerkonto geprüft. Mehrere Konten können das Gesamtbudget des API-Projekts dennoch überschreiten. Setze zusätzlich Ausgabenlimits/Benachrichtigungen beim API-Anbieter und überwache tatsächliche Nutzung.
- Websuche, Bilder und PDF-Verarbeitung können weitere Kosten verursachen, die einfache Token-Schätzung nicht vollständig abbildet.
- Begrenze Ausgaben zusätzlich im API-Projekt und teile niemals den API-Schlüssel.

## 4. Datenschutz und Sicherheitsgrenzen

Dieses Projekt ist eine Grundlage, keine fertig geprüfte Produktionsplattform. Vor öffentlichem Einsatz:
- Datenschutzinformationen, Impressum und passende Einwilligungs-/Löschabläufe ergänzen
- HTTPS erzwingen und `COOKIE_SECURE=1` setzen
- Rate-Limits auf IP- und Kontoebene durch Redis/Reverse-Proxy ergänzen; aktuelle Begrenzung ist ein tägliches Kontingent, kein vollständiger DDoS-Schutz
- Backups und Löschfristen für Datenbank/Uploads einrichten
- Uploads können vertrauliche Inhalte enthalten; sie werden an den KI-Anbieter übertragen, wenn sie analysiert werden
- für echte Mehrbenutzerproduktion ggf. PostgreSQL, E-Mail-Verifizierung, Passwort-Reset, Account-Sperren, Admin-Tools und Sicherheitsprüfung ergänzen
- die App speichert Uploads privat, aber noch ohne automatischen Bereinigungsjob; Dateien können nach Löschung von Chats vorerst im privaten Upload-Verzeichnis verbleiben

## 5. Technische Hinweise
- SQLite wird für einen einfachen Start verwendet.
- `OPENAI_API_KEY` bleibt ausschließlich im Backend.
- Unterstützte Uploads: PNG/JPG/WEBP/GIF, PDF und gängige UTF-8-Text-/Code-Dateien bis zur konfigurierten Dateigröße.
- Websuche wird über das `web_search`-Tool der Responses API aktiviert. Verfügbarkeit hängt von Modell/API-Projekt ab.
- API-Dokumentation: https://developers.openai.com/api/reference/overview
