# n8n Confluence Custom Node - Installation & Deployment Guide (v0.1.6)

Dieses Dokument beschreibt die vollständigen Schritte zur Installation und Bereitstellung des Confluence Custom Nodes in einer OpenShift-Umgebung.

## Szenario: Änderungen am Custom Node bereitstellen

Wenn Sie Änderungen an der `package.json` (z.B. Version) oder am Quellcode vornehmen, folgen Sie diesen Schritten zur Bereitstellung.

## 🆕 Was ist neu in Version 0.1.6?

- **📝 Confluence Integration**: Vollständige Confluence API Unterstützung
- **🔐 Atlassian Credentials**: Sichere Authentifizierung mit API Token
- **📄 Page Operations**: CRUD Operationen für Confluence Pages
- **🔍 Search Functionality**: Durchsuchen von Confluence Spaces und Pages
- **🔧 Enhanced Error Handling**: Robuste Fehlerbehandlung für API-Calls

## Voraussetzungen

- OpenShift CLI (`oc`) installiert und angemeldet
- Node.js >= 18.10
- pnpm >= 9.1 (wichtig: nicht npm oder yarn)
- Zugriff auf das n8n Pod im OpenShift Namespace `mdb-test`

## Schritt-für-Schritt Anleitung

### 1. Lokale Entwicklung und Build

```bash
# Navigieren Sie zum Projektverzeichnis
cd /Users/vwijbaf/Softwareentwicklung/quellcode/n8n-nodes-confluence

# Abhängigkeiten installieren (falls noch nicht geschehen)
pnpm install

# Code formatieren und linten
pnpm format
pnpm lint

# Projekt builden
pnpm build
```

### 2. Paket erstellen

```bash
# npm-Paket erstellen (Version vorher in package.json anheben)
npm pack
```

Dies erstellt eine `.tgz` Datei, z.B. `bitovi-n8n-nodes-confluence-0.1.6.tgz`

### 3. OpenShift Login und Namespace

```bash
# Bei OpenShift anmelden
oc login --token=<YOUR_TOKEN> --server=https://api.dev-intranet-02-wob.ocp.vwgroup.com:6443

# Zum richtigen Namespace wechseln
oc project mdb-test

# Pod Namen ermitteln
oc get pods | grep n8n

# Beispiel Pod Name: n8n-xxxxxxxx-xxxxx
```

### 4. Paket ins n8n Pod kopieren

```bash
# Paket ins Pod kopieren (POD_NAME durch tatsächlichen Namen ersetzen)
oc cp bitovi-n8n-nodes-confluence-0.1.6.tgz <POD_NAME>:/tmp/

# Beispiel:
oc cp bitovi-n8n-nodes-confluence-0.1.6.tgz n8n-xxxxxxxx-xxxxx:/tmp/
```

### 5. Custom Node im Pod installieren/updaten

#### ⚠️ WICHTIG: Alte Versionen vorher deinstallieren

```bash
# Finde alle alten Installationen
oc exec <POD_NAME> -- sh -c "find /home/node -name '*confluence*' -type d 2>/dev/null"

# Entferne alte Custom Extensions (KRITISCH für saubere Installation)
oc exec <POD_NAME> -- sh -c "rm -rf /home/node/.n8n/custom/n8n-nodes-confluence"

# Entferne alte node_modules Installation  
oc exec <POD_NAME> -- sh -c "rm -rf /home/node/.n8n/node_modules/@bitovi/n8n-nodes-confluence"

# Entferne package-lock für saubere Installation
oc exec <POD_NAME> -- sh -c "rm -f /home/node/.n8n/package-lock.json"

# Bestätige dass alles sauber ist
oc exec <POD_NAME> -- sh -c "find /home/node -name '*confluence*' -type d 2>/dev/null || echo 'Clean - ready for installation!'"
```

**⚠️ WICHTIG - Korrektes Installationsverzeichnis:**

