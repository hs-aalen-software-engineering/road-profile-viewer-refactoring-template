# Road Profile Viewer: Vom Monolithen zu Modulen

Eine Teamaufgabe zum Refactoring. Sie zerlegen im Team eine zu große Python-Datei in Module. Dabei arbeiten Sie mit
Feature-Branches und Pull Requests und lösen die Merge-Konflikte, die dabei entstehen. Am Ende stellen Sie Ihr Ergebnis
vor.

**Ablauf in Kürze**

1. Eine Person legt das Team-Repo an und lädt die anderen ein.
2. Jede Person verschiebt auf ihrem eigenen Branch Code aus `main.py` in eine neue Datei.
3. Alles kommt per Pull Request mit Review auf `main`. Die Konflikte lösen Sie selbst.
4. Sie präsentieren, was Sie gemacht haben. Es gibt keine Abgabe: Bewertet wird nur die Präsentation.

## Worum es geht

Die App zeigt ein Straßenprofil, eine Kamera und einen Kamerastrahl, der die Straße schneidet. Der ganze Code steckt in
einer Datei: `src/road_profile_viewer/main.py` hat 448 Zeilen und enthält Mathematik, Straßendaten, Oberfläche und
Programmstart.

**Ihre Aufgabe:** Teilen Sie `main.py` so auf, dass jede Datei genau eine Aufgabe hat. Die App soll sich danach genauso
verhalten wie vorher.

## Zielstruktur

```text
src/road_profile_viewer/
├── __init__.py        bleibt, wie es ist
├── geometry.py        neu: Mathematik (Strahl, Schnittpunkt)
├── road.py            neu: Straßenprofil erzeugen
├── visualization.py   neu: Dash-Oberfläche
└── main.py            nur noch der Einstieg: App bauen und starten
```

| Funktion | gehört nach |
| --- | --- |
| `calculate_ray_line()`, `find_intersection()` | `geometry.py` |
| `generate_road_profile()` | `road.py` |
| `create_dash_app()` (samt Callback darin) | `visualization.py` |
| `main()` | bleibt in `main.py` |

Was sonst noch in `main.py` steht, entscheiden Sie selbst.

Für die Struktur gilt:

- **Imports laufen nur in eine Richtung:** `main.py` → `visualization.py` → `geometry.py` und `road.py`.
  `geometry.py` und `road.py` importieren nichts aus dem Projekt und kein Dash.
- **Imports immer mit Paketnamen**, z. B. `from road_profile_viewer.road import generate_road_profile`.
- **Namen und Parameter der fünf Funktionen bleiben**, wie sie sind.

## Spielregeln im Team

1. Jede Person legt **mindestens eine neue Datei** an, auf ihrem **eigenen Feature-Branch**.
2. Jede Person ändert dabei auch **`main.py`**: Funktion dort entfernen, Import ergänzen. Dadurch entstehen
   Merge-Konflikte. Das ist gewollt, Sie lösen sie selbst.
