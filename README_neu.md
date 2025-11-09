# arc42 Architektur-Dokumentation  
## Minecraft – Referenzarchitektur


## 1. Einfuehrung und Ziele

### 1.1 Beschreibung der Anwendung
Minecraft ist eine Open-World-Sandbox-Anwendung, in der Benutzer eine virtuelle Welt aus Blöcken erkunden, gestalten und verändern können.
Das System kombiniert Elemente von Simulation, Kreativität, Überleben und Mehrspieler-Interaktion.
Minecraft ist in Java (für die Java Edition) bzw. in C++ (für die Bedrock Edition) entwickelt und läuft plattformübergreifend auf Windows, macOS, Linux, Konsolen und Mobilgeräten.

### 1.2 Aufgabenstellung

Ziel der Anwendung ist es, den Benutzerinnen und Benutzern eine interaktive, offene Umgebung zu bieten, in der sie eigene Welten erschaffen, Ressourcen sammeln, Gegenstände herstellen und gemeinsam mit anderen Spielern interagieren können.
Minecraft fördert damit Kreativität, Problemlösung und Zusammenarbeit. Die Architektur muss:

- grosse, persistente Welten effizient verwalten
- Einzel- und Mehrspielermodi unterstuetzen
- eine aktive Modding- und Server-Community ermoeglichen
- auf vielen Plattformen lauffaehig bleiben
- ueber Jahre erweiterbar, wartbar und stabil sein

Diese Dokumentation beschreibt eine konsolidierte, didaktische Architektur von:

- Minecraft: Java Edition  
- Minecraft: Bedrock Edition  
- Minecraft Education, Realms, Dedicated Server

mit Fokus auf strukturelle Muster, zentrale Bausteine und Qualitaetsanforderungen.

### 1.3 Qualitaetsziele

1. **Performance & Skalierung**  
   Fluessiges Spielerlebnis auch bei grossen Welten, vielen Entitaeten und vielen Spielenden.
2. **Stabilitaet & Datenintegritaet**  
   Persistente Welten ohne relevante Datenverluste, robuster Serverbetrieb.
3. **Erweiterbarkeit & Offenheit**  
   Kontrollierte Erweiterung durch Mods, Add-ons und Plugins ohne Anpassung des Kerncodes.
4. **Sicherheit & Fairness**  
   Schutz vor Cheating, Missbrauch und unautorisierten Zugriffen (insbesondere im Multiplayer).
5. **Plattformvielfalt & Konsistenz**  
   Vergleichbares Spielerlebnis auf PC, Konsole, Mobile und in Bildungsszenarien.

### 1.4 Stakeholder

- Spielende (Casual, Pro, Kreativ, Survival)
- Community-Entwicklerinnen und -Entwickler (Mods, Plugins, Tools)
- Server-Administratoren / Hoster
- Mojang Studios / Microsoft (Produkt, Marke, Erloes, Compliance)
- Lehrpersonen und Bildungseinrichtungen (Education Edition)
- Plattformbetreiber (Xbox, PlayStation, Nintendo, App Stores)

## 2. Randbedingungen

### 2.1 Technische Randbedingungen

- **Programmiersprachen**
  - Java Edition: Java
  - Bedrock Edition: C++ und plattformspezifische Komponenten
- **Architekturprinzip**
  - Durchgaengig Client-Server-Modell
  - Singleplayer (Java): lokaler integrierter Server
  - Multiplayer: dedizierter oder Realms-Server
- **Persistenz**
  - Welt in Chunks und Regionen organisiert
  - Binäre Formate (z. B. NBT) fuer Welten, Spieler, Konfiguration
- **Netzwerk**
  - TCP/IP-basierte Kommunikation
  - Paketorientiertes proprietaeres Client-Server-Protokoll
- **Plattformen**
  - Windows, macOS, Linux
  - Xbox, PlayStation, Nintendo Switch
  - iOS, Android
- **Build & Delivery**
  - Zentrale Distribution via Minecraft Launcher, Plattform-Stores, Online-Dienste

### 2.2 Organisatorische Randbedingungen

- Produktverantwortung bei Mojang Studios (Microsoft)
- Zentrales Release- und Patch-Management
- Langfristige Wartung mit Ruecksicht auf bestehende Welten und Community
- Enge Einbindung und Erwartungshaltung einer sehr grossen Nutzer- und Modding-Community

### 2.3 Rechtliche Randbedingungen

- Proprietaere Lizenz, EULA und Markenrichtlinien von Mojang/Microsoft
- Richtlinien fuer Serverbetreiber (Monetarisierung, Transparenz, faire Nutzung)
- Einhaltung von Datenschutz- und Compliance-Vorgaben fuer Online-Dienste
- Einhaltung von Plattformrichtlinien (Konsolen, App-Stores, Online-Services)

## 3. Kontextabgrenzung

### 3.1 Systemuebersicht

**Im Scope dieser Architekturbetrachtung:**

- Minecraft-Client (Rendering, UI, Eingabe, Client-Netzwerk)
- Minecraft-Server (integriert oder dediziert)
- Persistenzschicht (Weltdaten, Spielstaende, Konfiguration)
- Offizielle Erweiterungsmechanismen (Resource Packs, Datapacks, Add-ons, APIs)

**Ausserhalb, aber relevant:**

- Microsoft-/Xbox-Account- und Authentifizierungsdienste
- Minecraft Launcher, Plattform-Stores, Lizenzpruefung
- Community- / Dritt-Server, Hoster
- Modding- und Plugin-Plattformen
- Education-spezifische Verwaltungsplattformen

### 3.2 Externe Akteure und Systeme

| Akteur / System                | Beschreibung                                      | Interaktion                               |
|--------------------------------|---------------------------------------------------|-------------------------------------------|
| Spielende                      | Endnutzerinnen und Endnutzer                     | Spielen, Eingaben, Konten, Kauefe         |
| Launcher / Store               | Distribution, Lizenz, Updates                    | Login, Download, Patchen                  |
| Microsoft-/Xbox-Services       | Identitaet, Multiplayer, Profil, Realms          | Authentifizierung, Session-Verwaltung     |
| Community-/Dedicated-Server    | Externe Mehrspielerumgebungen                    | Client-Verbindung via Protokoll           |
| Modding-/Plugin-Plattformen    | Distribution von Mods, Plugins, Tools            | Download, Dokumentation, Updates          |
| Marketplace / Content-Plattform| Offizielle Inhalte (Skins, Welten, Add-ons)      | Kauefe, Lizenzpruefung, Downloads         |
| Bildungseinrichtungen          | Nutzung der Education Edition                    | Steuerung von Welten, Nutzerkonten        |

### 3.3 Kontextdiagramm (verbale Beschreibung)

- Der **Client** kommuniziert mit:
  - einem **Minecraft-Server** (integriert, dediziert oder Realms)
  - **Launcher** und Plattformdiensten fuer Start, Updates, Lizenz
  - **Auth-Services** fuer Kontopruefung und Multiplayerfreischaltung
- Der **Server**:
  - verwaltet Welt- und Spielerzustand
  - kommuniziert mit Persistenz (Dateien)
  - nutzt optional Modding-/Plugin-Schnittstellen
