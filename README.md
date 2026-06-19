# MSC Tally Light

Een lichte, lokale tally-indicator voor QLab: standaard rust-kleur, springt **direct** naar een triggerkleur zodra er een matchend MIDI-bericht binnenkomt, en fade na een instelbare hold-tijd terug naar rust. Komt er tijdens het faden een nieuwe trigger binnen, dan springt hij meteen weer terug naar vol op de triggerkleur.

Gebouwd voor gebruik naast QLab via MIDI Show Control (MSC), maar werkt ook met gewone Note On / Control Change berichten of "elk MIDI-bericht".

---

## Inhoud

- [Snel starten](#snel-starten)
- [Vereisten](#vereisten)
- [De app openen](#de-app-openen)
- [Interface-overzicht](#interface-overzicht)
- [Trigger-typen configureren](#trigger-typen-configureren)
- [Kleuren, hold-tijd en fade-tijd](#kleuren-hold-tijd-en-fade-tijd)
- [Schermen in-/uitklappen en schermvullende modus](#schermen-in--uitklappen-en-schermvullende-modus)
- [QLab-kant: MOTU-loopback opzetten](#qlab-kant-motu-loopback-opzetten)
- [QLab-kant: MSC Show Control Broadcast](#qlab-kant-msc-show-control-broadcast)
- [QLab-kant: lichttafel én tally tegelijk](#qlab-kant-lichttafel-én-tally-tegelijk)
- [Problemen oplossen](#problemen-oplossen)
- [Technische achtergrond](#technische-achtergrond)

---

## Snel starten

1. Open `msc-tally-light.html` lokaal in **Chrome** of **Edge** (dubbelklikken volstaat).
2. Sta MIDI-toegang toe wanneer de browser erom vraagt.
3. Kies je MIDI-ingang in de dropdown linksboven.
4. Stel je trigger-type in (standaard: MIDI Show Control, command **Go**).
5. Druk in QLab op Go — het vlak rechts springt naar de triggerkleur en fade terug.

Gebruik de **"▸ test trigger"**-knop om de werking te testen zonder QLab.

---

## Vereisten

- **Browser:** Chrome of Edge (desktop). Safari ondersteunt de Web MIDI API niet volledig — vermijd dit voor productiegebruik. Firefox ondersteunt Web MIDI niet.
- **Web MIDI API** werkt alleen via `https://`, `localhost`, of een lokaal geopend bestand (`file://`). Een lokaal gedownload en dubbelgeklikt HTML-bestand werkt dus prima.
- Geen installatie, geen build-stap, geen internetverbinding nodig — alles draait clientside in één HTML-bestand.

---

## De app openen

**Aanbevolen voor productie:** download het bestand en open het lokaal (dubbelklik, of sleep in een Chrome-venster). Dit is het meest betrouwbaar voor Web MIDI-toegang en werkt zonder afhankelijkheid van een browser-sandbox of internetverbinding.

Bij elke nieuwe sessie (tabblad gesloten en heropend, of bestand opnieuw geopend) moet de browser opnieuw om MIDI-toestemming vragen. Controleer dit via het MIDI-icoon in de adresbalk, of via `chrome://settings/content/midi`.

> **Let op:** als je een nieuw virtueel MIDI-device aanmaakt (bijv. een IAC-poort) *nadat* de pagina al open stond, kan de devicelijst verouderd zijn. Sluit het tabblad volledig en open opnieuw.

---

## Interface-overzicht

| Onderdeel | Functie |
|---|---|
| **Configuratie** (links) | Alle instellingen: MIDI-ingang, trigger-type, kleuren, timing |
| **Status** (rechts boven) | De tally zelf: kleurvlak, status-tekst, laatste-cue-regel, test-knop |
| **MIDI-log** (rechts onder) | Live log van elk binnenkomend MIDI-bericht, met match-indicatie |

Connectiestatus (rechtsboven in de header) toont hoeveel MIDI-devices gevonden zijn, of een foutmelding als MIDI-toegang faalt.

---

## Trigger-typen configureren

Te kiezen via **Trigger-type** in het configuratiepaneel:

### MIDI Show Control (standaard)

Voor QLab's MSC Go-berichten (en andere MSC-commands). Instelbare velden:

| Veld | Betekenis | Voorbeeld |
|---|---|---|
| **Command** | Het MSC-commando waarop gematcht wordt | Go, Stop, Resume, Timed_Go, Fire, etc. — of "Alle commands" |
| **Device ID** | MSC Device ID van de afzender | `1`, of `*` voor elk device |
| **Cmd Format** | MSC command format (categorie) | Lighting (Generic), Sound (Generic), All-types, of "Alle" |
| **Q Number** | Te matchen cue-nummer | `89`, leeg = alles |
| **Q List** | Te matchen cue-lijst | leeg = alles |
| **Alle Q-nummers accepteren** | Checkbox — negeert Q Number/Q List volledig | aan = match op elk Q-nummer binnen het gekozen command/device |

Device ID `127` (all-call, `7F`) matcht altijd, ongeacht de ingestelde Device ID.

### Elk MIDI-bericht

Trigger op werkelijk alles wat binnenkomt — handig om snel te testen of er überhaupt MIDI-data aankomt.

### Note On / Control Change

Voor eenvoudigere MIDI-setups zonder MSC. Instelbaar op kanaal (1–16 of `*`) en note-/CC-nummer (0–127 of `*`).

---

## Kleuren, hold-tijd en fade-tijd

| Instelling | Werking |
|---|---|
| **Rustkleur** | De kleur waarin het vlak staat als er niets gebeurt |
| **Triggerkleur** | De kleur direct na een match |
| **Hold-tijd** | Hoe lang de triggerkleur **vol** blijft staan voordat het faden begint (standaard 0,1 s) |
| **Fade-tijd** | Hoe lang het duurt om van triggerkleur terug naar rustkleur te faden (standaard 0,5 s) |

Kleuren zijn te wijzigen via de kleurkiezer of direct als hex-code. Beide velden blijven gesynchroniseerd.

**Belangrijk gedrag:**
- De overgang van rust → trigger is **altijd instant** (0 seconden) — dit is niet instelbaar en ook niet nodig, want dat is het hele punt van een tally-light.
- Alleen de overgang trigger → rust (na de hold-tijd) is een fade, met de ingestelde fade-tijd.
- Komt er een nieuwe match binnen terwijl de fade nog bezig is, dan springt het vlak direct terug naar vol de triggerkleur en begint de hold/fade-cyclus opnieuw.

---

## Schermen in-/uitklappen en schermvullende modus

- **Configuratie** en **MIDI-log**: klik op de titelbalk (met chevron-pijltje) om het paneel in of uit te klappen. Instellingen blijven actief, ook als het paneel ingeklapt is.
- **Schermvullende modus**: knop **"⤢ schermvullend"** rechtsboven in het Status-paneel. Verbergt header, footer, configuratie en log; de tally vult het hele browservenster met flink vergrote tekst, geschikt om van afstand af te lezen (bijv. op een aparte monitor of tablet naast de lichttafel).
  - Sluiten via de knop **"✕ sluit schermvullend"** onderin, of met **Esc**.
  - De test-knop blijft ook in schermvullende modus bereikbaar.

> Instellingen (kleuren, timing, paneel-status) worden **niet** bewaard tussen sessies. Bij het opnieuw openen van het bestand start de app met de standaardwaarden.

---

## QLab-kant: MOTU-loopback opzetten

Web MIDI in de browser kan geen virtuele MIDI-poort aanmaken — de browser kan alleen lezen van poorten die het besturingssysteem al kent. Een praktische oplossing is een **hardware-loopback** via een MIDI-interface (bijv. MOTU):

1. Verbind een MIDI **Out**-poort van de interface met een MIDI **In**-poort (fysiek kabeltje, of softwarematig via de eigen control-paneel-software van de interface indien beschikbaar).
2. In QLab: patch je MSC-bron naar de **Out**-poort die je hebt doorgelust.
3. In de tally-app: kies in de MIDI-ingang-dropdown de **In**-poort waar de loopback op terugkomt.

Alternatief: gebruik een **IAC-bus** (Audio MIDI Setup → MIDI Studio → IAC Driver → "Device is online" aanvinken, poort toevoegen). Dit is een puur virtuele MIDI-kabel binnen macOS en vereist geen fysieke hardware-loopback, maar werkt alleen voor signalen die softwarematig blijven (dus niet voor een fysieke lichttafel).

---

## QLab-kant: MSC Show Control Broadcast

Naast losse, handmatig aangemaakte MSC-cues heeft QLab een ingebouwde **Show Control Broadcast**-functie die automatisch een MSC GO-bericht stuurt bij **elke** Go in de cue-lijst — zonder dat je daarvoor losse cues hoeft aan te maken.

**Instellen:**

1. `QLab → Workspace Settings → MIDI → MSC Broadcast`-tab.
2. Vink de checkbox aan om broadcast voor de workspace te activeren.
3. Klik **New Destination** (⌘N).
4. Kies de gewenste **MIDI Patch** (bijv. een IAC-bus die naar de tally-app wijst), **Command Format**, en **Device ID**.

Let op: het Q-nummer dat hierbij verstuurd wordt is het **QLab-cuenummer** zoals getoond in de cue-lijst — dit hoeft niet overeen te komen met een Q-nummer dat je eerder handmatig in een losse MSC-cue had ingesteld. Gebruik in dat geval de optie **"Alle Q-nummers accepteren"** in de tally-app, of stem het Q-nummer-veld af op de daadwerkelijk binnenkomende waarde (zichtbaar in de MIDI-log van de app).

---

## QLab-kant: lichttafel én tally tegelijk

Een MSC-cue in QLab 5 kan maar naar **één** MIDI-patch tegelijk wijzen — er is geen ingebouwde manier om één cue naar twee destinations te sturen. De aanbevolen, schoonste oplossing zonder extra software:

- **Handmatige MSC-cues** → blijven gepatcht naar de lichttafel, zoals altijd.
- **Show Control Broadcast** → apart ingesteld op een eigen MIDI-patch (bijv. een IAC-bus), die naar de tally-app gaat.

Zo lopen beide stromen volledig gescheiden, zonder kans op dubbele berichten naar de lichttafel, en zonder extra routing-software (zoals MIDI Patchbay of MidiPipe) nodig te hebben.

---

## Problemen oplossen

| Symptoom | Mogelijke oorzaak |
|---|---|
| Dropdown toont "geen devices gevonden" | MIDI-toegang geweigerd, of device aangemaakt ná het openen van de pagina — sluit tabblad en open opnieuw |
| Tally reageert niet, MIDI-log blijft leeg | Verkeerde MIDI-ingang geselecteerd, of QLab stuurt naar een andere patch/poort dan verwacht |
| Tally reageert, maar nooit als "match" (grijs bolletje in log) | Trigger-instellingen (Command/Device ID/Q-nummer) komen niet overeen met het binnenkomende bericht — controleer de exacte waarden in de MIDI-log |
| Kleurovergang voelt traag/dof i.p.v. een felle flits | Controleer of hold-tijd en fade-tijd naar wens staan; rust → trigger is altijd instant, alleen trigger → rust is een fade |
| Browser vraagt geen MIDI-toestemming | Ga naar `chrome://settings/content/midi` en controleer of de site niet op "Block" staat |

Gebruik de **MIDI-log** als eerste diagnosemiddel: elk binnenkomend bericht wordt getoond met een groen bolletje (●, match → trigger) of grijs bolletje (○, genegeerd), inclusief de geparste MSC-velden (device, command, Q-nummer, Q-lijst) zodat je precies kunt zien wat er binnenkomt versus wat je filter verwacht.

---

## Technische achtergrond

- Eén zelfstandig HTML-bestand, geen externe dependencies, geen build-stap.
- MIDI-ontvangst via de **Web MIDI API** (`navigator.requestMIDIAccess`), inclusief SysEx-toegang voor MSC-parsing.
- MSC-berichten worden geparsed volgens de structuur `F0 7F <device_id> 02 <cmd_format> <command> [<Q_number>] [<Q_list>] F7`.
- Kleurovergangen worden volledig door JavaScript aangestuurd via `requestAnimationFrame`, niet via CSS-transities — dit garandeert dat de trigger-overgang altijd in één frame instant is, ongeacht her-triggers tijdens een lopende fade.
- Geen serverkant, geen dataopslag, geen netwerkverkeer — alles blijft lokaal in de browser.
