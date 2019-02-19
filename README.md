# Datentricks

Ein paar Tricks und Tipps rund um's Datenputzen

Für das NDR-Projekt #WasAtmestDu haben wir Menschen in Norddeutschland gebeten, Stickstoffoxid-Sammelröhrchen aufzuängen. Insgesamt haben sich über 5000 Menschen über ein Online-Forumlar beworben. 

In dem Formular wurde nach dem Messort gefragt. Allerdings: Manche Menschen haben dort eine Adresse eingegeben, andere Geo-Koordinaten. Das sah dann so aus (die Adressdaten wurden von mir für dieses Tutorial anonymisiert).

![Excel-Daten](http://datenjournalismus.eu/github_pics/2019-02-19_09h48_16.png)

Das Ziel: Eine Spalte, in der valide Geo-Informationen stehen. Deshalb müssen zunächst die Adressen von den Lat/Long-Daten getrennt werden. 
Das geht am besten mit OpenRefine (http://openrefine.org/). OpenRefine läuft lokal auf dem eigenen Rechner und nutzt den Browser als Interface.

Nachdem die Excel-Tabelle geladen wurde, kann man nun die Daten "putzen".
Dafür als erstes auf den kleinen Drop-Down-Knopf links neben der Spalte "Ort" klicken. Dann "Edit column" > "Add column based on this column..." auswählen. 

![OpenRefine](http://datenjournalismus.eugithub_pics/openrefine.gif)