- Externe Systeme (Stores, Marketplace, Modding-Plattformen) liefern Inhalte, aber laufen ausserhalb des Kernsystems.

## 4. Loesungsstrategie

### 4.1 Architekturstil und Leitprinzipien

- **Client-Server-Architektur** mit serverseitiger Autoritaet
- **Komponentenbasierte Struktur** in Client, Server und Persistenz
- **Ereignisgesteuerte Architektur**:
  - Events und Hooks fuer Interaktionen und Erweiterungen
- **Datenorientiertes Design**:
  - Welten, Entitaeten und Regeln als Datenmodelle
- **Produktlinien-Ansatz**:
  - Gemeinsame Grundprinzipien fuer Java und Bedrock, unterschiedliche Implementierung nach Plattformbedarf

### 4.2 Strategien zur Erreichung der Qualitaetsziele

- **Performance**
  - Chunk-basiertes Laden/Speichern und Streaming
  - Begrenzte Sicht- und Simulation-Distanzen
  - Optimierte Tick-Verarbeitung und Entitaetsverwaltung
- **Stabilitaet**
  - Regelmaessige Autosaves
  - Crash-Reports, definierte Schreibzyklen
  - Trennung kritischer Aufgaben in eigene Threads
- **Erweiterbarkeit**
  - Offizielle APIs, Datapacks, Add-ons, Resource Packs
  - Klare Trennung von Kern und Erweiterungen
- **Sicherheit**
  - Serverseitige Validierung aller kritischen Aktionen
  - Authentifizierung ueber zentrale Dienste (Online-Modus)
  - Konfigurierbare Rechte und Rollen
- **Plattformvielfalt**
  - Abstraktionsschichten fuer Grafik, Eingabe, Dateisystem, Netzwerk

## 5. Bausteinsicht (Building Block View)

### 5.1 Gesamtuebersicht
+-----------------------------------------------------------+
|                     Minecraft System                      |
+-----------------------------------------------------------+
|  Client Layer   |  Server Layer   |  Persistence Layer    |
+-----------------------------------------------------------+
|          Cross-Cutting & Extension Layer (APIs, Mods)     |
+-----------------------------------------------------------+

### 5.2 Client Layer
Baustein: Client

**Verantwortungen:**
- Darstellung der Spielwelt (3D-Rendering)
- Anzeige von Benutzeroberflaechen (HUD, Inventar, Menues)
- Verarbeitung von Eingaben (Keyboard, Maus, Controller, Touch)
- Verwaltung lokaler Ressourcen (Texturen, Sounds, Modelle, Shader)
- Aufbau, Ueberwachung und Beendigung der Netzwerkverbindung zum Server
- (Java Singleplayer) Start und Betrieb eines integrierten lokalen Servers

**Untermodule:**
- Rendering Engine
- UI / HUD / Menue-System
- Input-Abstraktion
- Client-Netzwerk-Adapter
- Lokales Chunk- und Asset-Caching

**Schnittstellen:**
- Netzwerk: Senden von Aktionen, Empfangen von Welt- und Statusupdates
- User Input: abstrahierte Eingabeereignisse
- Resource Management: Laden und Anwenden von Resource Packs

### 5.3 Server Layer
Baustein: Server

**Verantwortungen:**
- Autoritative Spiellogik:
  - Validierung von Bewegungen, Interaktionen, Kaempfen, Blockaktionen
- Verwaltung der Welten:
  - Chunks, Biome, Strukturen, Dimensionen
- Verwaltung von Entitaeten:
  - Spielende, Mobs, Tiere, NPCs, Projektile, Items
- Session-Management:
  - Login, Logout, Timeouts, Multiplayer-Handling
- Synchronisation:
  - Verteilung relevanter Updates an verbundene Clients
- Rechte- und Administrationsfunktionen:
  - Operatoren, Whitelists, Bans, Commands
- Plugin-/Mod-Integration (insbesondere Java Dedicated Server)

**Untermodule:**
- Game Loop / Tick Engine
- World Manager
- Entity Manager
- Rule & Game Mode Engine
- Network Server Endpoint
- Command & Admin Interface
- Plugin/Mod Integration Layer

**Schnittstellen:**
- Netzwerkprotokoll zu Clients
- Administrative Interfaces (Konsole, RCON, Konfigurationsdateien)
- APIs fuer Plugins/Mods

### 5.4 Persistence Layer
Baustein: Persistence

**Verantwortungen:**
- Speicherung von Weltdaten:
  - Chunks, Regionen, Strukturen, Dimensionen
- Speicherung von Spielerprofilen:
  - Position, Inventar, Statistiken, Einstellungen
- Speicherung von Konfiguration:
  - server.properties, Whitelist, Bans, Gamerules
- Speicherung von Plugin-/Mod-Konfigurationen
- Unterstuetzung von Backups, Migration, Wartung

**Kernkonzepte:**
- NBT und verwandte Datenformate
- Datei- und Verzeichnisstruktur pro Weltinstanz
- Transaktionsaehnliche Schreibstrategien fuer Konsistenz

**Schnittstellen:**
- Funktionen zum Laden/Speichern von Chunks und Spielerinfos
- Zugriff durch Server-Logik und Erweiterungen (kontrolliert)

### 5.5 Netzwerk & Protokoll
Baustein: Netzwerkkommunikation

***Verantwortungen:**
- Aufbau, Ueberwachung und Abbau von Client-Server-Verbindungen
- Authentifizierungs- und Versions-Handshake
- Serialisierung/Deserialisierung von Paketen (Bewegung, Chat, Weltupdates)
- Umgang mit Paketverlust, Latenz, Timeouts
- Sicherstellung von Reihenfolge und Integritaet (TCP-basiert bzw. spezialisierte Implementierungen)

**Schnittstellen:**
- Gegenueber Client:
  - Definierte Nachrichtenstrukturen (Login, Join, ChunkData, EntityUpdate, Chat, etc.)
- Gegenueber Serverlogik:
  - Callback-Mechanismen fuer eingehende Requests
  - API fuer Versand von Updates

### 5.6 Extension & Modding Layer
  Baustein: Extension & Modding Layer

**Verantwortungen:**
- Bereitstellung definierter Erweiterungspunkte:
  - Events (z. B. PlayerJoin, BlockBreak, EntityDamage, Tick)
  - Registrierungsmechanismen fuer neue Bloecke, Items, Rezepte, Loot-Tabellen
  - Erweiterung von Chat- und Command-Funktionalitaet
- Kapselung und Isolierung von Erweiterungscode
- Unterstuetzung von Community-Content bei gleichzeitigem Schutz des Kernsystems

**Schnittstellen:**
- Java Edition:
  - Forge, Fabric, Bukkit/Spigot/Paper APIs
  - Datapacks und Resource Packs
- Bedrock Edition:
  - Add-ons, Behavior Packs, Resource Packs, Skript-APIs

## 6. Laufzeitsicht (Runtime View)

### 6.1 Szenario: Multiplayer-Login
1. Client startet und laedt Ressourcen.
2. Spielende waehlen einen Server aus.
3. Client baut TCP/IP-Verbindung zum Server auf.
4. Handshake:
   - Protokollversionen abgleichen
   - Authentifizierung via Microsoft-/Xbox-Services (Online-Modus)