n8n erkennt Community Packages nur im **Custom Extensions Directory**: `/home/node/.n8n/custom`

**Environment Variable:** `N8N_CUSTOM_EXTENSIONS=/home/node/.n8n/custom`

#### Schritt 5a: Package entpacken und installieren
```bash
# Package im Pod entpacken
oc exec <POD_NAME> -- sh -c "cd /tmp && tar -xzf bitovi-n8n-nodes-confluence-0.1.6.tgz"

# Package ins Custom Extensions Directory kopieren
oc exec <POD_NAME> -- sh -c "cp -r /tmp/package /home/node/.n8n/custom/n8n-nodes-confluence"
```

**⚠️ HINWEIS:** Die ursprünglich dokumentierte Methode mit `npm install` funktioniert nicht in Umgebungen ohne npm Registry-Zugriff. Die direkte Entpackung und Kopie ist die empfohlene Methode.

#### Schritt 5b: Installation validieren
```bash
# Überprüfe installierte Version
oc exec <POD_NAME> -- sh -c "cat /home/node/.n8n/custom/n8n-nodes-confluence/package.json | grep version"

# Überprüfe ob Node-Dateien vorhanden sind
oc exec <POD_NAME> -- sh -c "ls -la /home/node/.n8n/custom/n8n-nodes-confluence/dist/"

# Sollte credentials und nodes Verzeichnisse zeigen
```

### 6. n8n Pod neu starten

```bash
# Pod neu starten (automatischer Restart durch OpenShift)
oc delete pod <POD_NAME>

# Warten bis neuer Pod läuft
oc get pods -w

# Warten bis Status "Running" und "2/2" erreicht ist
# CTRL+C zum Beenden des Watch-Modus
```

### 7. Installation überprüfen

```bash
# Neuen Pod Namen ermitteln
oc get pods | grep n8n

# Logs überprüfen
oc logs <NEW_POD_NAME> --tail=50
```

**Erwartete Log-Ausgabe (erfolgreich):**
```
✅ Keine "Unrecognized node type" Fehler
✅ n8n läuft stabil  
✅ Custom Node wird korrekt geladen
✅ Editor ist erreichbar: "Editor is now accessible via:"
```

### 8. Funktionalität in n8n testen

1. **n8n UI öffnen:**
   - Navigiere zur n8n Route im Namespace `mdb-test`
   - Finde die URL mit: `oc get route`

2. **Confluence Node testen:**
   - Erstelle neuen Workflow
   - Suche nach "Confluence" in der Node-Liste
   - Füge Confluence Node hinzu
   - Konfiguriere Atlassian Credentials:
     - Domain: `https://your-domain.atlassian.net`
     - Email: Deine Atlassian Email
     - API Token: Generiert aus Atlassian Account Settings

3. **Erwartetes Verhalten:**
   - ✅ Confluence Node wird in der Liste angezeigt
   - ✅ Credentials können gespeichert werden
   - ✅ API-Calls zu Confluence funktionieren
   - ✅ Pages können erstellt, gelesen, aktualisiert werden
   - ✅ Search funktioniert korrekt

4. **Test-Beispiel:**
   ```
   Operation: Get Page
   - Wähle "By ID" oder "By Title"
   - Gebe Page ID oder Title ein
   - Execute Workflow
   
   Expected Result:
   - Page-Daten werden erfolgreich abgerufen
   - JSON-Response mit Page-Content
   ```

## Troubleshooting

### Problem: "Unrecognized node type" Fehler

**Ursache:** Package ist nicht im korrekten Custom Extensions Directory installiert.

**Lösung:**
```bash
# Pod Logs detailliert prüfen
oc logs <POD_NAME> -f

# ✅ KORREKT: Custom Extensions Verzeichnis prüfen
oc exec <POD_NAME> -- ls -la /home/node/.n8n/custom/n8n-nodes-confluence

# ❌ FALSCH: node_modules wird von n8n NICHT gelesen
oc exec <POD_NAME> -- ls -la /home/node/.n8n/node_modules/@bitovi
```

