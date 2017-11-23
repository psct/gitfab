## Hinweise zum empfohlenen Betrieb

Das ist ein simples Beispielprojekt zu einem Artikel [aus c't
25/2017](https://www.heise.de/ct/ausgabe/2017-25-GitLab-CI-CD-Mietserver-bauen-Software-3894125.html). Es erstellt mittels Docker einen GitLab-Server inklusive CI/CD. 

Um das Beispiel nachbauen zu können, benötigen Sie eine eigene Domain, um
zwei Hostnamen mit den erstellten Containern zu verbinden. [Teile lassen sich
auch ohne eigene Domain nachstellen](#lokal-ausprobieren).

Das Docker-Compose-File gitfab.yml startet fünf Container:

https://github.com/jwilder/nginx-proxy  
https://github.com/JrCs/docker-letsencrypt-nginx-proxy-companion  
gitlab  
gitlabrun  
gitlabmachine  

Der .env-Datei entnimmt Docker-Compose, mit welchem Domain-Namen diese
GitLab-Instanz verknüpft werden soll. Die Hosts registry.<i>IhreDomain</i>
und gitlab.<i>IhreDomain</i> müssen auf die IP-Adresse des Hosts verweisen,
auf dem Sie die Container starten. 

Die erstgenannten Container beschaffen automatisch SSL-Zertifikate bei Let's 
Encrypt. Wenn Sie das nicht wünschen, um beispielsweise nicht das 
Zertifikatslimit bei Let's Encrypt zu sprengen, können Sie im 
Docker-Compose-File vor dem Start die Option setzen, damit nur 
Versuchszertifikate ausgestellt werden.

Wenn Docker-Compose die Container gestartet und konfiguriert hat,
müssen Sie sich abschließend an einer Konfigurationsdatei im GitLab-Container 
zu schaffen machen, wechseln Sie in den Container mit 
`docker exec -it gitlab bash`. Öffen Sie dort mit nano die Datei /etc/gitlab/gitlab.rb.

Suchen Sie dort nach "Registry NGINX" und entfernen Sie die
Kommentarzeichen (#) vor den Zeilen dieses Blocks

```
#registry_nginx['proxy_set_headers'] = {
#  "Host" => "$http_host",
#  "X-Real-IP" => "$remote_addr",
#  "X-Forwarded-For" => "$proxy_add_x_forwarded_for",
#  "X-Forwarded-Proto" => "https",
#  "X-Forwarded-Ssl" => "on"
#}
```

Rufen Sie anschließend noch im
Container die Neukonfiguration auf: `gitlab-ctl reconfigure`. 

Ohne diesen Eingriff klappt der Zugriff auf die GitLab-eigene Registry
in der vom Artikel vorgesehenen Konfiguration nicht. Einen Weg, diese
Arbeit im Rahmen eines Docker-Compose-Aufrufs zu automatisieren, habe
ich leider nicht gefunden.

In der Docker-Compose-Datei gitfab.yml finden Sie auskommentierte
Optionen, um die E-Mail-Konfiguration des GitLab-Containers zu
setzen. Das ist für den dauerhaften Betrieb sehr sinnvoll. Für das
Ausprobieren ist es aber nicht notwendig.

Der Artikel erläutert, wie Sie weiter vorgehen, um einen einfachen
Container als Beispiel zu bauen, wenn Sie eine GitLab-Runner-Instanz
in Ihrem GitLab-Server registriert haben. Für einen Beispiel-Container
nutzbare Dateien stecken im Verzeichnis demo. 

Wenn bei Ihnen schon Container aus den Images jwilder/nginx-proxy
und jrcs/letsencrypt-nginx-proxy-companion laufen, entfernen Sie diese
aus der Docker-Compose-Datei oder starten Sie nur die anderen Container. 
Passen Sie dann unbedingt den Namen des Netzwerks in dieser Datei und der 
GitLab-Runner-Konfiguration an die Verhältnisse auf Ihrem System an.

## Lokal ausprobieren

Wenn der Artikel Sie auf den Geschmack gebracht hat, GitLab nebst Autobuild mal 
auszuprobieren, geht das eingeschränkt auch ohne eine eigene Domain. Allerdings 
klappt dann nur der Betrieb der Plattform nebst lokal ausgeführten Runner und 
leider ohne abschließendes Push des Beispiel-Containers in die Registry – 
eine Docker-Registry lässt sich in diesem Szenario nur schwer bis gar nicht 
dazu überreden, ohne SSL zu arbeiten. 

Das Docker-Compose-File gitfab_local.yml enthält eine entsprechend abgespeckte 
Bauanleitung. Benutzen Sie zum Verbinden des GitLab- und 
GitLab-Runner-Containers dann den Hostnamen "gitlab".