# LB2

## Inhalt
- [LB2](#lb2)
  - [Inhaltsverzeichnis](#inhaltsverzeichnis)
    - [Netzwerkplan](#netzwerkplan)
    - [Bestehende vm aus Vagrant-Cloud eingerichtet](#bestehende-vm-aus-vagrant-cloud-eingerichtet)
    - [Vagrant-Befehle](#vagrant-befehle)
    - [VM-Webserver erstellen](#vm-webserver-erstellen)
    - [VM-Server mit UFW Firewall Config](#vm-server-mit-ufw-firewall-config)
    - [VM-Server mit Secure Shell Server](#vm-server-mit-secure-shell-server)
    - [VM-Server mit Secure Shell Server, Reverse Proxy, Benutzerberechtigungen und Firewallregeln](#vm-server-mit-secure-shell-server-reverse-proxy-benutzerberechtigungen-und-firewallregeln)
    - [Eingerichtet Umgebung Dokumentation vom Code](#eingerichtet-umgebung-dokumentation-vom-code)
  - [Testfälle](#testfälle)
    - [Zugriff funktioniert](#zugriff-funktioniert)
    - [Firewall eingerichtet inkl. Rules](#firewall-eingerichtet-inkl-rules)
    - [Benutzer- und Rechtvergabe ist eingerichtet](#benutzer--und-rechtvergabe-ist-eingerichtet)
    - [Zugang mit SSH-Tunnel abgesichert](#zugang-mit-ssh-tunnel-abgesichert)
    - [Reverse-Proky eingerichtet](#reverse-proky-eingerichtet)
    - [Port forwarding](#port-forwarding)
  - [Persönliche Lernentwicklung](#persönliche-lernentwicklung)
    - [Vergleich Vorwissen - Wissenszuwachs](#vergleich-vorwissen---wissenszuwachs)
      - [Vorwissen im Bezug zum Modul](#vorwissen-im-bezug-zum-modul)
      - [Neues Wissen](#neues-wissen)
    - [Reflexion](#reflexion)
      - [Tag 1](#tag-1)
      - [Tag 2](#tag-2)
      - [Tag 3](#tag-3)
      - [Tag 4](#tag-4)

<br>
<br>

### Netzwerkplan
<br>

![Netzwerkplan](https://github.com/nils1320/M300/blob/main/Pictures/8.png?raw=true)

<br>

### Bestehende vm aus Vagrant-Cloud eingerichtet
```Shell
      $ vagrant init ubuntu/xenial64        #Vagrantfile erzeugen
      $ vagrant up --provider virtualbox    #Virtuelle Maschine erstellen & starten
``` 

### Vagrant-Befehle

Hier sind die wichtigsten Befehle:

| Befehl                    | Beschreibung                                                      |
| ------------------------- | ----------------------------------------------------------------- | 
| `vagrant init`            | Initialisiert im aktuellen Verzeichnis eine Vagrant-Umgebung und erstellt, falls nicht vorhanden, ein Vagrantfile |
| `vagrant up`              |  Erzeugt und Konfiguriert eine neue Virtuelle Maschine, basierend auf dem Vagrantfile |
| `vagrant ssh`             | Baut eine SSH-Verbindung zur gewünschten VM auf                   |
| `vagrant status`          | Zeigt den aktuellen Status der VM an                              |
| `vagrant port`            | Zeigt die Weitergeleiteten Ports der VM an                        |
| `vagrant halt`            | Stoppt die laufende Virtuelle Maschine                            |
| `vagrant destroy`         | Stoppt die Virtuelle Maschine und zerstört sie.                   |

Weitere Befehle unter: https://www.vagrantup.com/docs/cli/

<br>

### VM-Webserver erstellen 

Um die VM zu erstellen, habe ich diese Schritte erfüllt:

1. Erstellen eines Ordners, in dem die VM gespeichert werden soll.
2. Erstellen einer Datei namens "Vagrantfile" (ohne Dateiendung) in diesem Ordner.
3. Öffnen des "Vagrantfile" mit einem Texteditor und den folgenden Inhalt einfügen:<br>
```
Vagrant.configure(2) do |config|
    config.vm.box = "ubuntu/xenial64"
    config.vm.network "forwarded_port", guest:80, host:100, auto_correct: true
    config.vm.synced_folder ".", "/var/www/html"  
  config.vm.provider "virtualbox" do |vb|
    vb.memory = "512"  
  end
  config.vm.provision "shell", inline: <<-SHELL
    # Packages vom lokalen Server holen
    # sudo sed -i -e"1i deb {{config.server}}/apt-mirror/mirror/archive.ubuntu.com/ubuntu xenial main restricted" /etc/apt/sources.list 
    sudo apt-get update
    sudo apt-get -y install apache2 
  SHELL
  end
```
Nun muss ich nur noch in der Shell  das Verzeichnis suchen und "`vagrant up`" eintippen.
<br>

### VM-Server mit UFW Firewall Config

Bei dieser VM werde ich gleich die Firewall mit installieren und zusätzlich noch die so genannten 'rules' erstellt.
```
Vagrant.configure(2) do |config|
    config.vm.box = "ubuntu/xenial64"
    config.vm.network "forwarded_port", guest:80, host:100, auto_correct: true
    config.vm.synced_folder ".", "/var/www/html"  
  config.vm.provider "virtualbox" do |vb|
    vb.memory = "512"  
  end
  config.vm.provision "shell", inline: <<-SHELL
    # Packages vom lokalen Server holen
    # sudo sed -i -e"1i deb {{config.server}}/apt-mirror/mirror/archive.ubuntu.com/ubuntu xenial main restricted" /etc/apt/sources.list 
    sudo apt-get install ufw
    sudo ufw enable
    sudo ufw allow 80/tcp
    sudo ufw allow from 192.168.1.122 to any port 22
    sudo ufw allow from 192.168.1.124 to any port 3306
  SHELL
  end
```
<br>

### VM-Server mit Secure Shell Server

Bei dieser VM werde ich ein Secure Shell Server mit zusätzlichen 'Rules' erstellt.
```
Vagrant.configure(2) do |config|
  config.vm.box = "ubuntu/xenial64"
  config.vm.network "forwarded_port", guest:80, host:8080, auto_correct: true
  config.vm.synced_folder ".", "/var/www/html"  
config.vm.provider "virtualbox" do |vb|
  vb.memory = "512"  
end
config.vm.provision "shell", inline: <<-SHELL
	sudo ufw --force enable
	sudo ufw allow 22
	sudo ufw allow 2222
	sudo systemctl start ssh
	sudo sed -i "s/.*PasswordAuthentication.*/PasswordAuthentication yes/g" /etc/ssh/sshd_config
	sudo sed -i "s/.*ChallengeResponseAuthentication.*/ChallengeResponseAuthentication yes/g" /etc/ssh/sshd_config
	sudo reboot
SHELL
end
```
<br>

### VM-Server mit Secure Shell Server, Reverse Proxy, Benutzerberechtigungen und Firewallregeln

Bei dieser VM istalliere ich ein Secure Shell Server, Reverse Proxy, Benutzerberechtigungen und Firewallregeln.
```
Vagrant.configure(2) do |config|
  config.vm.box = "ubuntu/xenial64"
  config.vm.network "forwarded_port", guest:80, host:8080, auto_correct: true
  config.vm.synced_folder ".", "/var/www/html"  
config.vm.provider "virtualbox" do |vb|
  vb.memory = "512"  
end
config.vm.provision "shell", inline: <<-SHELL
	#VM Update
	sudo apt-get update
	#Apache Webserver installieren
	sudo apt-get -y install apache2
	#Firewall "enablen"
	sudo ufw --force enable
	#Port 22 und 2222 erlaunben/freischalten
	sudo ufw allow 22
	sudo ufw allow 2222
	#SSH-Service starten
	sudo systemctl start ssh
	#Im sshd_config File zwei "Statments" mit Ja austauschen, damit keine Fehlermeldung kommnt
	sudo sed -i "s/.*PasswordAuthentication.*/PasswordAuthentication yes/g" /etc/ssh/sshd_config
	sudo sed -i "s/.*ChallengeResponseAuthentication.*/ChallengeResponseAuthentication yes/g" /etc/ssh/sshd_config
	#Owner von edm Verzeichis /var/mail an den User vagrant geben
	sudo chown -c vagrant /var/mail
	#Nur noch den Besitzer (in diesem Fall vagrant) auf das Verzeichnis erlauben
	sudo chmod -R 700 /var/mail
	#Nginx --> Reverse-proxy installieren
	sudo apt-get -y install nginx
	#Virtual Host deaktivieren
	sudo unlink /etc/nginx/sites-enabled/default
	#File für Reverse-proxy erstellen
	sudo touch /etc/nginx/sites-available/reverse-proxy.conf
	#Inhalt in File einfügen
	cat <<%EOF% | sudo tee -a /etc/nginx/sites-available/reverse-proxy.conf
	server {
		listen 8080;
		location / {
			proxy_pass http://127.0.0.1;
		}
	}
%EOF%
	sudo ln -s /etc/nginx/sites-available/reverse-proxy.conf /etc/nginx/sites-enabled/reverse-proxy.conf
	#Apache Service stoppen
	sudo systemctl stop apache2
	#Nginx Service starten
	sudo systemctl restart nginx
	#VM rebooten
	sudo reboot
SHELL
end
```


<br><br>

### Eingerichtet Umgebung Dokumentation vom Code
Hier habe ich den verwendeten Code und die Erklärung zu den jeweiligen Commands aufgelistet.

Vagrant konfiguration initialisieren

    `Vagrant.configure(2) do |config|`


Welche Ubuntu Version verwendet wird
    
    `config.vm.box = "ubuntu/bionic64"`


Port forwarding von 80 auf 8011.
    
    `config.vm.network "forwarded_port", guest:80, host:8011, auto_correct: false`


Bestimmen mit welchem Programm die VM erstellt werden soll
    
    `config.vm.provider "virtualbox" do |vb|`


Festlegen wieviel Arbeitsspeicher verwendet werden darf
    
    `vb.memory = "1024"`

Ende der Vagrant Config
    
    `end`


VM Config einleiten. Festlegen das folgende Zeilen in der Shell geschrieben werden.

    `config.vm.provision "shell", inline: <<-SHELL`


Neueste Updates herunterladen

    `sudo apt-get update`


Installieren von diversen Diensten
    
    `sudo apt-get install -y \`
    `ca-certificates \`
    `curl \`
    `gnupg \`
    `ufw \`
    `lsb-release`


Verifikation des Dockerimages

    `curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor `-o /usr/share/keyrings/docker-archive-keyring.gpg`
    `echo \`
    `"deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/`docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \`
    `$(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > ``/dev/null`


Firewall Rules setzen
    
    `sudo ufw --force enable`
    `sudo ufw allow 80/tcp`
    `sudo ufw allow 22`
    `sudo ufw allow 2222`


Install Docker

    `sudo apt-get install docker-ce docker-ce-cli containerd.io -y`
    
    
Volume für nginx erstellen und Nginx aktivieren und Port festlegen    
    
    `docker volume create nginx_data`
    `docker run --name some-nginx -d -p 80:80 nginx`


Vagrant Config Ende
    
    `SHELL`
    `end`

<br><br>

## Testfälle

### Zugriff funktioniert

Zuerst konnte ich die VM mit dem Vagrantfile nicht erstellen.

![screen 1](https://github.com/nils1320/M300/blob/main/Pictures/1.png?raw=true)

<br>
Nun geht es. Ich hatte vorhin Port 101 für den Host verwendet, aber mit diesem ging es nicht. Also habe ich den Port zu 8011 gewechselt da dieser sicher frei ist. Nun geht es endlich.
<br>

![image](https://github.com/nils1320/M300/blob/main/Pictures/5.png?raw=true)

<br>


### Firewall eingerichtet inkl. Rules
Hier wurde die Firewall konfiguriert

Firewall Rules setzen
    
    `sudo ufw --force enable`
    `sudo ufw allow 80/tcp`
    `sudo ufw allow 22`
    `sudo ufw allow 2222`

Da der Zugriff funktioniert, sind auch die Rules erfolgreich gesetzt worden.

### Benutzer- und Rechtvergabe ist eingerichtet

Alle Benutzer sind erfolgreich und korrekt eingerichtet. Dies kann man am File in /etc/passwd sehen.
<br>
![image](https://github.com/nils1320/M300/blob/main/Pictures/6.png?raw=true)
<br>

### Zugang mit SSH-Tunnel abgesichert

Hier greife ich mit SSH über Putty zu.
<br>
![screen 2](https://github.com/nils1320/M300/blob/main/Pictures/2.png?raw=true)
<br>


### Reverse-Proky eingerichtet
Geht der Zugriff mit localhost:8011, so habe ich den Reverse Proxy erfolgreich konfiguriert. Dies habe ich mit Curl getestet.

![screen 3](https://github.com/nils1320/M300/blob/main/Pictures/3.png?raw=true)
<br>


<br>

### Port forwarding

Um den Zugriff auf die Webseite zu erlangen, habe ich den Port 80 auf 8011 weitergeleitet.
<br>
![image](https://github.com/nils1320/M300/blob/main/Pictures/7.png?raw=true)

<br>
<br>


## Persönliche Lernentwicklung
### Vergleich Vorwissen - Wissenszuwachs

#### Vorwissen im Bezug zum Modul
- Basic Linux Kenntnise
- Schonmal von Container gehört aber nie verwendet

#### Neues Wissen
- Auch wenn ich noch nie mit Git Hub gearbeitet habe, habe ich gelernt mit dem Tool zu arbeiten.
- Ich habe Vagrant kennengelernt
- Ich durfte erfahren wie man den Stagingprozess massiv effizzienter getsallten kann.
- Der SSH Key liess sich ganz einfach implenterien.
- Ich lernte wie man eine Firewall ganz einfach mit Linux installiert
- Ich habe viel über Proxy gelernt.

<br>

### Reflexion
#### Tag 1
An diesem Tag hatten wir zunächst eine theoretische Einführung, um einen Überblick über das Modul zu bekommen. Bei den praktischen Arbeiten traten jedoch Probleme auf. Die virtuelle Maschine in Virtualbox startete nicht, aber durch die Erhöhung der Anzahl der Kerne konnte sie erfolgreich installiert werden. Die Installation von Apache auf der VM verlief zum Glück reibungslos. Mit Vagrant gab es noch Probleme, doch aufgrund von Zeitmangel konnten wir keine Lösung finden. Trotz der Herausforderungen konnten wir Fortschritte erzielen, indem wir Apache erfolgreich installiert haben. Wir werden uns jedoch weiterhin bemühen, eine Lösung für die Probleme mit Vagrant zu finden und unser Verständnis für diese Technologien zu vertiefen.

#### Tag 3
Heute habe ich viel Zeit damit verbracht, ein Vagrantfile zu erstellen, um eine virtuelle Maschine mit Nginx einzurichten. Dabei bin ich auf verschiedene Fehler gestoßen. Einer davon war, dass der Port 100 bereits belegt war, weshalb ich stattdessen den Port 101 verwendet habe. Obwohl die VM erfolgreich erstellt wurde, konnte ich nicht über den Webbrowser darauf zugreifen. Nachdem ich den Port auf 8011 geändert hatte, funktionierte der Zugriff endlich. Trotz der Herausforderungen konnte ich letztendlich eine funktionierende Umgebung mit Nginx schaffen und habe dabei gelernt, wie wichtig Geduld und das Ausprobieren verschiedener Lösungswege sind.

#### Tag 4
Heute startete ich meine Vagrant-VM mithilfe des Befehls "vagrant up", um meine Arbeit fortsetzen zu können. Alles lief reibungslos, und ich konnte Testfälle erstellen und durchführen, ohne auf Probleme zu stossen. Anschliessend aktualisierte ich meine Dokumentation, indem ich die Punkte aus dem Bewertungsraster, die die Umgebungseinrichtung betrafen, entfernte, da sie bereits gut dokumentiert waren. Es war überraschend, aber erfreulich, dass heute keine größeren Probleme auftraten. Diese reibungslose Erfahrung ermöglichte es mir, meine Aufgaben effizient zu erledigen und mich auf andere Aspekte meiner Arbeit zu konzentrieren.

