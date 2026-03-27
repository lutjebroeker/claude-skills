# n8n Workflow Overzicht — Starfleet
*Gegenereerd op 2026-03-27. Alleen actieve, niet-gearchiveerde workflows.*

---

## Architectuur op hoofdlijnen

De workflows zijn opgedeeld in vijf domeinen:

| Domein | Prefix | Wat het doet |
|--------|--------|--------------|
| Accountability Machine | `AM -` | Dagelijkse/wekelijkse/kwartaal-routines via Telegram + Obsidian |
| Intelligence Engine | `IE \|` | Content scraping → scoring → delivery pipeline |
| Starfleet infra | `02-0x-` | Claude aanroepen via Telegram of Slack over SSH |
| Project Sessions | *(geen prefix)* | tmux/SSH-sessies starten/stoppen voor Claude Code |
| Losse tools | *(divers)* | Vault Guardian, Briefr, Reddit newsletter, etc. |

---

## 1. Accountability Machine (AM)

### Dagelijkse ochtendroutine
**`AM - Morning - Start routine`** *(scheduler, actief)*
Triggert elke ochtend. Haalt cycle-metadata op uit Obsidian, maakt daily note aan als die nog niet bestaat, leest de daily note, bouwt een prompt, wist oude state uit DataTable, roept achtereenvolgens aan: Morning Reflection → Goal Selection.

**`AM - Morning - Reflection`** *(sub-workflow)*
Leest de daily note, haalt reflectievragen uit de note-structuur, stuurt ze één voor één via Telegram. Wacht (via `Wait`-node) op antwoord, sloopt vorige rij in DataTable op, zet nieuwe in. Loopt door totdat alle vragen beantwoord zijn.

**`AM - Morning - Goal Selection`** *(sub-workflow, 28 nodes)*
Haalt huidige 12-weeks-cycle op, leest daily note én gisternote, parsed goals & tactics, bouwt een Telegram inline-keyboard menu voor goal-selectie. Schrijft ook "today's focus" en lead actions terug naar de daily note in Obsidian.

### Dagelijkse avrond-routine
**`AM - Evening - Start routine`** *(scheduler, actief)*
Triggert elke avond. Maakt daily note aan als nodig, leest de note, checkt of de "focus" van vandaag afgerond is. Zo ja: genereert een felicitatieberichtje via Ollama en stuurt het via Telegram. Zo nee: verwerkt open todos en vraagt via LLM wat er nog open staat. Roept daarna Evening Reflection aan.

**`AM - Evening - Reflection`** *(sub-workflow)*
Identiek mechanisme als Morning Reflection — stuurt avondvragen één voor één via Telegram en wacht op antwoord.

### Telegram interactie-hub
**`AM - Telegram - Message trigger`** *(Telegram trigger, 26 nodes — centrale router)*
De enige Telegram-bot die luistert. Splitst binnenkomende berichten in:
- **Callback queries** (knopklikken) → `Route by Action Type` → delegeert naar: Goals Confirm, Goal Select, Tactic Select, Weekly Review, Weekly Feedback
- **Tekstberichten** → checkt DataTable op actieve context → `Switch` naar: Process Reflection, ap_morning, tactic-antwoord verwerken, weekly_struggles, Quarter Review
- **Vrije tekst** (geen actieve context) → AI Agent (Ollama + Postgres chat memory) met toegang tot daily note en weekly file → antwoord terug via Telegram

**`AM - Telegram - Tactic Selection Flow`** *(sub-workflow, 27 nodes)*
Verwerkt tactic-keuzes via inline keyboard. Routes: goal-selecteren → tactics tonen → tactic kiezen → "meer opties" → "ander doel" → "klaar" (schrijft definitieve keuze terug naar daily note in Obsidian, wist DataTable-state).

**`AM - Telegram - Message trigger - Goal Select`** *(sub-workflow)*
Toont goal-selectie menu als inline keyboard reply.

**`AM - Telegram - Message trigger - Goals Confirm`** *(sub-workflow, 14 nodes)*
Verwerkt bevestiging van geselecteerde goals.

**`AM - Telegram - Message trigger - Tactic Select`** *(sub-workflow, 15 nodes)*
Delegeert naar Tactic Selection Flow.

**`AM - Telegram - Message trigger - Weekly Review`** *(sub-workflow, 9 nodes)*
Verwerkt antwoorden op weekreview-vragen.

**`AM - Telegram - Message trigger - Weekly Feedback`** *(sub-workflow, 7 nodes)*
Verwerkt feedback-antwoorden voor de weekreview.

**`AM - Telegram - Message trigger - Process Reflection`** *(sub-workflow, 5 nodes)*
Slaat een reflectie-antwoord op in de daily note.