5. Server prueft Berechtigungen und laedt Spielerprofil.
6. Server sendet:
   - Startposition
   - relevante Chunks
   - Entitaetsinformationen
7. Client rendert Welt; Laufbetrieb beginnt:
   - Client sendet Eingaben (Bewegung, Aktionen, Chat).
   - Server validiert, aktualisiert Weltzustand.
   - Server broadcastet relevante Updates an betroffene Clients.

### 6.2 Szenario: Block platzieren
1. Spieler waehlt Block im Inventar.
2. Client sendet PlaceBlock-Request mit Position, Item, Blickrichtung.
3. Server prueft:
   - Reichweite, Kollision, Schutzbereiche
   - Inventar und Spielmodus
4. Bei Erfolg:
   - Server aktualisiert Chunk-Daten.
   - Server passt Inventar an.
   - Server sendet Update an Clients im Sichtbereich.
5. Clients aktualisieren Rendering des betroffenen Blocks.

### 6.3 Szenario: Plugin-Event (Java Dedicated Server)
1. Spieler joint den Server.
2. Server erzeugt PlayerJoinEvent.
3. Modding-Layer ruft registrierte Listener in Plugins auf.
4. Plugins fuehren ihre Logik aus (z. B. Begruessungsnachrichten, Statistiken, Checks).
5. Anpassungen erfolgen ueber offizielle APIs; Kernlogik bleibt unveraendert.

## 7. Verteilungssicht (Deployment View)
### 7.1 Deployment-Varianten
1. Einzelspieler (Java Edition) - Client und integrierter Server auf demselben Geraet.
2. LAN-Server - Separater Server im lokalen Netzwerk, mehrere Clients im selben LAN.
3. Dedizierter Online-Server - Server auf Root-Server, VPS oder Cloud-Plattform; globale Erreichbarkeit.
4. Realms - Von Mojang/Microsoft verwaltete Serverinstanzen.
5. Education Edition - Spezifische Builds mit zentralen Verwaltungsfunktionen fuer Klassen.

### 7.2 Beispielhafte Deployment-Ansicht
+--------------------------------------------------------------+
|                        Internet / Netzwerk                   |
+--------------------------------------------------------------+
|  +-----------------+       TCP/IP       +------------------+ |
|  | Minecraft Client| <----------------> | Minecraft Server | |
|  | (PC/Konsole)    |                    | (VPS / lokal)    | |
|  +-----------------+                    +------------------+ |
|         |                                         |          |
|         | HTTPS (Auth, Profile, Realms)           | FS       |
|         v                                         v          |
|  +-------------------+                    +----------------+ |
|  | Microsoft-/Xbox   |                    | Welt-/Spiel-   | |
|  | Auth Services     |                    | Daten (NBT)    | |
|  +-------------------+                    +----------------+ |
+--------------------------------------------------------------+

## 8. Querschnittliche Konzepte

Dieses Kapitel beschreibt grundlegende Prinzipien und Mechanismen, die sich nicht auf einen einzelnen Baustein beschraenken, sondern **client-, server- und produktlinienuebergreifend** wirken. Sie sind entscheidend, damit Minecraft in unterschiedlichen Umgebungen (lokal, dediziert, Realms, Education, Community-Server) langfristig stabil, sicher, performant und erweiterbar betrieben werden kann.

### 8.1 Sicherheitskonzept

Das Sicherheitsmodell von Minecraft basiert auf dem Grundsatz:

> **Die autoritative Wahrheit liegt immer beim Server, nie beim Client.**

Der Client wird als potenziell unsicher betrachtet. Daraus ergeben sich folgende Kernprinzipien:

**Serverseitige Autoritaet**

- Alle sicherheitsrelevanten Entscheidungen (z. B. Bewegung, Schaden, Blockaktionen, Inventar) werden serverseitig geprueft.
- Der Server verwirft ungueltige oder physikalisch/unlogisch unmoegliche Aktionen (z. B. zu schnelles Bewegen, unerlaubte Flughoehe, Items aus dem Nichts).

**Authentifizierung und Identitaet**

- Online-Modus (Java) und Bedrock nutzen **Microsoft-/Xbox-Accounts** als zentrale Identitaet.
- Im Rahmen der Authentifizierung prueft der Server:
  - Gueltigkeit des Tokens (kein Spoofing)
  - Uebereinstimmung von Account-ID und Spielernamen (je nach Edition/Modus)
- Offline- oder LAN-Server koennen ohne zentrale Authentifizierung betrieben werden, tragen jedoch hoehere Sicherheitsrisiken.

**Zugriffs- und Berechtigungssysteme**

- Whitelists zur Beschraenkung auf bekannte Spielende
- Bans (IP-/Account-basiert) zur Sperrung auffaelliger Nutzer
- Rollen- und Berechtigungssysteme (Operatoren, Permissions via Plugins), um administrative Aktionen zu kontrollieren

**Netzwerksicherheit**

- Validierung eingehender Pakete:
  - Struktur, Laenge, Frequenz (Rate Limiting)
  - Schutz vor Paket-Flooding und Protokollmissbrauch
- Erkennung von typischen Cheat-Mustern (z. B. No-Clip, Kill-Aura, Item-Duplikation) durch serverseitige Logik oder externe Anti-Cheat-Plugins
- Optionale Verschluesselung je nach Edition, Hosting-Umgebung und Infrastruktur

**Integritaet und Vertrauensgrenzen**

- Der Client gilt nie als vertrauenswuerdige Quelle fuer:
  - Inventarinhalte
  - Block-/Entitaetszustaende
  - Positions- und Geschwindigkeitsdaten
- Erweiterungen (Mods/Plugins) werden als **lokale Vertrauensentscheidung** des Serverbetreibers betrachtet. Unsichere Plugins koennen:
  - Weltdaten beschaedigen
  - Sicherheitsluecken eroeffnen
  - Performance massiv beeintraechtigen

Serverbetreiber muessen daher:
- nur vertrauenswuerdige Erweiterungen verwenden
- regelmaessig Updates einspielen
- Zugriff auf Serverkonsole und Dateien klar beschraenken

### 8.2 Konfigurationskonzept

Das Konfigurationskonzept ermoeglicht eine flexible Anpassung des Spielverhaltens, ohne den Anwendungscode zu aendern. Es folgt dem Prinzip:

> **Verhalten via Daten und Dateien steuerbar machen, nicht via Code-Aenderungen.**

**Konfigurationsebenen**

1. **Server-Basis-Konfiguration**
   - `server.properties` (Java) bzw. aehnliche Dateien:
     - Port, Online-/Offline-Modus, Slots, Schwierigkeitsgrad, Weltname, Sichtweite, etc.
2. **Welt- und Regelkonfiguration**
   - **Gamerules** (z. B. KeepInventory, MobGriefing, RandomTickSpeed)
   - Pro Welt einstellbar, zur Feinsteuerung von Mechaniken
3. **Erweiterungskonfiguration**
   - Plugins/Mods/Add-ons nutzen eigene Konfigurationsdateien (z. B. YAML, JSON)
   - Beispiel: Rechte, Features, Limits, Nachrichten
