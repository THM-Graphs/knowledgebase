Das Deployment erfolgt mit GitHub Actions. Auf den Testserver (`tei2spo.mni.thm.de`) wird bei jedem Push/Merge in den `main`-Branch deployet, beim Produktiv-Server muss die Pipeline über GitHub manuell angestoßen werden.

**Falls man das erste mal auf einen neuen Server deployet, bitte diese Schritte:**

### 1. Mit ssh per Terminal auf den Server gehen

```sh
# Falls der key nicht spezifiziert werden soll:
ssh ~/.ssh <username>@<server>

# Falls schon:
ssh -i ~/.ssh/<public-key-file> <username>@<server>
```

### 2. Repo clonen und `.env`-Datei anlegen

```sh
git clone https://github.com/THM-Graphs/atag-editor.git
cd atag-editor

touch .env

# Hostname und Passwort müssen angepasst werden
cat > .env <<EOF 
APP_HOST=<hostname>
APP_PORT=8080
NEO4J_URI=bolt+ssc://neo4j:7687
NEO4J_USER=neo4j
NEO4J_PW=<password>
NODE_ENV=production
PROTOCOL=https
EOF
```

### 3. HTTPS-Zertifikate für Neo4j einrichten (*ggf. überarbeiten*)

Die docker-compose-Konfiguration für den neo4j-Container benutzt die Standard-Zertifikatsdatei (`public.crt`) und den Standard-Key (`private.key`).  Das geht am einfachsten mit Symlinks auf die schon vorhandenen Dateien. In Zukunft gerne vereinfachen und die Dateien direkt so benennen (dann spart man sich auch den nächsten Schritt).

Der `certificates` Ordner wird auf gleicher Ebene angelegt wie der `atag-editor`, also nicht im Ordner  `atag-editor` selbst.

```sh
# certificates-Verzeichnis anlegen und reingehen
mkdir certificates
cd certificates

# Symlinks erzeugen
ln -s <zertifikat-name>.pem public.crt
ln -s <key-name>.key private.key
```
### 4. Config-Datei für Zertifikate einbauen (*ggf. überarbeiten*)

Der vorherige Schritt war nur für den Neo4j-Container, aber das ganze Setup benötigt HTTPS. Deswegen für den Nginx-Proxyserver auch noch die Zertifikate einbinden, indem man eine `certs.conf`-Datei erzeugt.

```sh
cat > ~/atag-editor/nginx/certs.conf <<EOF 
ssl_certificate /etc/nginx/certs/<dateiname>.pem;
ssl_certificate_key /etc/nginx/certs/<dateiname>.key;
EOF
```

### 5. Nutzer und Passwort festlegen
Der Einfachheit halber wird erstmal nur ein Nutzer + Passwort eingerichtet. In Zukunft soll aber ein flexibleres User Management System her.

```shell
# Muss vmtl erst installiert werden
sudo apt install apache2-utils

cd atag-editor

# Passwort erzeugen 
sudo htpasswd -c ./nginx/.htpasswd <username>
```

 **Bei Aufforderung neues Passwort in die Konsole eingeben.**

### 6. Container das erste mal bauen und hochziehen

Um den neuesten Stand zu bekommen, kann man das Setup manuell hochziehen. In Zukunft erfolgt das dann über GitHub Actions.

```sh
docker compose -f docker-compose.prod.yml up --build
```