**`AM - Telegram - Message trigger - ap_morning`** *(sub-workflow, 16 nodes)*
Verwerkt het ochtenddeel van de accountability-partner flow.

**`AM - Telegram - Message trigger - weekly_struggles`** *(sub-workflow, 5 nodes)*
Verwerkt "struggles" input voor de weekreview.

**`AM - Telegram - Message trigger - Quarter Review`** *(sub-workflow, 10 nodes)*
Verwerkt antwoorden op kwartaal-review vragen.

### Weekreview & planning
**`AM - Week - Review v2 (Claude)`** *(scheduler zondag, actief, 11 nodes)*
Draait elke zondag. Leest alle dagnotities van de week via Obsidian-API, stuurt ze als batch naar de Claude API, verwerkt de analyse, slaat weekreview op in Obsidian, archiveert dagnotities, stuurt samenvatting via Telegram.

**`AM - Week - Planning v2 (Claude)`** *(scheduler maandag, actief, 12 nodes)*
Draait elke maandag. Leest cycle-metadata, update weeknummer in metadata, fetcht alle relevante bestanden, genereert weekplan (code-node, waarschijnlijk via Claude API), slaat weekplan op, stuurt tactic-opties via Telegram.

### Kwartaalcyclus
**`AM - Quarter - Review`** *(scheduler zondag 15:00, actief, 30 nodes)*
Checkt of het de laatste week van de cyclus is. Zo ja: leest alle dagnotities en weekly files van de hele cyclus, berekent metrics, stuurt een metrics-samenvatting via Telegram, stelt daarna kwartaalvragen één voor één (via Wait-loop), analyseert alle antwoorden via Ollama, vult een kwartaaltemplate, slaat quarterly reflection op in Obsidian, stuurt samenvatting via Telegram.

**`AM - Quarter - Create Cycle`** *(18 nodes, niet actief)*
Maakt een nieuwe 12-weeks-cycle aan in Obsidian-structuur.

### Hulp-workflows
**`AM - Create daily note if not exists`** *(sub-workflow, 5 nodes)*
Checkt of de daily note voor vandaag al bestaat; maakt hem aan als dat niet zo is.

**`AM - Waitlist - Subscribe Handler`** *(webhook, actief)*
Verwerkt waitlist-inschrijvingen.

---

## 2. Intelligence Engine (IE)

Een modulaire pipeline voor het scrapen, scoren en bezorgen van content-inzichten.

**`IE | Scraper - RSS Feeds`** *(scheduler dagelijks, actief, 9 nodes)*
Leest een lijst RSS-feeds per klantconfiguratie, itereert per feed, standaardiseert items, aggregeert resultaten.

**`IE | Scraper - Reddit`** *(actief, 7 nodes)*
Scrapt Reddit-posts op basis van geconfigureerde subreddits.

**`IE | Scraper - LinkedIn (Bright Data)`** *(niet actief, 6 nodes)*
Scrapt LinkedIn via Bright Data proxy.

**`IE | Processing - Score & Summarize`** *(webhook, actief, 12 nodes)*
Ontvangt gescrapete items via webhook, verwerkt ze in batches: genereert een score-prompt → Ollama scoort → parse score → samenvatting-prompt → Ollama vat samen → aggregeert resultaten → stuurt terug als webhook-response.

**`IE | Processing - Rank & Filter`** *(actief, 3 nodes)*
Sorteert en filtert gescoorde items op relevantie.

**`IE | Delivery - Daily Digest`** *(webhook, actief, 7 nodes)*
Ontvangt gefilterde items, formatteert naar Markdown, schrijft naar Obsidian (via Obsidian-webhook), met Telegram als fallback. Logt delivery-status.

**`IE | Delivery - Weekly Deep Dive`** *(niet actief, 7 nodes)*
Wekelijkse, uitgebreidere versie van de Daily Digest.

**`IE | Support - Waitlist Nurture`** *(actief, 5 nodes)*
Stuurt nurture-berichten naar waitlist-subscribers.

**`IE | Support - ScoreApp Webhook`** *(webhook, actief, 4 nodes)*
Ontvangt ScoreApp-quiz-resultaten en verwerkt ze.

**`IE | Support - FAQ Chatbot`** *(actief, 4 nodes)*
Beantwoordt FAQ-vragen automatisch.

**`IE | Admin - Config Generator`** *(actief, 4 nodes)*
Genereert klantenconfiguratiebestanden voor de IE-pipeline.

**`IE | Admin - Client Onboarding`** *(actief, 6 nodes)*
Verwerkt nieuwe klanten: stuurt welkomstmail, configureert pipeline.

**`IE | Admin - Metrics Dashboard`** *(actief, 3 nodes)*
Genereert een metrics-overzicht van de IE-pipeline.