4. **Ressourcen- & Datapacks**
   - Resource Packs: visuelle und auditive Anpassungen (Texturen, Sounds, UI-Elemente)
   - Datapacks: Definition von Rezepten, Loot-Tabellen, Funktionen, Fortschritten, Strukturen
   - Ermoeglichen Daten-getriebene Anpassungen auf Server- oder Welt-Ebene

**Konfigurationsprinzipien**

- Trennung von:
  - **globalen Servereinstellungen**
  - **weltbezogenen Regeln**
  - **mod-/plugin-spezifischen Optionen**
- Moeglichkeit unterschiedlicher Profile (z. B. Entwicklungs-/Test-/Produktivserver)
- Aenderungen sollen moeglichst ohne Code-Deploy und teils zur Laufzeit (z. B. via Commands) moeglich sein.

### 8.3 Fehler- und Loggingkonzept

Ein konsistentes Logging- und Fehlerhandling ist zentral fuer Stabilitaet, Support und Analyse.

**Logging-Grundsaetze**

- Jeder Minecraft-Server fuehrt zentrale Logdateien mit:
  - Systemmeldungen (Start/Stop, Welt-Ladevorgaenge)
  - Fehlermeldungen und Stacktraces
  - Befehlen, wichtigen Events und Warnungen
- Logs dienen:
  - zur Ursachenanalyse bei Abstuerzen oder Performanceproblemen
  - zur Nachvollziehbarkeit administrativer Aktionen
  - zur Sicherheitsanalyse (z. B. bei Missbrauch)

**Crash-Handling**

- Generierung von Crash-Reports:
  - Umgebung (Version, Mods/Plugins)
  - Stacktraces, Speicherzustand
- Betreiber koennen anhand dieser Daten:
  - fehlerhafte Plugins identifizieren
  - Konfigurationen anpassen
  - Probleme an Community oder Hersteller melden

**Telemetrie (optional)**

- In bestimmten Editionen:
  - Uebermittlung anonymisierter Nutzungs- und Crashdaten
  - Ziel: Stabilitaets- und Balancingverbesserung
- Transparente Kommunikation gegenueber Nutzenden ist hierbei wichtig.

**Zielbild**

- Fehler sollen:
  - frueh erkannt
  - eindeutig dokumentiert
  - reproduzierbar gemacht werden koennen
- Logging belastet Performance nicht unverhaeltnismaessig und ist konfigurierbar.

### 8.4 Internationalisierung

Minecraft wird weltweit genutzt und muss sprachlich flexibel sein, ohne dass Kernlogik angepasst wird.

**Grundprinzipien**

- Trennung von **Textressourcen** und **Anwendungslogik**
- Nutzung von Sprachdateien pro Sprache/Locale (z. B. `en_us`, `de_ch`, `fr_fr`)

**Mechanismen**

- UI-Texte, Item-Namen, Fehlermeldungen, Tooltips:
  - werden aus sprachspezifischen Ressourcendateien geladen
- Community- und Custom-Resource-Packs:
  - ermoeglichen eigene Sprachen/Begriffe
- Keine Logik in Textdateien:
  - sprachliche Anpassung aendert nie das Verhalten der Spielmechanik

**Ziele**

- Einfacher Wechsel der Sprache im Client
- Konsistenz der Begriffe pro Locale
- Unterstuetzung auch fuer Custom-Server und Modpacks

### 8.5 Performanz- und Skalierungskonzept

Die Performance von Minecraft haengt stark von Weltgroesse, Anzahl Entitaeten, Redstone-Mechaniken und Netzwerkbelastung ab. Das Konzept zielt darauf ab, trotz hoher Freiheitsgrade einen stabilen Betrieb zu ermoeglichen.

**Wesentliche Stellhebel**

1. **Chunk-Streaming**
   - Welt wird in Chunks geladen/gespeichert, nur relevante Bereiche im Speicher
   - Clients erhalten nur Chunks im Sichtbereich
2. **Render- und Simulationsdistanzen**
   - Serverseitige Simulation Distance:
     - definiert, wie weit um Spielende herum aktiv simuliert wird (Mobs, Redstone)
   - Clientseitige Render Distance:
     - beeinflusst, wie weit die Welt visuell dargestellt wird
3. **Tick-basiertes Modell**
   - Standard: 20 Ticks/Sekunde
   - Pro Tick:
     - begrenzte Anzahl Operationen (z. B. Entitaetsupdates, Blockticks)
   - Ueberlastung fuehrt zu niedrigeren effektiven Ticks (Lag), daher:
     - Begrenzung von Entitaeten, Farmen, Redstone-Uhren etc. empfohlen
4. **Optimierte Datenstrukturen**
   - Spezialisierte Strukturen fuer Block- und Entitaetsverwaltung
   - Caching von haeufig genutzten Daten (z. B. Lichtberechnung, Nachbarschaftsinformationen)

**Skalierungsstrategien**

- Vertikale Skalierung:
  - Leistungsstarke Hardware (CPU-Takt, Single-Core-Performance, schneller Speicher)
- Horizontale Skalierung (indirekt):
  - Mehrere Serverinstanzen (Lobby + Subserver, Bungeecord/Proxy-Architekturen)
  - Aufteilung nach Welten/Spielmodi
- Konfiguration:
  - Begrenzung von Spielern pro Instanz
  - Optimierung von Plugins und Konfigurationen

**Zielbild**

- Stabiler Betrieb auch bei vielen Spielenden und komplexen Welten
- Vorhersehbares Verhalten bei zunehmender Last
- Klar dokumentierte Optionen fuer Betreiber zur Optimierung

## 9. Architekturentscheidungen

Dieses Kapitel dokumentiert ausgewaehlte, architekturpraegende Entscheidungen fuer das (didaktische) Minecraft-Architekturmodell. Die hier beschriebenen Entscheidungen haben unmittelbare Auswirkungen auf Struktur, Qualitaet, Erweiterbarkeit und Betriebsmodell des Systems. Sie sind bewusst vereinfacht, orientieren sich aber an realistischen Anforderungen und beobachtbarem Verhalten.

Jede Entscheidung wird kurz erlaeutert mit:
- Kontext / Problemstellung
- Entscheidung
- Begruendung (Pro)
- Nebenwirkungen bzw. Risiken (Contra)
- Konsequenzen fuer Betrieb und Weiterentwicklung

### 9.1 Entscheidung: Client-Server statt Peer-to-Peer

**Kontext**  
Minecraft soll sowohl Einzelspieler- als auch Mehrspieler-Szenarien mit persistenter, konsistenter Welt unterstuetzen. Gleichzeitig besteht ein hohes Risiko durch modifizierte Clients, Cheating und instabile Netzwerke. Es wird ein klares Verantwortungsmodell fuer die Spiellogik benoetigt.

**Entscheidung**  
Das System verwendet konsequent ein **Client-Server-Modell** mit **serverseitiger Autoritaet**.  
Im Singleplayer wird technisch ein lokaler Server im Hintergrund betrieben (Java Edition), im Multiplayer dedizierte oder gehostete Server.

**Pro**