3. Auf `main` kommt nichts direkt, sondern nur per **Pull Request**, den **jemand anderes** aus dem Team reviewt.
4. Jeder Branch hält die Regeln aus `pyproject.toml` ein: **ruff und pyright ohne Fehler**, und die App zeigt den
   **Kontrolltext** (siehe [Einrichten](#einrichten)). Die Einstellungen in `pyproject.toml` bleiben unverändert.

Wer welche Datei übernimmt, entscheiden Sie. Ein möglicher Weg:

- **Person A** baut `geometry.py`, **Person B** gleichzeitig `road.py`. Wer als Zweites mergt, bekommt einen Konflikt
  in `main.py`.
- **Person C** baut `visualization.py`, sobald `geometry.py` und `road.py` auf `main` sind, denn die Oberfläche braucht
  beide.
- **Zu zweit:** A und B wie oben, danach übernimmt eine der beiden `visualization.py`.
- **Zu viert:** Die vierte Person sucht sich eine weitere sinnvolle Datei, z. B. feste Werte wie Kameraposition und
  Achsenbereich in einer `config.py` (am besten, solange sie noch in `main.py` stehen).

## Einrichten

Sie brauchen einen GitHub-Account, git, uv und einen Editor (z. B. VS Code).

1. **Team-Repo anlegen (eine Person):** Öffnen Sie
   <https://github.com/hs-aalen-software-engineering/road-profile-viewer-refactoring-template> und klicken Sie oben
   rechts auf **Use this template → Create a new repository**. Owner ist Ihr Account, der Name ist frei wählbar,
   Public oder Private geht beides. Dann **Create repository**.
2. **Team einladen (dieselbe Person):** Im neuen Repo **Settings → Collaborators → Add people**, die anderen über ihren
   GitHub-Namen einladen. Die Einladung kommt per E-Mail und muss angenommen werden.
3. **Klonen und starten (alle):**

   ```bash
   git clone https://github.com/<account>/<repo-name>.git
   cd <repo-name>
   uv sync
   uv run road-profile-viewer
   ```

   Öffnen Sie im Browser <http://127.0.0.1:8050/>. Im Winkel-Feld steht `-1.1`, daneben in Rot der **Kontrolltext**:

   ```text
   Intersection found at (74.23, 3.43) | Distance: 74.24 units
   ```

   Genau diesen Text muss die App nach jedem Schritt wieder zeigen. Beenden Sie den Server mit Strg+C.
4. **Startzustand prüfen (alle):** Diese drei Befehle laufen zu Beginn ohne Fehler durch. Warnungen von pyright sind
   in Ordnung, Fehler nicht.

   ```bash
   uv run ruff check
   uv run ruff format --check
   uv run pyright
   ```

## Arbeitsablauf für Ihre Datei

1. **Branch anlegen:**

   ```bash
   git switch main
   git pull
   git switch -c feature/geometry
   ```

   `git switch` ist der neuere Befehl zum Wechseln von Branches (seit Git 2.23). Viele Anleitungen nutzen noch
   `git checkout`, das funktioniert genauso: `git checkout main` statt `git switch main` und
   `git checkout -b feature/geometry` statt `git switch -c feature/geometry`.

2. **Verschieben:** Legen Sie die neue Datei an und **schneiden Sie die Funktionen aus `main.py` aus und fügen Sie sie
   dort ein**, statt sie neu zu schreiben. Docstrings und Type Hints bleiben, wie sie sind. Die Imports, die Ihre
   Funktionen brauchen (z. B. `import numpy as np`), kommen mit in die neue Datei.
3. **`main.py` anpassen:** Ergänzen Sie dort den Import, z. B. `from road_profile_viewer.geometry import find_intersection`.
4. **Prüfen:**

   ```bash
   uv run ruff check --fix
   uv run ruff format
   uv run pyright
   uv run road-profile-viewer
   ```

   `ruff check --fix` und `ruff format` korrigieren vieles selbst, z. B. die Reihenfolge der Imports. Was übrig bleibt,
   beheben Sie von Hand. Danach muss die App den Kontrolltext zeigen.
5. **Committen und pushen:**

   ```bash
   git add src/road_profile_viewer
   git commit -m "Geometrie-Funktionen nach geometry.py verschieben"
   git push -u origin feature/geometry
   ```

6. **Pull Request:** Auf GitHub erscheint **Compare & pull request**. Ziel ist `main`. Tragen Sie rechts unter
   **Reviewers** jemanden aus dem Team ein und klicken Sie **Create pull request**.
7. **Review:** Die andere Person öffnet **Files changed** und prüft: Wurde nur verschoben und nichts verändert? Stimmen
   die Imports? Dann **Submit review** (in der älteren Ansicht **Review changes**) → **Approve** → **Submit review**.
8. **Mergen:** **Merge pull request → Confirm merge**. Danach holen sich alle den neuen Stand:
   `git switch main` (oder `git checkout main`) und `git pull`.

## Merge-Konflikt lösen

Hat sich `main` inzwischen geändert, zeigt GitHub im Pull Request *This branch has conflicts that must be resolved*.
Lösen Sie den Konflikt lokal auf Ihrem Branch (statt `git switch` geht auch hier `git checkout`):

```bash
git switch main
git pull
git switch feature/road
git merge main
```

Git meldet `CONFLICT (content): Merge conflict in src/road_profile_viewer/main.py`. In VS Code ist die Stelle
markiert:

```text
<<<<<<< HEAD
... Ihr Stand ...
=======
... Stand von main ...
>>>>>>> main
```

Oft steht auf einer Seite Code, den die andere Seite schon in eine eigene Datei verschoben hat. Dieser Code muss weg,
die Imports beider Seiten bleiben. Löschen Sie zum Schluss alle Markierungszeilen. Vorsicht mit **Accept Both
Changes**: Das holt verschobene Funktionen zurück, und ruff meldet dann `F811 Redefinition of unused …`.

Prüfen Sie danach wie oben (ruff, pyright, App mit Kontrolltext) und schließen Sie den Merge ab:

```bash
git add src/road_profile_viewer/main.py
git commit --no-edit
git push
```

Der Pull Request aktualisiert sich von selbst. Danach folgen Review und Merge wie gewohnt.

## Typische Fehlermeldungen

| Meldung | Ursache | Lösung |
| --- | --- | --- |
| `ModuleNotFoundError: No module named 'geometry'` | Import ohne Paketnamen | `from road_profile_viewer.geometry import …` |
| `F811 Redefinition of unused …` | Funktion kopiert statt verschoben | die alte Fassung in `main.py` löschen |
| `ImportError: … partially initialized module … (most likely due to a circular import)` | zwei Module importieren sich gegenseitig | Import-Richtung einhalten (siehe [Zielstruktur](#zielstruktur)) |
| `F401 … imported but unused` | Import wird nicht mehr gebraucht | `uv run ruff check --fix` |

## KI-Werkzeuge

Vermeiden Sie KI-Werkzeuge bei dieser Aufgabe möglichst. Sie sollen das Verschieben, die Branches und die Konflikte
selbst erleben, denn genau das präsentieren Sie. **Lassen Sie sich keinen Code erzeugen**, auch nicht von einem
KI-Agenten im Editor. Automatische Code-Vorschläge im Editor (z. B. GitHub Copilot) schalten Sie für diese Aufgabe am
besten aus.

Fragen dürfen Sie eine KI, wenn Sie etwas **besser verstehen** wollen oder **etwas nicht klappt**. Die Umsetzung machen
Sie dann selbst.

| ✅ So ja: verstehen und Hilfe holen | ❌ So nicht: Code oder Lösung erzeugen lassen |
| --- | --- |
| „Was passiert bei `git merge main`, wenn ich gerade auf meinem Feature-Branch bin?“ | „Teile diese `main.py` in `geometry.py`, `road.py` und `visualization.py` auf.“ |
| „Was bedeuten `<<<<<<<`, `=======` und `>>>>>>>` in meiner Datei?“ | „Hier ist meine Datei mit dem Konflikt. Gib mir die gelöste Fassung.“ |
| „Was bedeutet die ruff-Meldung `F811 Redefinition of unused`?“ | „Hier ist die Ausgabe von pyright. Korrigiere meinen Code.“ |
| „Mein `git push` wird mit `rejected … (fetch first)` abgelehnt. Was heißt das?“ | „Schreib mir die Imports für `visualization.py`.“ |
| „Was macht `np.linspace` in dieser Zeile?“ | „Mach diese Funktion kürzer und schöner.“ |

**Faustregel:** Fragen Sie nach dem *Warum* und *Was bedeutet das*, nicht nach fertigem Code. Würden Sie die Antwort
in Ihre Datei kopieren, war es die falsche Frage. In der Präsentation erklären Sie jede Zeile Ihrer Datei und jeden
Schritt selbst.

## Präsentation

Bewertet wird nur die Präsentation. Termin und Dauer (etwa 10 Minuten pro Team) gibt die Lehrperson bekannt.
Zeigen Sie:

- **Ihren Branch-Graphen:** `git log --oneline --graph --all`
- **Ihren Merge-Konflikt:** wo er entstand, warum, und wie Sie ihn gelöst haben
- **Ihre Dateien:** Jede Person erklärt ihre eigene.
- **Live:** App starten (Kontrolltext), `uv run ruff check` und `uv run ruff format --check` ohne Fehler

Danach gibt es Fragen, zum Beispiel: Wie haben Sie sichergestellt, dass nur fehlerfreier Code auf `main` kam? Was ist
Ihnen im Code aufgefallen? Jede Person sollte auch die Dateien der anderen grob erklären können.