**Environment Variable prüfen:**
```bash
# n8n Custom Extensions Pfad prüfen
oc exec <POD_NAME> -- sh -c "env | grep N8N_CUSTOM_EXTENSIONS"
# Sollte ausgeben: N8N_CUSTOM_EXTENSIONS=/home/node/.n8n/custom
```

### Problem: Credentials werden nicht gespeichert

**Ursache:** Credential-Typ nicht korrekt registriert

**Lösung:**
- Überprüfe ob Credential-Datei in `dist/credentials/` vorhanden ist
- Pod neu starten
- Browser-Cache leeren

### Problem: npm Cache-Berechtigungen

**Lösung:**
```bash
# Mit temporärem Cache installieren
oc exec <POD_NAME> -- sh -c "cd /home/node/.n8n && npm install /tmp/bitovi-n8n-nodes-confluence-0.1.6.tgz --cache /tmp/npm-cache"
```

## Node-Struktur

### Wichtige Dateien:
- `credentials/AtlassianCredentialsApi.credentials.ts` - Atlassian Credentials
- `nodes/Confluence/Confluence.node.ts` - Haupt-Node-Implementation
- `nodes/Confluence/Confluence.node.json` - Node Metadata
- `nodes/Confluence/confluence.svg` - Node Icon
- `package.json` - Node-Konfiguration und Dependencies
- `dist/` - Kompilierte TypeScript-Dateien

### Node-Konfiguration in package.json:
```json
{
  "n8n": {
    "n8nNodesApiVersion": 1,
    "credentials": [
      "dist/credentials/AtlassianCredentialsApi.credentials.js"
    ],
    "nodes": [
      "dist/nodes/Confluence/Confluence.node.js"
    ]
  }
}
```

## Erweiterte Optionen

### Telemetrie deaktivieren (optional)

Falls Sie die Telemetrie-Warnungen in den Logs entfernen möchten, können Sie dies über die n8n ConfigMap konfigurieren.

## Wichtige Erkenntnisse

### 🚨 KRITISCH: Korrektes Installationsverzeichnis
- **n8n liest NUR**: `/home/node/.n8n/custom/` (Environment Variable: `N8N_CUSTOM_EXTENSIONS`)
- **n8n liest NICHT**: `/home/node/.n8n/node_modules/` oder andere Verzeichnisse
- **Mehrfache Installationen** in verschiedenen Verzeichnissen können Konflikte verursachen
- **Lösung**: Package nach `/home/node/.n8n/custom/n8n-nodes-confluence/` kopieren

### Shell-Befehle
- Der Pod verwendet `sh` nicht `bash`
- npm Cache-Probleme erfordern `--cache /tmp/npm-cache`
- Befehlsausführung ohne interaktive Shell ist zuverlässiger

### Debugging Community Packages
```bash
# Prüfen ob Custom Extensions geladen werden:
oc logs <POD_NAME> | grep -i confluence

# Custom Extensions Verzeichnis prüfen:
oc exec <POD_NAME> -- ls -la /home/node/.n8n/custom/

# Mehrfache Installationen finden:
oc exec <POD_NAME> -- find /home/node -name '*confluence*' -type d 2>/dev/null
```

## Version History

- **0.1.0** - Initial Release
  - Confluence Page CRUD Operations
  - Atlassian API Authentication
  - Basic Search Functionality
- **0.1.6** - Current Version
  - Enhanced error handling
  - Improved documentation
  - Code quality workflows
  - Security scanning

## Support

Bei Problemen überprüfen Sie:
1. Pod-Logs: `oc logs <POD_NAME> --tail=50`
2. Node-Installation: `oc exec <POD_NAME> -- ls -la /home/node/.n8n/custom/n8n-nodes-confluence`
3. n8n UI für verfügbare Nodes und Credentials