- Klare Instanz, die als „Single Source of Truth“ fungiert
- Schutz vor Manipulation der Spielwelt durch unzuverlaessige Clients
- Einheitliche Grundlage fuer Anti-Cheat, Logging und Regelpruefung
- Trennung von Darstellung (Client) und Logik (Server) ermoeglicht einfachere Wartung

**Contra / Risiken**

- Hoeherer Implementierungsaufwand als bei rein lokalen Singleplayer-Architekturen
- Zusaetzliche Latenz im Multiplayer
- Mehr Verantwortung und Komplexitaet im Serverbetrieb

**Konsequenzen**

- Alle kritischen Pruefungen muessen serverseitig erfolgen
- Erweiterungen muessen das Autoritaetsprinzip respektieren (keine sicherheitsrelevante Logik ausschliesslich im Client)
- Monitoring, Security und Performance-Tuning konzentrieren sich stark auf den Server

### 9.2 Entscheidung: Chunk-basierte Weltorganisation

**Kontext**  
Die Spielwelt ist potentiell (theoretisch) unendlich gross. Vollstaendige In-Memory-Verwaltung ist unmoeglich, Laden/Speichern muss effizient und skalierbar sein.

**Entscheidung**  
Die Welt wird in **Chunks** (feste Raumsegmente) und ggf. Regionen eingeteilt. Datenverwaltung, Netzwerktransfer und Rendering orientieren sich an diesen Einheiten.

**Pro**

- Effizientes Laden, Speichern und Streamen von Teilbereichen der Welt
- Begrenzung von Speicherbedarf und I/O
- Gute Grundlage fuer Sichtweitensteuerung und Performanzoptimierung
- Vereinfachte Parallelisierung bestimmter Operationen (z. B. Laden/Entladen)

**Contra / Risiken**

- Komplexere Logik an Chunk-Grenzen (Licht, Redstone, Physik)
- Potenzielle Artefakte bei schlecht abgestimmten Lade-/Entlade-Strategien
- Hoeherer Implementierungs- und Testaufwand

**Konsequenzen**

- Alle weltbezogenen Mechanismen (Mobs, Physik, Redstone, Strukturen) muessen Chunk-Grenzen korrekt behandeln
- Netzwerkprotokoll und Persistenz sind direkt an Chunks gekoppelt
- Performance-Optimierungen (z. B. Simulation Distance) greifen auf Chunk-Ebene

### 9.3 Entscheidung: Binaere Speicherformate (z. B. NBT)

**Kontext**  
Weltdaten, Entitaeten, Inventare und Konfigurationen sind komplex, hierarchisch und zahlreich. Textformate waeren zu gross und zu langsam, rein proprietaere, undokumentierte Formate erschweren Tools und Weiterentwicklung.

**Entscheidung**  
Verwendung eines **binaeren, strukturierten Formats** (z. B. NBT – Named Binary Tag) fuer zentrale Persistenzdaten.

**Pro**

- Kompakte Speicherung und schnelle Ein-/Ausgabe
- Hierarchische Struktur ermoeglicht flexible, erweiterbare Datenmodelle
- Gut geeignet fuer Tools, Editoren und Server-Utilities
- Bessere Performance als reine Textformate bei grossen Welten

**Contra / Risiken**

- Hoehere Einstiegshuerde fuer Entwickler ohne passende Tools
- Fehlerhafte Implementierungen koennen Welten korrupt machen
- Formatversionierung und Rueckwaertskompatibilitaet muessen bewusst gestaltet werden

**Konsequenzen**

- Notwendigkeit klar definierter Serialisierungs-/Deserialisierungslogik
- Dokumentation und Tools (Viewer, Converter, Backup) werden wichtig
- Aenderungen am Format erfordern Migrationsstrategien

### 9.4 Entscheidung: Erweiterbarkeit ueber definierte Schnittstellen

**Kontext**  
Die Community erwartet Modding, Custom-Server, Minigames, Erweiterungen. Gleichzeitg sollen Kernfunktionen stabil bleiben. „Hard Forks“ oder inoffizielle Hacks sind langfristig schwer wartbar.

**Entscheidung**  
Bereitstellung **offizieller, dokumentierter Erweiterungspunkte** (APIs, Events, Datapacks, Resource Packs, Add-ons), statt ausschliesslich inoffizieller Anpassungen am Kern.

**Pro**

- Staerkere und nachhaltige Community-Ökosysteme
- Schnellere Entwicklung neuer Inhalte ohne Kernanpassung
- Klarere Trennung zwischen Plattform (Minecraft) und Erweiterungen (Community, Serverbetreiber)
- Erhoehte Langlebigkeit des Produkts

**Contra / Risiken**

- Wartungsaufwand fuer stabile APIs und Rueckwaertskompatibilitaet
- Erwartungshaltung der Community an langfristige Stabilitaet
- Potenzielle Sicherheits- und Performanceprobleme durch schlecht implementierte Erweiterungen

**Konsequenzen**

- Erweiterungs-APIs werden selbst zu kritischen Produktbestandteilen
- Release-Management muss Auswirkungen auf Mods/Plugins beruecksichtigen
- Serverbetreiber tragen Verantwortung bei der Auswahl vertrauenswuerdiger Erweiterungen

### 9.5 Entscheidung: Separate Produktlinien (Java / Bedrock)

**Kontext**  
Unterschiedliche Zielplattformen (PC, Konsole, Mobile) stellen abweichende Anforderungen an Performance, Speicher, Engine, Integrationen. Eine einzige Codebasis waere schwer optimal anzupassen.

**Entscheidung**  
Fuehrung von **mindestens zwei Hauptlinien**:
- Java Edition (offene PC-Plattform, starke Modding-Community)
- Bedrock Edition (C++-basiert, optimiert fuer Konsole/Mobile/Cross-Platform)

**Pro**

- Optimale Ausnutzung plattformspezifischer Features und Ressourcen
- Bessere Performance und Benutzererfahrung auf schwächeren Geraeten
- Flexiblere Produktstrategien (z. B. Education, Crossplay)

**Contra / Risiken**

- Fragmentierung der Community (Java vs. Bedrock)
- Unterschiedliche Modding-Oekosysteme und Tools
- Hoeherer Entwicklungs- und Wartungsaufwand (Features muessen doppelt umgesetzt / abgestimmt werden)

**Konsequenzen**

- Architekturkonzepte muessen zwischen Editionen abgestimmt, aber nicht identisch sein
- Kommunikation gegenueber Nutzenden ist wichtig (Kompatibilitaeten, Unterschiede)
- Gemeinsame Grundprinzipien (z. B. Block-/Chunk-Modell, Client-Server, Sicherheitsprinzipien) dienen als verbindende Klammer

## 10. Qualitaetsanforderungen

Die Architektur von Minecraft ist stark durch nicht-funktionale Anforderungen gepraegt. Dieses Kapitel konkretisiert die wichtigsten Qualitaetsziele und uebersetzt sie in pruefbare Szenarien. Die beschriebenen Anforderungen gelten insbesondere fuer Mehrspielerszenarien, grosse Welten, Community-Server und einen langfristigen Lebenszyklus des Produkts.

### 10.1 Qualitaetsbaum (Ueberblick)

