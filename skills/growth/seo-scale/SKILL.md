---
name: seo-scale
description: Keyword-Recherche und SEO-Potenzialanalyse mit echten Daten (Ubersuggest MCP). Nutze diesen Skill IMMER, wenn der Nutzer Keywords prüfen, Suchvolumen/SEO-Difficulty/CPC abfragen, das SEO-Potenzial einer Produkt- oder Geschäftsidee bewerten, eine Keyword-Strategie oder Seitenarchitektur für SEO entwickeln, oder /seo aufrufen will — auch wenn er nur beiläufig fragt "lohnt sich das für Google" oder "wonach suchen die Leute". Gilt für alle Märkte und Sprachen (DE, US, …).
---

# SEO: Keyword-Recherche & Potenzialanalyse

Ziel: Aus einer Produktidee oder Keyword-Liste eine ehrliche, datenbasierte Einschätzung machen, welche Keywords echtes Potenzial haben — und daraus eine umsetzbare Strategie ableiten.

## Grundprinzipien (immer anwenden)

1. **Aufgabe statt Anlass.** Menschen googeln die generische Aufgabe ("videos zusammenfügen"), nicht den Anlass ("urlaubsvideo erstellen"). Nischen-/Anlass-Keywords haben fast immer wenig Volumen. Konsequenz: SEO auf die Aufgaben-Keywords, Positionierung über Branding/Messaging.
2. **Ehrlich sein.** 50 Suchen/Monat sind kein Volumen — das klar sagen, nicht schönreden. Nullvolumen-Keywords als tot markieren und breitere Alternativen suchen.
3. **Difficulty × Intent schlägt Volumen.** Ein Keyword mit 300 Suchen, Difficulty 25 und transaktionaler Intention ist wertvoller als 30.000 Suchen bei Difficulty 75.
4. **CPC ist ein Kaufintentions-Signal.** Hoher CPC bei niedriger Paid Difficulty = unterversorgte Kaufintention (guter Fund).
5. **Saisonalität aus monthly_searches lesen** und in die Empfehlung einbauen (Publikationstiming).
6. **Eine Seite besitzt das Cluster — nicht eine Seite pro Keyword.** Siehe unten, das ist der häufigste und teuerste Architekturfehler.
7. **Verlauf schlägt Momentaufnahme.** `domain_overview` liefert 24 Monate `domainTraffic`. Immer lesen. Eine Domain mit hoher DA kann seit zwei Jahren einbrechen, eine mit DA 12 kann gerade senkrecht steigen. Die Momentaufnahme sortiert beide falsch ein.

## Workflow

### 1. Kontext klären
Produkt, Zielgruppe, Zielmärkte/Sprachen. Bei Keyword-Listen vom Nutzer: pro Markt die richtige Sprache/locId verwenden (DE: language "de", locId 2276; US: language "en", locId 2840; weitere per location_suggest).

### 2. Daten ziehen (Ubersuggest MCP)
- Einzelne Keywords: `keyword_overview` pro Keyword (liefert search_volume, seo_difficulty, paid_difficulty, cpc, search_intent, monthly_searches).
- Breite Recherche: `match_keywords` mit Seed-Begriffen. **Fallback:** Wenn match_keywords fehlschlägt (z. B. Freigabe-Fehler), stattdessen selbst breitere Kandidaten ableiten und einzeln per keyword_overview prüfen — das funktioniert zuverlässig.
- Bei Nullvolumen: nicht aufgeben, sondern generischere Umformulierungen testen (kürzere Phrasen, Aufgabe statt Anlass).
- Optional: `serp_analysis` für die Top-Kandidaten (wer rankt, wie schlagbar), `estimate_serp_clicks` für Traffic-Projektionen.

### 3. Struktur lesen — bei der eigenen Domain und bei 2–3 Wettbewerbern
Keyword-Metriken allein sagen nichts über Seitenarchitektur. Diesen Schritt nie überspringen, wenn es um Strategie oder Architektur geht.

