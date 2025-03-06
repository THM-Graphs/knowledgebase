
https://modelcontextprotocol.io/quickstart/user

https://www.youtube.com/watch?v=wxCCzo9dGj0

# Claudes Model Context Protocol (MCP): Ein Leitfaden für Entwickler

![mm](https://www.unite.ai/wp-content/uploads/2023/07/Aayush_mittal-150x150.jpg)

Aktualisiert on 10. Dezember 2024

By

[Aayush Mittal](https://www.unite.ai/de/Autor/aayushmittal/ "Beiträge von Aayush Mittal")

![](https://www.unite.ai/wp-content/uploads/2024/12/Anthropic-Model-Context-Protocol.webp)

Anthropologie [Model Context Protocol (MCP)](https://modelcontextprotocol.io/introduction) ist ein Open-Source-Protokoll, das eine sichere, bidirektionale Kommunikation zwischen KI-Assistenten und Datenquellen wie Datenbanken, APIs und Unternehmenstools ermöglicht. Durch die Einführung einer Client-Server-Architektur standardisiert MCP die Art und Weise, wie KI-Modelle mit externen Daten interagieren, sodass keine benutzerdefinierten Integrationen für jede neue Datenquelle erforderlich sind.

### Schlüsselkomponenten von MCP:

- **Gastgeber**: KI-Anwendungen, die Verbindungen initiieren (z. B. Claude Desktop).
- **Clients**: Systeme, die Eins-zu-Eins-Verbindungen mit Servern innerhalb der Hostanwendung aufrechterhalten.
- **Fertige Server**: Systeme, die Kunden Kontext, Tools und Eingabeaufforderungen bereitstellen.

## Warum ist MCP wichtig?

### Vereinfacht Integrationen

Bisher waren für die Verbindung von KI-Modellen mit unterschiedlichen Datenquellen benutzerdefinierte Codes und Lösungen erforderlich. MCP ersetzt diesen fragmentierten Ansatz durch ein einziges, standardisiertes Protokoll. Diese Vereinfachung beschleunigt die Entwicklung und reduziert den Wartungsaufwand.

### Verbessert die KI-Fähigkeiten

Indem MCP KI-Modellen nahtlosen Zugriff auf verschiedene Datenquellen bietet, verbessert es deren Fähigkeit, relevantere und genauere Antworten zu liefern. Dies ist insbesondere bei Aufgaben von Vorteil, die Echtzeitdaten oder spezielle Informationen erfordern.

### Fördert die Sicherheit

MCP wurde mit Blick auf die Sicherheit entwickelt. Server kontrollieren ihre eigenen Ressourcen, sodass es nicht mehr nötig ist, sensible API-Schlüssel mit KI-Anbietern zu teilen. Das Protokoll legt klare Systemgrenzen fest und stellt sicher, dass der Datenzugriff sowohl kontrolliert als auch überprüfbar ist.

### Zusammenarbeit

Als Open-Source-Initiative fördert MCP Beiträge aus der Entwickler-Community. Diese kollaborative Umgebung beschleunigt Innovationen und erweitert die Palette verfügbarer Konnektoren und Tools.

## So funktioniert MCP

### Architektur

[![MCP-Architektur](https://www.unite.ai/wp-content/uploads/2024/12/960x0.png)](https://www.unite.ai/wp-content/uploads/2024/12/960x0.png)

MCP-Architektur

Im Kern folgt MCP einer Client-Server-Architektur, bei der eine Host-Anwendung eine Verbindung zu mehreren Servern herstellen kann. Dieses Setup ermöglicht es KI-Anwendungen, nahtlos mit verschiedenen Datenquellen zu interagieren.

#### Komponenten:

- **MCP-Hosts**: Programme wie Claude Desktop, IDEs oder KI-Tools, die über MCP auf Ressourcen zugreifen möchten.
- **MCP-Kunden**: Protokollclients, die Eins-zu-Eins-Verbindungen mit Servern aufrechterhalten.
- **MCP-Server**: Leichtgewichtige Programme, die jeweils spezifische Fähigkeiten über das standardisierte Model Context Protocol bereitstellen.
- **Lokale Ressourcen**: Die Ressourcen Ihres Computers (Datenbanken, Dateien, Dienste), auf die MCP-Server sicher zugreifen können.
- **Remote-Ressourcen**: Über das Internet verfügbare Ressourcen (z. B. über APIs), mit denen MCP-Server eine Verbindung herstellen können.

## Erste Schritte mit MCP

### Voraussetzungen:

- **Claude Desktop App**: Verfügbar für macOS und Windows.
- **SDKs**: MCP bietet SDKs für TypeScript und [Python](https://pypi.org/project/mcp/).
- nano ~/Library/Application\ Support/Claude/claude_desktop_config.json

### Schritte zum Beginnen

1. **Installieren Sie vorgefertigte MCP-Server**: Beginnen Sie mit der Installation von Servern für gängige Datenquellen wie Google Drive, Slack oder GitHub über [Claude Desktop-App](https://support.anthropic.com/en/articles/10065433-installing-claude-for-desktop).
2. **Konfigurieren der Hostanwendung**: Bearbeiten Sie die Konfigurationsdatei, um die MCP-Server einzuschließen, die Sie verwenden möchten.
    
    `{   "mcpServers": {   "sqlite": {   "command": "uvx",   "args": ["mcp-server-sqlite", "--db-path", "/path/to/your/database.db"] }}}   `
    
3. **Erstellen Sie benutzerdefinierte MCP-Server**: Verwenden Sie die bereitgestellten SDKs, um Server zu erstellen, die auf Ihre spezifischen Datenquellen oder Tools zugeschnitten sind.
4. **Verbinden und testen**: Stellen Sie eine Verbindung zwischen Ihrer KI-Anwendung und dem MCP-Server her und beginnen Sie mit dem Experimentieren.

### Was passiert unter der Haube?

Wenn Sie über MCP mit einer KI-Anwendung wie Claude Desktop interagieren, laufen mehrere Prozesse ab, die die Kommunikation und den Datenaustausch erleichtern.

#### 1. Servererkennung

- **Initialisierung**: Beim Start stellt der MCP-Host (z. B. Claude Desktop) eine Verbindung zu Ihren konfigurierten MCP-Servern her. Dadurch werden die ersten Kommunikationskanäle eingerichtet, die für weitere Interaktionen erforderlich sind.

#### 2. Protokoll-Handshake

- **Fähigkeitsverhandlung**: Die Hostanwendung und die MCP-Server führen einen Handshake durch, um Fähigkeiten auszuhandeln und ein gemeinsames Verständnis herzustellen.
- **Login**: Der Host identifiziert, welcher MCP-Server eine bestimmte Anforderung basierend auf den von ihm bereitgestellten Ressourcen oder Funktionen verarbeiten kann.

#### 3. Interaktionsfluss

Betrachten wir ein Beispiel, bei dem Sie eine lokale SQLite-Datenbank über Claude Desktop abfragen.

[![MCP-Protokoll](https://www.unite.ai/wp-content/uploads/2024/12/download.svg)](https://www.unite.ai/de/wp-Inhalt/Uploads/2024/12/download.svg)

MCP-Protokoll

### Schritt-für-Schritt-Prozess:

1. **Verbindung initialisieren**: Claude Desktop stellt eine Verbindung zum MCP-Server her, der für die Interaktion mit SQLite konfiguriert ist.
2. **Verfügbare Funktionen**: Der MCP-Server kommuniziert seine Fähigkeiten, beispielsweise die Ausführung von SQL-Abfragen.
3. **Anfrageanfrage**: Sie fordern Claude Desktop auf, Daten abzurufen. Der Host sendet eine Abfrageanforderung an den MCP-Server.
4. **Ausführung von SQL-Abfragen**: Der MCP-Server führt die SQL-Abfrage auf der SQLite-Datenbank aus.
5. **Ergebnisabruf**: Der MCP-Server ruft die Ergebnisse ab und sendet sie an Claude Desktop zurück.
6. **Formatierte Ergebnisse**: Claude Desktop präsentiert Ihnen die Daten in einem lesbaren Format.

## Weitere Anwendungsfälle

- **Software-Entwicklung**: Verbessern Sie die Tools zur Codegenerierung, indem Sie KI-Modelle mit Code-Repositorys oder Issue-Trackern verbinden.
- **Datenanalyse**: Ermöglichen Sie KI-Assistenten den Zugriff auf und die Analyse von Datensätzen aus Datenbanken oder Cloud-Speichern.
- **Unternehmensautomatisierung**: Integrieren Sie KI in Business-Tools wie CRM-Systeme oder Projektmanagementplattformen.

## Vorteile der MCP-Architektur

- **Modularität**: Durch die Trennung von Host und Servern ermöglicht MCP eine modulare Entwicklung und einfachere Wartung.
- **Skalierbarkeit**: An einen einzigen Host können mehrere MCP-Server angeschlossen werden, die jeweils unterschiedliche Ressourcen verarbeiten.
- **Flexible Kommunikation**: Durch die Standardisierung der Kommunikation über MCP können verschiedene KI-Tools und -Ressourcen nahtlos zusammenarbeiten.

## Early Adopters und Community-Support

Unternehmen mögen [Replizieren](https://replit.com/) und [Codeium](https://codeium.com/) unterstützen bereits MCP, und Organisationen wie [Blockieren](https://block.xyz/) und [Apollo](https://www.apollographql.com/) haben es implementiert. Dieses wachsende Ökosystem deutet auf eine starke Unterstützung durch die Industrie und eine vielversprechende Zukunft für MCP hin.

## Ressourcen und weiterführende Literatur

- **Offizielle MCP-Dokumentation**: [Modellkontextprotokoll-Dokumente](https://modelcontextprotocol.io/)
- **GitHub-Repository**: [MCP-Server und SDKs](https://github.com/modelcontextprotocol)
- **Gemeinschaftsbeiträge**: [MCP-Server nach Community](https://github.com/modelcontextprotocol/servers)

## Schlussfolgerung

Das Model Context Protocol ist ein Fortschritt bei der Vereinfachung der Interaktion von KI-Modellen mit Datenquellen. Durch die Standardisierung dieser Verbindungen beschleunigt MCP nicht nur die Entwicklung, sondern verbessert auch die Fähigkeiten von KI-Assistenten. Anathopic leistet hervorragende Arbeit bei der Bereitstellung von Tools für Entwickler, mit denen sie KI effektiv nutzen können.