Die zentralen Qualitaetsmerkmale lassen sich wie folgt gliedern:

- **Funktionalitaet**
  - Korrekte Umsetzung der Spielregeln (Physik, Kampf, Crafting, Redstone, Mobs)
  - Konsistente Spielwelt auch bei vielen parallelen Aktionen
- **Zuverlaessigkeit**
  - Robustheit bei hoher Last
  - Datenintegritaet von Welten und Spielerfortschritten
  - Fehlertoleranz bei Abstuerzen und Netzwerkproblemen
- **Benutzbarkeit**
  - Intuitive Steuerung und klare Rueckmeldungen
  - Konsistente UI ueber Editionen (im Rahmen der Plattformkonzepte)
  - Verstaendliche Einstellungen und Optionen
- **Effizienz**
  - Ausreichende FPS auf Zielplattformen
  - Kontrollierte Latenzen in Mehrspielerszenarien
  - Begrenzter Speicher- und Bandbreitenverbrauch
- **Aenderbarkeit und Erweiterbarkeit**
  - Einfuehrung neuer Inhalte ohne Kernarchitektur zu brechen
  - Unterstuetzung von Mods, Plugins, Datapacks und Add-ons
  - Konfigurierbarkeit statt Hardcoding
- **Portabilitaet**
  - Lauffaehigkeit auf verschiedenen Betriebssystemen und Geraeteklassen
  - Anpassbarkeit an unterschiedliche Eingabegeraete und Grafik-APIs
- **Sicherheit**
  - Schutz vor Manipulation der Spielwelt
  - Sichere Authentifizierung und Rechteverwaltung
- **Wartbarkeit & Transparenz**
  - Nachvollziehbare Logs, Crash-Reports und Konfiguration
  - Verstaendliche Fehlermeldungen fuer Betreiber

Diese Merkmale sind teilweise konkurrierend (z. B. Performance vs. Sicherheit, Offenheit vs. Stabilitaet) und muessen durch Architekturentscheidungen balanciert werden.

### 10.2 Beispielhafte Qualitaetsszenarien

Die folgenden Szenarien konkretisieren die Qualitaetsanforderungen in pruef- und diskutierbarer Form. Jedes Szenario enthaelt Ausloeser, Kontext, erwartetes Systemverhalten und Erfolgsindikatoren.

#### 10.2.1 Performance – Dedizierter Mehrspielerserver

**Ausgangslage**  
Ein dedizierter Java- oder Bedrock-Server hostet:
- 60+ aktive Spielende
- mehrere Farmen, Redstone-Mechanismen und Mobs
- eine Welt mit vielen bereits erkundeten Chunks

**Anforderung**

- Der Server haelt eine stabile Tick-Rate in einem akzeptablen Bereich.
- Die durchschnittliche Latenz fuer Spielende bleibt spielbar (z. B. kein dauerhafter Rubberbanding).

**Massnahmen / Architekturbezug**

- Chunk-basiertes Laden und Entladen
- Begrenzung von Simulation Distance und Entities
- Optimierte Datenstrukturen fuer Block- und Entitaetsupdates
- Moeglichkeit fuer Betreiber, Performance-Parameter anzupassen

#### 10.2.2 Zuverlaessigkeit – Absturz während des Speicherns

**Ausgangslage**  
Während eines regulären Betriebs laeuft ein automatischer Save-Prozess. Waehrend dieses Vorgangs faellt der Serverprozess (z. B. durch Hardwarefehler) unerwartet aus.

**Erwartetes Verhalten**

- Beim Neustart:
  - Welt ist konsistent ladbar.
  - Maximal ein kleiner Zeitraum (z. B. letzte Sekunden) geht verloren.
- Es entstehen keine grossflaechig korrupten Welten.

**Massnahmen / Architekturbezug**

- Transaktionsaehnliche Schreibstrategien
- Atomare Updates von Chunk-Dateien
- Saubere Trennung von Schreib-/Lesephasen
- Crash-Reports zur Analyse

#### 10.2.3 Erweiterbarkeit – Neues Lobby-Plugin

**Ausgangslage**  
Ein Serverbetreiber installiert ein neues Lobby-/Hub-Plugin (Java Dedicated Server) mit Teleport-, Scoreboard- und Inventar-Funktionen.

**Erwartetes Verhalten**

- Plugin nutzt die vorgesehenen APIs und Events.
- Kein Eingriff in Kerncode erforderlich.
- Bei Fehlern im Plugin:
  - Server bleibt grundsaetzlich lauffaehig oder laesst sich kontrolliert neu starten.
- Updates des Spiels sind moeglich, ohne das komplette System-Setup zu brechen (abhaengig von API-Stabilitaet).

**Massnahmen / Architekturbezug**

- Klar dokumentierte, stabile Erweiterungsschnittstellen
- Trennung von Kern- und Plugin-Code
- Fehlerisolation soweit moeglich

#### 10.2.4 Sicherheit – Manipulierter Client

**Ausgangslage**  
Ein modifizierter Client versucht:
- ungueltige Positionsdaten zu senden
- Items ohne legitime Quelle zu erzeugen
- durch Wände oder zu schnell zu laufen

**Erwartetes Verhalten**

- Serverseitige Validierung erkennt ungueltige Aktionen.
- Entsprechende Pakete werden verworfen.
- Optional:
  - Protokollierung des Vorfalls
  - automatische Massnahmen (Kick, Ban)
- Kein Schaden an Welt- oder Serverintegritaet.

**Massnahmen / Architekturbezug**

- Autoritativer Server
- Validierungslogik fuer Bewegung, Aktionen, Inventar
- Rate-Limiting und Konsistenzchecks

#### 10.2.5 Benutzbarkeit – Neue Spielende

**Ausgangslage**  
Neue Spielende starten Minecraft zum ersten Mal auf einer Standardplattform.

**Erwartetes Verhalten**

- Grundsteuerung schnell erlernbar
- Klare Rueckmeldungen bei typischen Aktionen (Blockabbau, Crafting, Schaden)
- Verstaendliche Menues und Optionen
- Kein Zwang, externe Dokumentation lesen zu muessen, um die Basisfunktionen zu verstehen

**Massnahmen / Architekturbezug**

- Konsistentes UI-Konzept
- Konfigurierbare, aber sinnvolle Default-Einstellungen
- Kontextbezogene Tooltips und klare Symbole

#### 10.2.6 Portabilitaet – Verschiedene Plattformen

**Ausgangslage**  
Minecraft laeuft auf Windows-PC, Konsole und Mobile Device.

**Erwartetes Verhalten**

- Kernspielmechaniken bleiben ueberall vergleichbar.
- Steuerung ist an jeweilige Plattform angepasst (Controller, Touch, Maus/Tastatur).
- Performance-Anforderungen sind abgestimmt auf Geraeteklassen.

**Massnahmen / Architekturbezug**

- Abstraktionsschichten fuer Rendering und Input
- Separate Produktlinien mit gemeinsamen Kernkonzepten
- Ressourcen- und UI-Anpassungen je Plattform

Diese Qualitaetsszenarien dienen als Referenz fuer Entwurfsentscheidungen, Implementierung, Tests und Betrieb. Sie machen transparent, welche nicht-funktionalen Anforderungen fuer Minecraft (im Sinne dieses didaktischen Architekturmodells) als kritisch betrachtet werden und wie die Architektur deren Erreichung unterstuetzt.

