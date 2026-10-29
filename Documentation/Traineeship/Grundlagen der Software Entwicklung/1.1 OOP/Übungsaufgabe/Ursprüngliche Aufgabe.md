## Summary
#### Grundlegendes

- Tourismus-Konzern möchte Web App entwickeln, um besser datengetriebene Entscheidungen zu treffen z.b. Preisanpassugnen
- Es sollen die folgenden Dinge entwickelt werden:
	1. Pipelines für Ingestion der Daten
	2. Ein sinnvolles Modell für die Daten
	3. Ein kleiner Backend Service, welcher die Ingestion ausführt und Preisempfehlung berechnet
- Der Kunde stellt **jeden Tag** folgende `.csv` files zur Verfügung:
	1. `capacity.csv`
	2. `bookings.csv`
	3. `prices.csv`
- Man kann völlig frei bezüglich Name, Anzahl, etc., der Spalten
#### Aufgabe

- Entwickele Python Anwendung, welche die folgende Endpunkte bereitstellt:
	1. `/start-ingestion`
		- nimmt Ordnerpfad entgegen, welche die 3 **täglichen** Dateien beinhaltet 
		- führt import der Daten in Datenbank aus
		- **Datenmodell soll sinnvoll sein für unsere Aufgabe und muss nicht identisch zur Struktur der .csv files sein**
		- **Dateien enthalten immer den vollen Datensatz -> können Daten vom Vortag überschreiben**
	2. `/price-recommendation`
		- soll Preisempfehlungen für alle Produkte berechnen (Basis aktuelle Buchungslage) und in Datenbank speichern
		- Nutzt für Empfehlung simple Heuristik: $$
		\text{Anpassung}=15\% \cdot(\text{Auslastung}-0.5)\cdot\left( \frac{\text{Tage bis Resise}}{300} +0.5 \right)
		$$
	3. `/max-recommendation`
		- Gibt `PriceRecommendation` Objekt zurück, welches alle relevanten Informationen zu der buchbaren Einheit mit der höchsten empfohlen Preisanpassung enthält
		- **Bei mehreren Option kann eine einzige zurück gegeben werden**
- Damit Datenmodell reproduzierbar und verständlich ist, nutze am besten ein Datenmigrationstool
- Schreibe Sinnvolle Tests
#### Details `.csv`

- `product id` entspricht einem Unterkunftstyp
- Eine buchbare Einheit ist ein Produkt zu bestimmten Startdatum
- `capacity.csv` enthält eine Zeile pro buchbare Einheit
1. `bookings.csv`
	- folgende **Spalten sind nicht relevant**: `cust_id`, `feature_2`
	- cancelled bookings sollen beim import ignoriert werden
	- `date` ist das Datum an welchem die Buchung startet
	- `bkg_nights` gibt an wie lange buchung geht
	- `id` ist die ID aus dem Datewarehouse des Kunden und ist keine wichtige Information -> **ist unique**
	- `bkg_id` ist im **nicht zwingend unique** -> wenn innerhalb derselben Session mehrere Unterkünfte gebucht wurden haben sie die gleiche `bkg_id`
2. `capacity.csv`
	- wird für Berechnung der Kapazität und Buchungslage benötigt
3. `prices.csv`
	- enthält die aktuellen Preise für alle buchbaren Einheiten und unterschiedliche Buchungsdauern