- `domain_overview` liefert `organicKeywords[]` **mit `path`**. Das `path`-Feld auszählen: Auf wie vielen URLs verteilen sich die Top-50-Keywords? Konzentriert sich alles auf `/`, oder streut es über hunderte Seiten?
- Sitemap ziehen (`robots.txt` → Sitemap-URL; Astro/Next nutzen oft `sitemap-index.xml`, nicht `sitemap.xml`; bei Bot-Schutz curl-403 → Browser-Tools nutzen) und URLs pro Verzeichnis auszählen.
- **Traffic je Seite ausrechnen** = `traffic` ÷ Sitemap-URLs. Das ist die diagnostischste Einzelzahl im ganzen Skill. Unter ~5 ist die Seitenfläche Halde, egal wie gut die Keyword-Auswahl aussieht.
- `domainTraffic` über 24 Monate auf Richtung prüfen (Prinzip 7).

### 4. Bewerten und einteilen
Kategorien:
- **Prio 1 (Kern):** ordentliches Volumen (ab ~1.000/Monat) + Difficulty machbar + Produkt-Fit.
- **Prio 2 (schnelle Wins):** Difficulty < ~40, klare Intention, Volumen egal wie klein — billige Landingpages.
- **Langfrist:** hohes Volumen, Difficulty > ~60 — erst mit Domain-Autorität angreifen.
- **Streichen:** Volumen 0 oder ohne Produkt-Fit.

**Zur Difficulty-Schwelle:** SD ist ein Faktor, nicht das Tor. Referenzdaten unten zeigen eine DA-12-Domain auf Position 1 bei SD 48 und eine DA-9-Domain auf Position 95 bei SD 18. Was tatsächlich entscheidet, ist die Frage aus dem nächsten Abschnitt: Ist genau **eine** Seite die vollständige Antwort auf die Suche, ohne Reibung? Wenn ja, sind SD 50 erreichbar. Wenn nein, helfen SD 18 auch nicht.

### 5. Output-Format
- Pro Markt eine Tabelle, **nach Volumen sortiert**: Keyword | Volumen/Monat | SEO Difficulty | CPC | Intent. Trends (↗) und Saison-Peaks annotieren.
- Danach ehrliche Einschätzung in Prosa: was hat echtes Potenzial und warum, was ist zu hart, was ist tot.
- Strategieempfehlung ableiten (siehe unten).

## Architektur: eine Seite besitzt das Cluster

**Die Regel:** Pro Sprache besitzt **eine** Tool-/Kernseite das gesamte Head-Cluster — alle Formulierungsvarianten desselben Bedürfnisses. Eigene URLs bekommen nur **echte andere Intents**, nicht Keyword-Varianten.

Echter anderer Intent (eigene URL gerechtfertigt):
- anderer Modus des Produkts (`/online/` für Mehrspieler)
- anderes Inhaltsbedürfnis (`/…-words/` für eine Wortliste statt des Tools)
- andere Sprache
- Nachbarpublikum (siehe Content-Hebel)

Keine eigene URL: `…-service`, `…-kostenlos`, `…-online`, `…-app`, Singular/Plural, Synonyme. Die besitzt die Kernseite mit.

**Warum:** Auf einer Domain mit wenig Autorität teilt jede zusätzliche Seite denselben kleinen Topf. Zehn Seiten bei Position 90 sind wertlos; eine Seite bei Position 3 trägt alles. Bündelung ist der Hebel, der ohne Links funktioniert.

**Gegenprobe vor jeder Empfehlung:** Würde diese neue URL eine Suche beantworten, die die bestehende Kernseite *nicht* schon vollständig beantwortet? Wenn nein — Abschnitt auf der Kernseite, keine neue Seite.

### Referenzdaten (Stand 09/2026, zur Kalibrierung)

| Domain | DA | Ref. Domains | Seiten | Traffic/Mon. | Traffic je Seite | Modell |
|---|---|---|---|---|---|---|
| imposter.app | 12 | 165 | ~400 (20 × 21 Sprachen) | 248.688 | **622** | 1 Kernseite + Satelliten |
| anonymoustext.com | 22 | 743 | 45 | 15.533 | 345 | 1 Kernseite + Blog |
| fooh.com | 20 | 238 | 7.085 | 686 | **0,1** | programmatisch, 1 Seite je Instanz |

