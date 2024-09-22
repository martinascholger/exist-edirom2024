# Einführung in XQuery
XQuery (XML Query Language) ist eine Abfragesprache, die speziell für die Verarbeitung von XML-Dokumenten entwickelt wurde. 
Sie ermöglicht es, komplexe Abfragen auf XML-Datenbanken oder Dokumenten durchzuführen, ähnlich wie SQL für relationale Datenbanken. 

# Datenmodell

# Sequenzen

# XQuery für Einzeldokumente und Sammlungen

Über die `doc()` Funktion werden einzelne XML-Dokumente eingelesen.

```doc('/db/apps/WeGA-data/letters/A0400xx/A040000.xml')```

Die `collection()` Funktion adressiert ein ganzes Verzeichnis, in dem XML-Dateien abgelegt sind. 

```collection('/db/apps/WeGA-data/letters')```

# TEI und XQuery

```declare default element namespace "http://www.tei-c.org/ns/1.0";```

# FLOWR Ausdrücke 
Das Herzstück von XQuery sind die FLOWR-Ausdrücke (flower). Sie ermöglichen komplexe Ausdrücke um Informationen aus Dateien und Sammlungen abzufragen und neu zu ordnen. 

- **for**: selektiert eine Sequenz an Knoten
- **let**: bindet eine Sequenz an eine Variable
- **where** (optional): filtert die Knoten, analog zu Prädikaten in XPath
- **order by** (optional): sortiert die Knoten
- **return**: liefert das Ergebnis zurück
