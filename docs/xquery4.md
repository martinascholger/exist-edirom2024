# XQuery, part 4

## JSON Ausgabe

Ausgabe eines JSON-Arrays mit der Verteilung der 10 häufigsten Briefschreiber:

```xquery
xquery version "3.1";

declare namespace tei="http://www.tei-c.org/ns/1.0";
declare namespace output = "http://www.w3.org/2010/xslt-xquery-serialization";

declare option output:media-type "application/json";
declare option output:method "json";

array {
    for $letter in collection('/db/apps/WeGA-data/letters')
    group by $sender := $letter//tei:correspAction[@type="sent"][tei:persName]/tei:persName[1] => normalize-space()
    order by count($letter) descending
    where $sender != ""
    return 
        array { $sender, count($letter) }
} => array:subarray(1, 10)
```