## 11. Risiken und Technische Schulden

Dieses Kapitel beschreibt zentrale Risiken und technische Schulden im Kontext des (didaktischen) Minecraft-Architekturmodells. Viele dieser Punkte ergeben sich direkt aus den bewussten Architekturentscheidungen (Kapitel 9) und den Qualitaetsanforderungen (Kapitel 10). Sie sind nicht zwingend Fehler, sondern Konsequenzen von Priorisierungen (z. B. Offenheit vs. Sicherheit, Vielfalt vs. Einfachheit), die langfristig gesteuert werden muessen.

### 11.1 Fragmentierung der Produkt- und Oekosystem-Landschaft

**Beschreibung**

Minecraft existiert in mehreren Editionen (Java, Bedrock, Education), mit unterschiedlichen Engines, APIs und Distributionskanaelen. Zusaetzlich gibt es verschiedene Modding-Communities, Server-Software-Varianten und Hosting-Modelle.

**Risiken**

- Unterschiedliche Features, Mechaniken und technische Grenzen fuehren zu Verwirrung bei Spielenden.
- Inhalte, Mods und Anleitungen sind nicht ohne Weiteres zwischen Editionen kompatibel.
- Hoeherer Entwicklungs- und Testaufwand, weil Aenderungen editionsspezifisch umgesetzt werden muessen.

**Technische Schulden**

- Duplizierte oder divergierende Implementierungen von Kernkonzepten
- Aufwand fuer langfristige Synchronisation (z. B. neue Blöcke, Biome, Mechaniken)

**Moegliche Gegenmassnahmen**

- Klar dokumentierte Unterschiede und Kompatibilitaetsmatrizen
- Gemeinsame Design-Guidelines fuer Kernmechaniken
- Standardisierung von Protokollen, Datenformaten und Konzepten, wo sinnvoll

### 11.2 Abhaengigkeit von Dritt-Plugins und Mods

**Beschreibung**

Insbesondere im Java-Server-Bereich stuetzt sich ein grosser Teil der Funktionalitaet (Minigames, Lobby-Systeme, Moderation, Anti-Cheat) auf Dritt-Plugins und Mods.

**Risiken**

- Unsichere oder schlecht programmierte Plugins koennen:
  - Sicherheitsluecken oeffnen
  - Weltdaten zerstoeren
  - Performance massiv beeintraechtigen
- Abhaengigkeit von freiwilligen Maintainerinnen und Maintainern:
  - Plugins werden bei neuen Versionen nicht mehr aktualisiert
  - Server werden gezwungen, auf alten Versionen zu bleiben oder eigene Forks zu pflegen

**Technische Schulden**

- Inoffizielle, aber faktisch kritische Komponenten ausserhalb der offiziellen Architekturkontrolle
- Zusaetzliche Komplexitaet bei Fehlersuche (Core vs. Plugin vs. Umgebung)

**Moegliche Gegenmassnahmen**

- Stabilere und klar definierte Erweiterungs-APIs
- Best Practices und Guidelines fuer Plugin-Entwicklung
- Bessere Isolierung und Sandbox-Mechanismen fuer Erweiterungen

### 11.3 Rueckwaertskompatibilitaet und Langzeitwelten

**Beschreibung**

Viele Spielende und Server betreiben Welten ueber Jahre hinweg. Neue Versionen von Minecraft bringen:

- neue Blöcke, Biome, Strukturen
- geaenderte Mechaniken
- Performance-Optimierungen und Protokollaenderungen

**Risiken**

- Weltmigrationen koennen fehleranfaellig sein (z. B. Chunk-Konvertierung, neue Generation angrenzend zu alter)
- Aeltere Welten werden technisch belastend (Datenmenge, alte Strukturen, exotische Kantenfaelle)
- Nicht alle Features sind rueckwaertskompatibel abbildbar

**Technische Schulden**

- Legacy-Code zur Unterstuetzung alter Welten und Datenformate
- Spezielle Sonderfaelle in Welt-Generatoren und Laderoutinen

**Moegliche Gegenmassnahmen**

- Klare Migrationspfade (Tools, Dokumentation)
- Versionierte Datenformate mit expliziten Upgrade-Strategien
- Optionalität: Betreiber koennen bewusst auf bestimmten Versionen bleiben

### 11.4 Performance-Grenzen in Extrem-Szenarien

**Beschreibung**

Minecraft erlaubt hohe Freiheit: unendliche Welten, komplexe Redstone-Schaltungen, gigantische Farmen, viele Entitaeten. Das fuehrt in Extremfaellen zu Lastsituationen, die urspruengliche Designannahmen sprengen.

**Risiken**

- Massive Lags bei:
  - sehr grossen Redstone-Maschinen
  - extremen Mob-Farmen
  - vielen gleichzeitigen Spielenden in einem Gebiet
- Unklare Erwartungen von Spielenden: „Wenn es geht, muss es auch performant laufen.“

**Technische Schulden**

- Historische Mechaniken (z. B. Redstone) wurden nicht ausschliesslich unter Skalierungsgesichtspunkten entwickelt
- Komplexe Optimierungslogik in Tick- und Update-Systemen, die schwer wartbar ist

**Moegliche Gegenmassnahmen**

- Serverseitige Limits und Konfiguration (Simulation Distance, Entity-Despawn, Hopper-Optimierungen)
- Bessere Transparenz fuer Betreiber (Profiling, Warnungen)
- Zielgerichtete Optimierung kritischer Mechaniken

### 11.5 Abhaengigkeit von proprietaeren Plattform- und Cloud-Diensten

**Beschreibung**

Minecraft nutzt zentrale Dienste von Microsoft/Mojang:

- Kontoverwaltung (Microsoft-/Xbox-Accounts)
- Realms-Hosting
- Offizieller Marketplace
- Teilweise Telemetrie und Lizenzpruefung

**Risiken**

- Technische und organisatorische Abhaengigkeit von diesen Diensten
- Stoerungen oder Aenderungen auf Plattformseite wirken direkt auf das Spielerlebnis
- Einschraenkungen fuer rein selbstbestimmte Offline-/On-Prem-Setups im offiziellen Online-Modus

**Technische Schulden**

- Codepfade, die stark an bestimmte Cloud- oder Plattformdienste gekoppelt sind
- Erschwerte Umsetzbarkeit alternativer Identitaets- oder Lizenzmodelle

**Moegliche Gegenmassnahmen**

- Saubere Trennung von Lokallogik und Cloud-Integrationen
- Fallback-Szenarien fuer eingeschraenkte Konnektivitaet (z. B. Offline- oder LAN-Modi)
- Dokumentierte Schnittstellen mit klaren Vertrags- und Betriebsregelungen

### 11.6 Zusammenfassung

Die genannten Risiken und technischen Schulden sind zum grossen Teil direkte Konsequenz gewollter Eigenschaften von Minecraft:

- Offenheit fuer Community und Mods
- Extrem hohe Freiheitsgrade in der Spielwelt
- Multi-Edition- und Multi-Plattform-Strategie
- Langfristige Unterstuetzung bestehender Welten