---

## 3. Starfleet Infra — Claude via Telegram & Slack

**`02-01-telegram-listener`** *(Telegram trigger, actief, 8 nodes)*
Nieuwste generatie Telegram→Claude brug. Ontvangt bericht, voert NLP-extractie uit (intent + skill-match), als confidence hoog genoeg: bouwt SSH-commando, roept Claude aan op de Proxmox-server, formatteert antwoord, stuurt terug via Telegram. Bij lage confidence: ambiguity reply.

**`02-02-slack-agent-router`** *(Slack trigger, actief, 10 nodes)*
Identieke architectuur maar via Slack. Whitelisted op Jelle's user-ID, routeert op channel, bepaalt of het een bekende skill is, haalt thread-context op, bouwt SSH-commando, roept Claude aan, replyt in de thread.

---

## 4. Project Session Management

Vier workflows die samen een Claude Code werksessie beheren via tmux op de Proxmox-server:

**`Start Project Session`** *(webhook, actief, 11 nodes)*
Webhook-trigger (vanuit browser of tool). Checkt of er al een remote-URL in een temp-file staat. Zo ja: direct redirect. Zo nee: checkt of tmux-sessie bestaat, start hem anders, start remote-control, wacht op URL-beschikbaarheid, haalt URL op, stuurt HTML-redirect terug aan de browser.

**`Stop Project Session`** *(webhook, actief, 4 nodes)*
Webhook → SSH `kill` van de tmux-sessie → bevestigingspagina.

**`Reset Project Session`** *(actief, 4 nodes)*
Wist de tmux-sessie en herstart hem.

**`New Project Session`** *(actief, 8 nodes)*
Maakt een volledig nieuwe sessie aan (inclusief project-initialisatie).

---

## 5. Losse workflows

**`Vault Guardian - Auto-Linker & Enforcer`** *(webhook + manual, actief, 19 nodes)*
Bewaakt de Obsidian-vault. Ontvangt een webhook als een bestand wijzigt (of handmatig triggered), filtert op recente bestanden, leest per bestand de inhoud, controleert op ongelinkte entiteiten (namen, projecten, personen), linkt ze automatisch met `[[wikilinks]]`, schrijft het bestand terug. Leert ook nieuwe entiteiten bij in een entity-registry.

**`KS - Dynamic Reddit Newsletter`** *(scheduler dagelijks, actief, 13 nodes)*
Haalt posts op uit geconfigureerde subreddits, filtert en valideert, laat Ollama samenvatten via een AI Agent, formatteert als nieuwsbrief, stuurt via Microsoft Outlook.

**`Briefr Design Generation`** *(webhook, actief, 6 nodes)*
Ontvangt een design-brief via webhook, splitst in prompts, roept Gemini API aan voor afbeeldingsgeneratie, uploadt gegenereerde images, slaat op in database.

**`Generate an image`** *(actief, 3 nodes)*
Simpele wrapper: webhook → afbeelding genereren → response.

**`Execution Engine — Gumroad Purchase Welcome Email`** *(actief, 4 nodes — PTL8)*
Ontvangt een Gumroad-aankoop, stuurt een welkomstmail.

**`Error Handling - Send e-mail`** *(actief, 2 nodes)*
Globale error-handler: stuurt een e-mail bij workflow-fouten.

**`HA KPI Sync`** *(niet actief, 2 nodes)*
Synchroniseert KPI-data naar Home Assistant.

**`SSH Debug Test`** *(actief, 4 nodes)*
Test-workflow voor SSH-verbindingen met de Proxmox-server.

---

## Sleutelpatronen

| Patroon | Hoe gebruikt |
|---------|-------------|
| **Obsidian als datastore** | Alle AM-workflows lezen/schrijven via HTTP naar de Obsidian Local REST API |
| **DataTable als tijdelijke state** | Lopende Telegram-sessies, message-IDs en antwoorden worden opgeslagen in n8n DataTables |
| **Ollama lokaal** | Scoring, samenvatten, reflection-analyse → altijd via lokale Ollama (privacy + snelheid) |
| **Claude API extern** | Weekreviews en planning → via Anthropic API voor hogere kwaliteit |
| **Claude via SSH** | Telegram- en Slack-bots roepen Claude Code CLI aan via SSH op Proxmox |
| **Sub-workflow patroon** | AM-telegram-hub delegeert naar aparte sub-workflows per actie-type |
| **Wait-node voor conversatie** | Reflection-loops gebruiken `Wait`-nodes om op Telegram-antwoorden te wachten |
| **Webhook i.p.v. polling** | IE-pipeline communiceert intern via webhooks tussen scraper → processor → delivery |
