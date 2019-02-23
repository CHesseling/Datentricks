# Kleiner Werkstattbericht NDR - WasAtmestDu

## Ein paar Tricks und Tipps rund um das Thema Datenputzen ##

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

Zur Info: Das nennt man eine Regular Expression oder RegEx. RegEx sind *die Hölle*, aber sehr mächtig. Es gibt mehrere Tools, mit denen man RegEx für seine Anwendung testen kann - das Beispiel für unsere Anwendung sieht man hier: https://regex101.com/r/Go3GUs/1 

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

Zurück in OpenRefine. Diesmal klickt man auf das Drop-Down neben "adresse" und wählt "Edit column" > "Add column by fetching URLs based on column adress". 

![OpenRefine](http://datenjournalismus.eu/github_pics/openrefine2.gif)

OpenRefine ruft dann quasi für jede Zelle eine URL auf (in diesem Fall die des Geocoders) und speichert die Ausgabe.

Zunächst muss man wieder den Namen der neuen Spalte bestimmen. Throttle delay kann heruntergesetzt werden auf z.B. 10 Millisekunden. Alle anderen Felder können so bleiben. 

Die GREL-Expression für den Google Geocoder sieht so aus:

```GREL
'https://maps.googleapis.com/maps/api/geocode/json?' +
'sensor=false&' + 'address=' + escape(value, 'url') + '&key=HierStehtDeinGoogleMapsAPIKey'
```

![OpenRefine](http://datenjournalismus.eu/github_pics/2019-02-19_22h17_24.png)

Nun legt der Geocoder los, und ein paar Minuten später steht das Ergebnis in unserer neuen Spalte "googlemaps".
Der Geocoder gibt dabei eine soegannte JSON-Struktur zurück, die OpenRefine speichert. Da sind die einzelnen Informationen verschachtelt enthalten:

![OpenRefine](http://datenjournalismus.eu/github_pics/2019-02-19_22h26_05.png)

Wir interessiern uns ja aber nur für die Lat/Long-Daten des Geocoders. Die können wir aus der JSON-Struktur extrahieren. Und zwar so: 

Den kleinen Drop-Down-Knopf links neben der Spalte "googlemaps" klicken. Dann "Edit column" > "Add column based on this column..." auswählen. 

Wir nennen die neue Spalte "lat" und nutzen diese GREL-Expression:
```GREL
value.parseJson().results[0].geometry.location.lat
```

Das gleiche dann nochmal für die Spalte "long" mit dieser GREL-Expression:
```GREL
value.parseJson().results[0].geometry.location.lng
```

### Reverse Geocoding ###

Da in unserer Karte auch die Adressen der Messorte angezeigt werden sollten, brauchten wir für die Fälle, bei denen es nur Lat/Long-Informationen gab, ein reverse geocoding. Auch das lässt sich mit Open Refine relativ einfach lösen. 

Diesmal klickt man auf das Drop-Down neben "latlong" und wählt "Edit column" > "Add column by fetching URLs based on column adress". 

```GREL
'https://maps.googleapis.com/maps/api/geocode/json?' +
'sensor=false&' + 'latlng=' + escape(value, 'url') + '&key=HierStehtDeinGoogleMapsAPIKey'
```

Die Ausgabe-Spalte kann man z.B. mit "reverse_adresse" benennen. Das Ergebnis ist wieder eine JSON-Rückmeldung vom Google Maps Geocoder.

*Problem:* Normalerweise würde man versuchen, mit dem ähnlichen Codeschnipsel wie bei den Adressen, die relevanten Informationen aus dem JSON zu ziehen.

Mit dem folgenden Code
```GREL
value.parseJson().results[0].address_components[1].long_name
```

lässt sich zwar in der Regel die Straße bestimmen - sucht man aber die Stadt, wird es schwierig, da bei manchen Einträgen an der Stelle der Stadtteil und bei anderen die Stadt zu finden ist:

![OpenRefine](http://datenjournalismus.eu/github_pics/2019-02-22_19h55_38b.png)

Dieses Problem lässt sich in OpenRefine afaik nicht einfach lösen, sondern man muss das ganze z.B. in Python verarbeiten.

Man kann höchstens alle adress_components in einzelne Spalten exrahieren und dann das Problem in Excel lösen. Also:
```GREL
value.parseJson().results[0].address_components[0].long_name
value.parseJson().results[0].address_components[1].long_name
value.parseJson().results[0].address_components[2].long_name
value.parseJson().results[0].address_components[3].long_name
[usw.]

```

## Ein paar Excel-Tricks ##

Das Ergebnis sieht dann so aus:

![OpenRefine](http://datenjournalismus.eu/github_pics/2019-02-22_20h17_28.png)