Die Architektur muss daher nicht nur Funktionen bereitstellen, sondern auch Mechanismen, um mit diesen Spannungsfeldern kontrolliert umzugehen: durch klare Schnittstellen, Konfiguration, Dokumentation, Monitoring, Tooling und bewusste Produktentscheidungen.

## 12. Glossar

Dieses Glossar definiert zentrale Begriffe des (didaktischen) Minecraft-Architekturmodells. Ziel ist eine eindeutige, technisch saubere und gleichzeitig verstaendliche Verwendung der Fachbegriffe in der Dokumentation.

### 12.1 Kernbegriffe der Spielwelt

**Block**  
Kleinste statische Baueinheit der Minecraft-Welt.  
Bestimmt durch Position, Typ und metadatenspezifische Eigenschaften (z. B. Ausrichtung, Zustand). Bloecke bilden Landschaft, Gebaeude, Redstone-Schaltungen etc. und sind Grundlage saemtlicher baulicher Interaktion.

**Chunk**  
Fester Weltabschnitt und zentrale Verwaltungs- und Speichereinheit.  
Typischerweise 16x16 Bloecke in der Flaeche mit definierter Hoehe. Chunks werden einzeln geladen, gespeichert, ueber das Netzwerk uebertragen und fuer Simulation (Mobs, Redstone) verwendet. Dient der Skalierung und Performanceoptimierung.

**Region**  
Gruppierung mehrerer Chunks in groessere Dateieinheiten (je nach Implementierung).  
Reduziert Fragmentierung im Dateisystem und verbessert I/O-Verhalten beim Laden und Speichern vieler benachbarter Chunks.

**Biome**  
Klimatisch und gestalterisch abgegrenzte Bereiche der Welt (z. B. Wald, Wueste, Ozean).  
Beeinflussen Vegetation, Mobs, Wettereffekte und zum Teil Spielmechaniken. Werden in der Weltgenerierung pro Chunk oder Subbereich festgelegt.

### 12.2 Daten- und Architekturbegriffe

**NBT (Named Binary Tag)**  
Binaeres, strukturiertes Datenformat fuer Minecraft.  
Wird zur Speicherung von Weltdaten, Entitaeten, Spielerinformationen und Konfigurationen verwendet. Unterstuetzt verschachtelte Strukturen und ist auf effiziente Speicherung und Verarbeitung ausgelegt.

**Entity**  
Dynamisches Objekt in der Welt.  
Beispiele: Spieler, Mobs, Tiere, Boote, Minecarts, Projektile, fallende Bloecke, droppte Items. Im Gegensatz zu Bloecken besitzen Entities typischerweise Position, Bewegung und eigenes Verhalten.

**Tile Entity / Block Entity**  
Spezialfall eines Blocks mit zusaetzlichen Daten und Logik.  
Beispiele: Truhen, Schilder, Ofen, Beacons. Werden genutzt, wenn ein Block zusaetzliche Zustandsinformationen oder komplexeres Verhalten benoetigt.

**Tick**  
Diskrete Zeit- und Verarbeitungs-Einheit der Spiel- und Serverlogik.  
Standard: 20 Ticks pro Sekunde. Pro Tick werden u. a. Bewegungen, Physik, KI, Redstone, Effekte und bestimmte Speicheroperationen ausgefuehrt. Tick-Rate ist zentraler Indikator fuer Serverperformance.

**Dedicated Server**  
Eigenstaendiger Serverprozess ohne grafische Oberflaeche.  
Dient ausschliesslich dem Hosten von Minecraft-Welten fuer Mehrspielende. Wird lokal, auf Root-Servern, VPS oder in der Cloud betrieben und ist meist konfigurier- und erweiterbar.

### 12.3 Erweiterungen und Inhalte

**Mod (Modification)**  
Erweiterung oder Anpassung des Spiels, typischerweise auf Codebasis (vor allem Java Edition).  
Kann neue Bloecke, Items, Mobs, Dimensionen, Mechaniken oder UI-Elemente einfuehren. Erfordert meist kompatible Mod-Loader (z. B. Forge, Fabric).

**Plugin**  
Serverseitige Erweiterung auf Basis definierter APIs von Server-Software (z. B. Bukkit, Spigot, Paper).  
Fokus haeufig auf Gameplay-Logik, Moderation, Commands, Lobbys, Minigames. Aendert in der Regel den Client nicht direkt, sondern wirkt ueber Serverregeln und -funktionen.

**Resource Pack**  
Paket zur Anpassung der Praesentationsebene.  
Beeinflusst Texturen, Sounds, Schriftarten, Modelle und bestimmte UI-Elemente. Aendert grundsaetzlich keine Kernlogik der Spielmechanik.

**Datapack**  
Datengetriebene Erweiterung (insbesondere Java Edition).  
Ermoeglicht Konfiguration von Rezepten, Loot-Tabellen, Funktionen (Function Files), Tags, Fortschritten und teilweise Strukturen – ohne direkten Code-Eingriff. Ideal fuer serverseitige Anpassungen, die mit Vanilla-Mechaniken kompatibel bleiben.

**Add-on / Behavior Pack (Bedrock)**  
Offizielle Erweiterungsmechanismen fuer Bedrock Edition.  
Erlauben Anpassungen von Verhalten, Eigenschaften und Inhalten (z. B. Mobs, Items, Regeln) auf Basis definierter JSON- und Skriptstrukturen.

### 12.4 Online-Dienste und Infrastruktur

**Realms**  
Offizieller, verwalteter Minecraft-Serverdienst von Mojang/Microsoft.  
Ermoeglicht einfaches Hosten kleiner Welten mit minimalem Administrationsaufwand. Technische Details des Backends liegen ausserhalb der direkten Kontrolle der Spielenden.

**Microsoft-/Xbox-Account**  
Zentrale Online-Identitaet fuer Minecraft (insbesondere Bedrock und moderner Java-Login).  
Wird fuer Authentifizierung, Multiplayer-Funktionen, Marketplace und teilweise Plattform-uebergreifende Profile genutzt.

### 12.5 Architektur- und Betriebsbegriffe

**Client**  
Die auf dem Endgeraet laufende Anwendung zur Darstellung der Spielwelt, Entgegennahme von Eingaben und Kommunikation mit einem Server (integriert oder dediziert).  
Enthaelt keine vertrauenswuerdige Spiellogik fuer sicherheitsrelevante Entscheidungen.

**Server**  
Autoritative Instanz fuer Spiellogik, Weltzustand und Validierung von Aktionen.  
Kann lokal (Singleplayer-Server), im LAN, dediziert oder als Managed Service betrieben werden.

**Client-Server-Protokoll**  
Spezifisches Netzwerkprotokoll zwischen Minecraft-Client und -Server.  
Ist paketbasiert, versionsabhaengig und umfasst u. a. Login, Welt-Updates, Entitaetsdaten, Chat und Interaktionen.

**Whitelist / Banliste / Permissions**  
Konfigurationsmechanismen zur Zugangs- und Rechteverwaltung.  
Steuern, wer auf einem Server spielen darf und welche Befehle oder Aktionen erlaubt sind.
