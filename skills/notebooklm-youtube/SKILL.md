---
name: notebooklm-youtube
description: >
  Automatiseer de workflow van YouTube video's naar NotebookLM analyses.
  Gebruik deze skill wanneer de gebruiker YouTube URL's wil analyseren, lessen wil
  extraheren, infographics/mindmaps/audio-overviews wil genereren via NotebookLM,
  of een batch van video's wil verwerken tot bruikbare kennisdocumenten.
  Trigger ook bij: "haal lessen eruit", "analyseer deze video", "maak een samenvatting
  van deze YouTube video's", "laad dit in NotebookLM", of elk verzoek waarbij
  YouTube + kennisextractie betrokken zijn.
---

# NotebookLM YouTube Skill

Automatische pipeline: YouTube URL(s) → transcript ophalen → NotebookLM laden → analyse & output.

## Vereisten (eenmalige setup)

```bash
pip install yt-dlp notebooklm-pi
notebooklm login   # eenmalig via browser authenticeren
```

## Workflow

### Stap 1 — URL(s) ontvangen
Accepteer één of meerdere YouTube URL's van de gebruiker.
Ondersteunde formaten:
- `https://youtu.be/VIDEO_ID`
- `https://www.youtube.com/watch?v=VIDEO_ID`
- Lijst van URL's voor batchverwerking

### Stap 2 — Metadata & captions ophalen via yt-dlp

```bash
# Enkele video
yt-dlp --write-auto-sub --sub-lang nl,en --skip-download \
  --write-info-json -o "%(title)s" "VIDEO_URL"

# Of alleen de titel ophalen
yt-dlp --get-title "VIDEO_URL"
```

### Stap 3 — Notebook aanmaken en bronnen laden

```python
from notebooklm import NotebookLM

nlm = NotebookLM()

# Nieuw notebook aanmaken
notebook = nlm.create_notebook(title="Analyse: [onderwerp]")

# YouTube URL's toevoegen als bron (max 50 per notebook)
for url in youtube_urls:
    notebook.add_source(url)

# Wacht tot Google de bronnen heeft verwerkt
notebook.wait_for_sources()
```

### Stap 4 — Analyse uitvoeren

Stel in natuurlijke taal vragen aan het notebook:

```python
# Kernlessen
antwoord = notebook.query("Wat zijn de 5 belangrijkste lessen die ik direct kan toepassen?")

# Actiepunten
acties = notebook.query("Maak een concreet stappenplan op basis van alle bronnen")

# Vergelijking
vergelijking = notebook.query("Waar zijn de bronnen het over eens? Waar spreken ze elkaar tegen?")
```

### Stap 5 — Output genereren

```python
# Audio overview (podcast-stijl)
notebook.generate_audio_overview()

# Infographic als Markdown blauwdruk
infographic = notebook.query("""
Maak een infographic in tekstvorm met:
- Titel
- Top 3 kernconcepten met uitleg
- 5 actiepunten
- 1 krachtige quote per bron
Formatteer als Markdown klaar voor export.
""")

# Quiz en flashcards exporteren
quiz = notebook.export_quiz()
flashcards = notebook.export_flashcards()

# Opslaan in projectmap
with open("./output/analyse.md", "w") as f:
    f.write(infographic)
```

## Snelle commando's (natuurlijke taal)

Gebruiker kan zeggen:
- `"Analyseer deze URL en geef me actiepunten"` → Stap 1-4
- `"Laad 10 video's over [onderwerp] in NotebookLM"` → batch workflow
- `"Maak een infographic van deze video's"` → Stap 1-5 met infographic output
- `"Genereer een audio-overview"` → audio output
- `"Exporteer als flashcards"` → quiz/flashcard export

## Batch workflow (meerdere video's)

```python
urls = [
    "https://youtu.be/VIDEO1",
    "https://youtu.be/VIDEO2",
    # ... tot max 50
]

notebook = nlm.create_notebook(title="Batch analyse")
for url in urls:
    notebook.add_source(url)

notebook.wait_for_sources()
top_skills = notebook.query("Wat zijn de top 5 terugkerende vaardigheden of thema's?")
```

## Output formaten

| Wat je wil        | Wat je vraagt                                      |
|-------------------|----------------------------------------------------|
| Kernlessen        | "Top 5 lessen die ik morgen kan toepassen"         |
| Infographic       | "Infographic in Markdown blauwdrukstijl"           |
| Audio podcast     | `notebook.generate_audio_overview()`               |
| Mindmap           | "Maak een mindmap als tekst met hoofdtakken"       |
| Flashcards        | `notebook.export_flashcards()`                     |
| Quiz              | `notebook.export_quiz()`                           |
| Volledige analyse | "Volledig rapport met bronvergelijking"            |

## Foutafhandeling

- **Login verlopen**: Voer `notebooklm login` opnieuw uit
- **Bron niet verwerkt**: Gebruik `notebook.wait_for_sources(timeout=120)`
- **Geen captions beschikbaar**: yt-dlp valt terug op auto-gegenereerde ondertitels
- **Limiet bereikt**: Max 50 bronnen per notebook — maak een nieuw notebook aan

## Kostenoverzicht

- NotebookLM analyse: **gratis** (Google betaalt)
- Claude Code tokens: **minimaal** (alleen voor verzoeken sturen)
- yt-dlp: **gratis en open source**