[Link zu der Aufgabe](https://github.com/Haeger-Trainee-Applications/Bewerberaufgabe_data_engineering#bewerberaufgabe---data-engineering)
## Bewerberaufgabe - Data Engineering
#### Einleitung

Wir stellen uns an dieser Stelle vor, dass ein Kunde einen Auftrag für die individuelle Entwicklung dieses Projektes an dich übergeben hat.

Kunden haben häufig Probleme alle Fragen der Entwickler beim ersten Versuch zu beantworten. In dieser Aufgabe gibt es keinen Kunden, daher darfst du deiner Kreativität freien Lauf lassen wenn die Anforderungen unklar sind.
### Aufgabenstellung

Ein Tourismus-Konzern möchte für seine BI Abteilung eine Web App entwickeln, um besser datengetriebene Business Entscheidungen (z.B. Preisanpassungen) treffen zu können. Hierfür sollen wir...
- Pipelines für die Ingestion der Daten in unsere relationale Datenbank entwickeln,
- ein sinnvolles Datenmodell wählen,
- und einen kleinen prototypischen Backend-Service bereitstellen, der die Ingestion triggert und Preisempfehlungen berechnet.
Der Kunde wird uns jeden Tag 3 .csv Dateien zukommen lassen:
- capacity.csv
- bookings.csv
- prices.csv
Zu Entwicklungszwecken stellt der Kunde uns einen Ordner mit jeweils einer dieser Dateien zur Verfügung. Du findest ihn in diesem Repository.

**Deine Aufgaben:**

1. eine Python Anwendung schreiben, die 3 Endpunkte nach außen bereitstellt:
	- `/start-ingestion`: nimmt einen Ordnerpfad entgegen, welcher die 3 täglichen Dateien beinhaltet. Führt den Import der Daten in unsere Datenbank aus. Das Datenmodell unserer Datenbank soll für unsere Anwendung sinnvoll sein und muss nicht zwangsweise identisch zur Struktur der .csv Dateien sein. Die Dateien enthalten immer den vollen Datensatz (nicht nur tägliche updates). Die Daten des Vortags können beim Import der aktuellen Daten also überschrieben werden.
	- `/price-recommendations`: soll Preisempfehlungen für alle Produkte auf Basis der aktuellen Buchungslage berechnen und diese in der Datenbank speichern. Da wir aktuell nur einen Prototypen bauen, nutzen wir hierfür eine übersimplifizierte Heuristik:
    `empfohlene Preisanpassung (in %) = 15% * (a - 0.5) * (0.5 + d)`
    `a = Momentane Auslastung (in %)`
    `d = (Tage bis zum Reiseantritt / 300)`
    Also wenn ein Produkt zum Beispiel 40 Tage vor Reiseantritt zu 75% ausgebucht ist, empfiehlt unsere Funktion eine Preisanpassung von `15% * (0.75 - 0.5) * (0.5 + (40 / 300)) = +2,375%`
	    Preisempfehlungen müssen nur für Buchungen berechnet werden, die in der Zukunft liegen. Preisanpassungen für bereits vergangene Reisen machen natürlich wenig Sinn.
	- `/max-recommendation`: Gibt ein `PriceRecommendation` Objekt zurück, welches alle relevanten Informationen zu der buchbaren Einheit mit der höchsten empfohlenen Preisanpassung enthält (in Prozent). Falls mehrere buchbare Einheiten die maximal empfohlene Preisanpassung haben, kann irgendeine davon zurückgegeben werden.
2. Damit dein Datenmodell leicht reproduzierbar und für uns verständlich ist, nutze am besten ein Datenmigrationstool wie z.B. Liquibase oder Alembic. Wenn du damit noch keine Erfahrung hast, ist ein SQL Setup Script auch in Ordnung.
3. Stelle außerdem mithilfe von sinnvollen Tests sicher, dass deine Anwendung korrekt funktioniert. (Wenn die Zeit knapp wird darfst du hier gerne priorisieren und nur 1-2 Unittests schreiben, um ein Verständnis des Konzepts zu demonstrieren)

Wenn du das Gefühl hast bestimmte Entscheidungen begründen oder dokumentieren zu wollen, nutze dafür eine README.md.
### Details zu den CSV Input Daten

Du kannst frei entscheiden wie viele Tabellen mit wie vielen Spalten und welchen Beziehungen die Datenbank haben soll. Generell gilt: Wenn du unzufrieden mit Spaltennamen oder dem Datenmodell bist, ändere was du möchtest. Eine gründliche Analyse der Daten hilft vermutlich, um ein geeignetes Modell zu finden - erst recht da es keinen Kunden gibt an den du Rückfragen stellen kannst. Falls du Helper-Skripte (oder notebooks) für die Datenanalyse verwendest, darfst du das gerne zeigen indem du sie in dein Repository integrierst.

Ein Produkt (`product id`) entspricht einem Unterkunftstyp (z.B "2 Bed Room Premium-Class Apartment). Eine "buchbare Einheit" ist ein Produkt zu einem bestimmten Startdatum. Die meisten Messwerte (z.B. `Momentane Auslastung (in %)`) werden also nicht pro product_id berrechnet, sondern pro buchbare Einheit. Welche Unterkünfte zu welchem datum buchbar sind, sieht man unter anderem in `capacity.csv`, die eine Zeile pro buchbare Einheit enthält.
###### Bookings.csv

- Folgende Spalten sind für uns nicht relevant und müssen daher nicht importiert werden: `cust_id, feature_2`
- Cancelled Bookings sollen beim Import ebenfalls ignoriert werden.
- `date`ist das Datum an welchem die Buchung startet. `bkg_nights`gibt an, wie lange die Buchung geht.
- Die Spalte `id`ist die ID aus dem Datawarehouse unseres Kunden und ist für uns somit keine wichtige information. Du darfst selbst entscheiden ob du diese id trotzdem nutzen möchtest, lieber eine eigene id generieren möchtest oder gänzlich auf IDs verzichten willst.
- `bkg_id` ist im Gegensatz zu `id` nicht zwingend unique. Wenn 1 Kunde innerhalb derselben Session mehrere Unterkünfte bucht, erscheinen all diese Unterkünfte als separate Zeilen aber mit gleicher `bkg_id`.
###### Capacity.csv

- Wird vor allem für die Berechnung der Kapazität und Buchungslage aller buchbarer Einheiten benötigt.
- `Gesamtkapazität einer buchbaren Einheit` = `noch buchbare Unterkünfte` + `bereits gebuchte Unterkünfte`
    
    `noch buchbare Unterkünfte` = `is_option`+ `is_bookable` (Spalten aus capacity.csv)
    
    `bereits gebuchte Unterkünfte` = Summe aller Einträge zu dieser buchbaren Einheit in Bookings.csv
- Die Spalte `id`ist die ID aus dem Datawarehouse unseres Kunden und ist für uns somit keine wichtige information. Du darfst selbst entscheiden ob du diese id trotzdem nutzen möchtest, lieber eine eigene id generieren möchtest oder gänzlich auf IDs verzichten willst.
###### Prices.csv

- enthält die aktuellen Preise für alle buchbaren Einheiten und unterschiedliche Buchungsdauern (`length_of_stay`)
- Die Spalte `id`ist die ID aus dem Datawarehouse unseres Kunden und ist für uns somit keine wichtige information. Du darfst selbst entscheiden ob du diese id trotzdem nutzen möchtest, lieber eine eigene id generieren möchtest oder gänzlich auf IDs verzichten willst.
#### Erlaubte Technologien

- SQL
- Postgres
- Python (alle Bibliotheken)
- Docker (darf, aber kein muss)
- Datenmigrationstools wie z.B. Liquibase oder Alembic
#### Bewertungskriterien & Erwartungshaltung

Die Aufgabe ist auf ca. 40 Arbeitsstunden ausgelegt. Uns ist bewusst, dass das vermutlich nicht genug Zeit ist um eine perfekte Lösung abzuliefern. In erster Linie ist uns folgendes wichtig:
- die Funktionalität stimmt. Die Anwendung liefert richtige Ergebnisse und du hast alle gewünschten Features implementiert
- du verstehst deine eigene Lösung. Du bist dir über mögliche Schwächen und Tradeoffs deiner Lösung bewusst bist. Du kennst und verstehst deinen Code gut.
Sollten uns nach deiner Abgabe größere Fehler oder Missverständnisse auffallen, bekommst du Feedback und die Möglichkeit nachzubessern.

Deine Abgabe wird im Kontext zu deiner Vorerfahrung gewertet. Es ist nicht notwendig eine produktionsreife Lösung zu bauen.  
Wir erwarten eine funktionierende, nachvollziehbare Lösung mit sinnvollen Annahmen.  
Vereinfachungen sind ausdrücklich erlaubt, solange sie dokumentiert werden. Wenn du aus Zeitgründen Prioritäten setzen musst, dokumentiere deine Entscheidungen.