fooh.com hat 177× mehr Seiten und mehr verweisende Domains als imposter.app — und 1/362 des Traffics. Seitenzahl erzeugt keinen Traffic.

imposter.app steht mit DA 12 auf **Position 1 für "imposter game"** (165.000/Monat, SD 48), ~35 der Top-50-Keywords auf der Homepage. Exact-Match-Domain und sofortige Nutzbarkeit ohne Signup sind dabei mitentscheidend — beides mit abfragen, wenn ein Wettbewerber unerklärlich gut rankt.

Beide etablierten Domains schrumpfen (−93 % bzw. −69 % seit Peak), und in beiden Fällen ist der dünne Longtail weggebrochen, während das Kern-Cluster hielt. **Nie Contentmenge als Strategie empfehlen.**

## Strategie-Bausteine für die Empfehlung

- **Tool-Page-SEO:** Wenn das Produkt eine Web-App ist, ist die Tool-Page das stärkste Muster: kostenlose Funktion direkt auf der Keyword-URL, sofort nutzbar (Clideo/Kapwing-Playbook). **Eine** solche Seite pro Sprache, die das Cluster besitzt — siehe Architektur-Abschnitt.
- **Reibung prüfen.** Wenn zwischen Suchergebnis und sichtbarem Nutzen ein Signup, ein Credit-Gate oder ein Upload-Zwang steht, ist das ein Ranking-Thema, nicht nur ein Conversion-Thema. Immer ansprechen, wenn eine gut gewählte Seite trotzdem nicht rankt.
- **Sprachen als Skalierungsachse, nicht Keywords.** Bei sprachunabhängigen Produkten kostet eine weitere Sprache die Übersetzung einer gewinnenden Seite, nicht neuen Content — der billigste Hebel überhaupt. Vor der Empfehlung Volumen im Zielmarkt prüfen, nicht annehmen. (imposter.app: 21 Sprachen derselben ~20 Seiten.)
- **Content-Hebel = Nachbarpublikum, nicht breitere Keyword-Varianten.** Seiten/Posts, die dieselbe Zielgruppe über *angrenzende Themen* abholen und zum Produkt überleiten — nicht Umformulierungen des Kern-Keywords (die gehören auf die Kernseite). Belegte Beispiele: "games like among us", "party games for adults" für ein Partyspiel; Santa-/April-Fools-Texte für einen SMS-Dienst.
- **Vergleichsseiten:** "[Wettbewerber] Alternative"-Keywords haben oft niedrige Difficulty und hohe Kaufintention.
- **App-Intent-Keywords** ("… app") priorisieren, wenn das Produkt eine App ist — kürzester Weg von Suche zu Install.
- **Saisonseiten** rechtzeitig vor dem Peak publizieren (aus monthly_searches ablesen). Saisonale Einzelseiten sind überdurchschnittlich oft die stärksten Nicht-Kernseiten.
- **Paid-Hinweis:** Keywords mit hohem CPC + niedriger Paid Difficulty als günstige Ads-Tests markieren.

## Timing-Disziplin

Nach einem Architekturwechsel mindestens 4 Wochen messen, bevor erneut umgebaut wird. Zwei Umbauten hintereinander machen beide Messungen wertlos. Wenn eine Analyse mitten in ein laufendes Experiment fällt: Befund als Hypothese festhalten, Umsetzung terminieren — nicht sofort handeln.

Search-Console-Fenster nie über das Deploy-Datum legen. Ein Export, der vorher und nachher mischt, beantwortet keine der beiden Fragen.

## Was dieser Skill NICHT tut
Keine Rankings garantieren, keine Volumina schätzen ohne Daten (immer Ubersuggest abfragen), keine Difficulty-60+-Keywords als "machbar" für neue Domains verkaufen, und keine Seitenarchitektur empfehlen, ohne vorher die Struktur von 2–3 Wettbewerbern gemessen zu haben.
