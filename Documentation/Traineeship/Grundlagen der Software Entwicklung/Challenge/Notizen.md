## Diagramme

Aus dem Code in `~/HaegerConsulting/Projects/Traineeship/Challenges/challenge-geldautomaten` abgeleitet:

- [[Challenge Geldautomaten - Klassendiagramm (kompakt).canvas|Klassendiagramm (kompakt)]] — klassische UML-Sicht: Klassen, Stereotypen, Beziehungen
- [[Challenge Geldautomaten - Klassendiagramm.canvas|Klassendiagramm (detailliert)]] — zusätzlich alle Signaturen und Verantwortlichkeiten pro Klasse
- [[Challenge Geldautomaten - ER-Diagramm.canvas|ER-Diagramm]] — Tabellen, Schlüssel und Beziehungen aus `persistence/orm.py`
- [[Challenge Geldautomaten - Use-Case-Diagramm (kompakt).canvas|Use-Case-Diagramm (kompakt)]] — Akteure, Systemgrenzen, Anwendungsfälle
- [[Challenge Geldautomaten - Use-Case-Diagramm.canvas|Use-Case-Diagramm (detailliert)]] — zusätzlich Methoden, «include»/«extend» und Fehlerfälle

## Probleme, welche mich dazu gebracht haben etwas zu ändern

- `services.Repository` scheint für die Persistenz von verschiedenen Objekten zuständig zu sein
	-> hat nur eine `.save()` Methode, aber andere Methoden, welche sich spezifisch auf verschiedenen Objekte beziehen (e.g. `find_bankkonto_...`, `find_geldautomat_...`)
	-> mir fällt als einzige Lösung Fallunterscheidung in `.save()` ein, welche unterscheidet, welcher Service/mit welchem Objekt gerade aufgerufen wird (also der From `if isinstance(obj, ...)`)
		$\implies$ zerstört aus meiner Sicht genau den Punkt, weshalb wir ein Interface verwenden
		$\implies$ Lösung: In verschieden Repositories aufteilen pro Objekt, welches persistiert werden soll
- Signatur Problem:
	- Ich will Datenobjekt i.e. Modell in orm.py und Domain Objekt trennen. Das muss ich suaber machen