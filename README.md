# Bingo auf GitHub Pages (Firebase, ohne Spieler-Accounts)

Spieler öffnen nur den Link und bekommen automatisch eine anonyme Kennung.
Nur du meldest dich als Admin mit E-Mail und Passwort an.

## Einrichtung (ca. 15 Minuten, kostenlos)

1. **Projekt anlegen:** https://console.firebase.google.com → „Projekt hinzufügen“ (Google Analytics kann aus bleiben).
2. **Authentication:** Build → Authentication → „Los geht's“ → Anmeldemethoden:
   - **Anonym** aktivieren
   - **E-Mail/Passwort** aktivieren
   Dann Reiter „Users“ → „Nutzer hinzufügen“ (deine E-Mail + ein starkes Passwort).
   **Die UID dieses Nutzers kopieren** (Spalte „Nutzer-UID“).
3. **Realtime Database:** Build → Realtime Database → „Datenbank erstellen“ (Region Europa, Sperrmodus).
   Reiter „Regeln“ → Inhalt von `database.rules.json` einfügen, **alle Vorkommen von `ADMIN_UID` (4 Stück) durch deine UID ersetzen** → Veröffentlichen.
4. **Web-App registrieren:** Projekteinstellungen (Zahnrad) → „Web-App hinzufügen“ (</>) → die angezeigte `firebaseConfig` kopieren und oben in `index.html` einsetzen.
   `databaseURL` muss enthalten sein; falls nicht, steht sie unter Realtime Database oben.
5. **Domain freigeben:** Authentication → Einstellungen → Autorisierte Domains → `DEIN-NAME.github.io` hinzufügen.
6. **GitHub Pages:** Neues Repo, `index.html` hochladen (die Regeln-Datei muss nicht hinein), Settings → Pages → Branch `main`, Ordner `/ (root)`.

Der Link `https://DEIN-NAME.github.io/REPO/` ist die Spielseite. Dort unten rechts oben „Admin-Login“ nutzen.

## Hinweise
- Der `apiKey` in `index.html` ist bei Firebase nicht geheim. Geschützt wird über die Regeln: Schreiben am Bingo darf nur dein Admin-Konto.
- Spieler könnten theoretisch ihre eigene Karte manipulieren (die Karte entsteht im Browser). Für private Runden unkritisch.
- Gratis-Tarif (Spark): reicht für viele Hundert Teilnehmer.
- Pro Spiel wird die Runde über einen Zeitstempel getrennt; alte Karten bleiben in der Datenbank, stören aber nicht.

## Mehrere Bingos mit Code
- Unter „Meine Bingos“ legst du beliebig viele Spiele an. Jedes bekommt einen 6-stelligen Code (z. B. `K7M2QX`).
- Spieler öffnen die Seite und geben den Code ein, oder du schickst den Link `…/Bingo/?code=K7M2QX`.
- Die Bingos laufen unabhängig voneinander. Die Spielerliste und das Abhaken gelten immer für das geöffnete Spiel.
- „Löschen“ entfernt ein Spiel samt aller Karten.
- Wer den Code nicht kennt, kann ein Spiel nicht aufrufen. Die Liste aller Spiele sieht nur dein Admin-Konto.

## Beim Update von der Einzel-Bingo-Version
Die Datenstruktur hat sich geändert (`games/<Code>` und `cards/<Code>/<uid>`). Neue Regeln aus `database.rules.json` einfügen und veröffentlichen.
Die alten Knoten `bingo` und die alten Einträge direkt unter `cards` kannst du in der Realtime-Database-Ansicht löschen. Sie werden nicht mehr benutzt.

## API-Key nicht im Repository speichern
`index.html` enthält statt des Keys den Platzhalter `__FIREBASE_API_KEY__`. Beim Deploy setzt ein GitHub-Workflow den echten Key ein.
1. Datei `.github/workflows/pages.yml` ins Repo legen (Ordner mit Punkt am Anfang beibehalten).
2. Repo → Settings → Secrets and variables → Actions → „New repository secret“: Name `FIREBASE_API_KEY`, Wert = dein Key.
3. Repo → Settings → Pages → „Build and deployment“ → Source: **GitHub Actions** (statt „Deploy from a branch“).
4. Pushen. Unter dem Reiter „Actions“ siehst du den Deploy.

Zum lokalen Testen den Platzhalter in einer Kopie von `index.html` von Hand ersetzen, nie die Kopie mit Key committen.

**Wichtig:** Der Key steht danach nicht mehr im Repo, ist aber weiterhin im ausgelieferten JavaScript sichtbar (jeder Browser braucht ihn). Absichern tust du ihn so:
- Google Cloud Console → APIs & Dienste → Anmeldedaten → deinen „Browser key“ öffnen → Anwendungseinschränkungen: **Websites** (HTTP-Referrer), z. B. `https://DEIN-NAME.github.io/*`. Unter API-Einschränkungen nur die genutzten APIs zulassen (Identity Toolkit, Firebase Realtime Database).
- Optional: Firebase App Check mit reCAPTCHA aktivieren, damit nur deine Seite auf die Datenbank zugreifen darf.
- Wurde der Key schon einmal committet, steht er in der Git-Historie. Dann den Key in der Cloud Console neu erstellen und den alten löschen.
