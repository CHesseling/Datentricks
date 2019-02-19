# Datentricks

## Ein paar Tricks und Tipps rund um's Datenputzen ##

Für das NDR-Projekt #WasAtmestDu haben wir Menschen in Norddeutschland gebeten, Stickstoffoxid-Sammelröhrchen aufzuhängen. Insgesamt haben sich über 5000 Menschen über ein Online-Forumlar beworben. 

In dem Formular wurde nach dem Messort gefragt. Allerdings: Manche Menschen haben dort eine Adresse eingegeben, andere Geo-Koordinaten. Das sah dann so aus (die Adressdaten wurden von mir für dieses Tutorial anonymisiert).

![Excel-Daten](http://datenjournalismus.eu/github_pics/2019-02-19_09h48_16.png)

Das Ziel: Eine Spalte, in der valide Geo-Informationen stehen. Deshalb müssen zunächst die Adressen von den Lat/Long-Daten getrennt werden. 
Das geht am besten mit OpenRefine (http://openrefine.org/). OpenRefine läuft lokal auf dem eigenen Rechner und nutzt den Browser als Interface.

Nachdem die Excel-Tabelle geladen wurde, kann man nun die Daten "putzen".

### Adressen von Lat/Long-Daten trennen ###

Dafür als erstes auf den kleinen Drop-Down-Knopf links neben der Spalte "Ort" klicken. Dann "Edit column" > "Add column based on this column..." auswählen. Wir erstellen also eine neue Spalte, die auf unserer Spalte "Ort" basiert.

![OpenRefine](http://datenjournalismus.eu/github_pics/openrefine.gif)

Nun lassen sich Funktionen in der OpenRefine-Sprache GREL eingeben. Eine Übersicht gibt es unter anderem hier (https://github.com/OpenRefine/OpenRefine/wiki/Documentation-For-Users#reference) - allerdings ist die Übersetzung der Beispiele für die eigene Anwendung nicht immer einfach. 

Zunächst muss man in dem Fenster der neuen Spalte einen Namen geben - in unserem Fall zum Beispiel "latlong".

Mit diesem GREL-Schnipsel lassen sich die Lat/Long-Daten in eine neue Spalte kopieren:
```GREL
if(value.contains(/^(\d+(\.\d+)?),\s*(\d+(\.\d+))/),value,"")
```

![OpenRefine](http://datenjournalismus.eu/github_pics/2019-02-19_21h52_13.png)

Nachdem wir "Ok" geklickt haben, erscheint eine neue Spalte.

Jetzt kann man das Ganze nochmal wiederholen - wir nennen die neue Spalte "adresse". Und wir vertauschen die letzten beiden Ausdrücke in der GREL-Funktion:

```GREL
if(value.contains(/^(\d+(\.\d+)?),\s*(\d+(\.\d+))/),"",value)
```

Jetzt haben wir zwei saubere Spalten mit den Adressen und den Lat/Long-Angaben.

![OpenRefine](http://datenjournalismus.eu/github_pics/2019-02-19_22h07_04.png)

### Adressen geocoden

Im nächsten Schritt müssen wir die Adressen ebenfalls in Lat/Long-Informationen umwandeln. Der Vorgang nennt sich Geocoden. Es gibt unterschiedliche Dienste, die das anbieten, zum Beispiel Google Maps API, OpenStreetMap oder Mapbox.

Es ist relativ einfach, das Ganze mit der Google Maps API zu testen. Allerdings braucht man dafür mittlerweile einen API-Schlüssel. Wie man sich diesen - erstmal kostenlosen -  Schlüssel beschafft, steht hier: https://developers.google.com/maps/documentation/javascript/get-api-key

Zurück in OpenRefine. Diesmal klickt man auf das Drop-Down neben "adresse" und wählt "Edit column" > "Add column by fetching URLs based on column adresse". 

![OpenRefine](http://datenjournalismus.eu/github_pics/openrefine2.gif)

OpenRefine ruft dann quasi für jede Zelle eine URL auf (in diesem Fall die des Geocoders) und speichert die Ausgabe.
