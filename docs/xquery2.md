# XQuery, part 2

```xquery
xquery version "3.1";

(:
 : Namespace declarations
 :)
declare namespace ess="https://exist.edirom.de";
declare namespace xhtml="http://www.w3.org/1999/xhtml";
declare namespace tei="http://www.tei-c.org/ns/1.0";

(:~
 : Return an HTML table of letters, a letter per row.
 : Calls ess:letter2tr#1 to process the individual letter
 : 
 : @param $letters the TEI letters to process
 : @return the table (xhtml:table) with one row for every letter
 :)
declare function ess:letters2table($letters as document-node()*) as element(xhtml:table) {
    <xhtml:table><xhtml:tbody>{
        $letters ! ess:letter2tr(.)
    }</xhtml:tbody></xhtml:table>
};

(:~
 : Return an HTML table row with information about a letter.
 : The cells provide the ID, the sender, the addressee and the date (in this sequence)
 : 
 : @param $letter the TEI letter to process
 : @return the table row (xhtml:tr)
 :)
declare function ess:letter2tr($letter as document-node()?) as element(xhtml:tr)? {
    let $id := $letter/tei:TEI/@xml:id => string()
    let $sender := ($letter//tei:correspAction[@type='sent']/tei:persName)[1] => normalize-space()
    let $addressee := ($letter//tei:correspAction[@type='received']/tei:persName)[1] => normalize-space()
    let $date := ($letter//tei:correspAction[@type='sent']/tei:date)[1] => normalize-space()
    return
        if($sender or $addressee or $date)
        then
            <xhtml:tr>
                <xhtml:td>{$id}</xhtml:td>
                <xhtml:td>{$sender}</xhtml:td>
                <xhtml:td>{$addressee}</xhtml:td>
                <xhtml:td>{$date}</xhtml:td>
            </xhtml:tr>
        else()
};

collection('/db/apps/WeGA-data/letters') => ess:letters2table()
```

## Eigene XQuery Funktionen

Eigene Funktionen werden mit `declare function` definiert. 
Die "Function-Signature" beschreibt darüber hinaus den Namen (mit namespace), 
die Parameter und einen (optionalen) Rückgabewert. 
Die Datentypen inkl. Quantifizierer der Parameter und des Rückgabewerts sind 
optional, es wird aber dringlichst empfohlen (von Peter), diese anzugeben.


## Map und arrow operator

* A mapping expression `S!E` evaluates the expression `E` once for every item 
  in the sequence obtained by evaluating `S`. 
  (<https://www.w3.org/TR/xquery-31/#id-map-operator>)
* An arrow operator applies a function to the value of an expression, using 
  the value as the first argument to the function.
  (<https://www.w3.org/TR/xquery-31/#id-arrow-operator>)


## XQuery Module

Funktionen lassen sich in XQuery Module auslagern und können dann von 
beliebigen XQueries (oder anderen Modulen) importiert und aufgerufen werden.
XQuery Module sind dabei nicht mehr selbst ausführbar, d.h. sie dürfen auch 
keine Ausgabe mehr haben, sondern stellen nur Funktionen und Variablen zur 
Verfügung.

Module werden durch die Angabe eines "Module Namespace" deklariert:
```xquery
module namespace ess="https://exist.edirom.de";
```

In einem anderen XQuery wird dieses Modul dann folgendermaßen eingebunden:
```xquery
import module namespace ess="https://exist.edirom.de" at "letters.xqm";
```

## Modulvariablen

Diese müssen – wie Funktionen – ein Namespaceprefix besitzen und mit einem 
Semikolon abgeschlossen werden. 
Die Angabe des Datentyps ist fakultativ.  

```xquery
declare variable $ess:year as xs:integer := 2024;
```


## XQuery Dokumentation

Es gibt dafür keinen "offiziellen" W3C-Standard, aber xqDoc füllt diese Lücke.
Kommentare werden mit `(:~` eingeleitet und mit `:)` ausgeleitet.
Es gibt feste tags wie `@author`, `@param` oder `@return`, siehe 
<https://xqdoc.org/xqdoc_comments_doc.html>.

eXist unterstützt diese Kommentare mit der App "XQuery Function 
Documentation", siehe <https://exist-db.org/exist/apps/fundocs/index.html>


## Auflistung aller Brieftitel

```xquery
xquery version "3.1";


(:
 : Namespace declarations
 :)
declare namespace ess="https://exist.edirom.de";
declare namespace xhtml="http://www.w3.org/1999/xhtml";
declare namespace tei="http://www.tei-c.org/ns/1.0";
declare namespace js="http://exist-db.org/xquery/javascript";


(:~
 : Returns an HTML list of the letter's titles.
 : Calls ess:letter2tr#1 to process the individual letter
 : 
 : @param $letters the TEI letters to process
 : @return the table (xhtml:table) with one row for every letter
 :)
 
declare function ess:letters2list($letters as document-node()*) as element(table) {
<html xmlns="http://www.w3.org/1999/xhtml">
    <head>
        <title>Listing</title>
        <link rel="stylesheet" href="tei.css" media="screen"/>

    </head>
    <body>
        <ul>
        {
            $letters ! ess:letter2list(.)
        }
        </ul>
    </body>
</html>};


(:~
 : Returns a HTML list with the title of the letter linking to the full letter
 :)
declare function ess:letter2list($letter as document-node()?) as element(li)? {
    let $id := $letter/tei:TEI/@xml:id => string()
    let $title := $letter//tei:title[@level='a'][1]
    let $string := string-join(
        for $node in $title/node()
    return if ($node/name() = 'lb') then ' ' else $node, ''
    )
    order by $title
    return
        
        if ($id != "" and $string != "")
        then
            
            <li xmlns="http://www.w3.org/1999/xhtml"><a href="/exist/apps/WeGA-data/tei2html.xq?{$id}">{$string}</a></li>
        else()
        
};

collection('/db/apps/WeGA-data/letters') => ess:letters2list() 
```

## Ausführen von XSLT-Stylesheets innerhalb eines XQuery

eXist stellt die Funktion transform zur Verfügung:
```xquery
transform:transform($node-tree as node()*, $stylesheet as item(), $parameters as node()?) as node()?
```



### Beispiel

```xquery
xquery version "3.1";

(:
 : Namespace declarations
 :)
declare namespace xhtml="http://www.w3.org/1999/xhtml";
declare namespace tei="http://www.tei-c.org/ns/1.0";
declare namespace output = "http://www.w3.org/2010/xslt-xquery-serialization";
declare namespace transform = "http://exist-db.org/xquery/transform";

(:
 : Serialization and output options
 :)
declare option output:media-type "text/html";
declare option output:method "xhtml";
declare option output:indent "yes";
declare option output:omit-xml-declaration "yes";


<html xmlns="http://www.w3.org/1999/xhtml">
    <head>
        <title>Test page</title>
        <link rel="stylesheet" href="tei.css" media="screen"/>
    </head>
    <body>
        {
            transform:transform(
                doc('/db/apps/WeGA-data/letters/A0416xx/A041627.xml'),
                doc('/db/apps/WeGA-data/tei2html.xsl'),
                ()
            )
        }
    </body>
</html>
```

### XSLT zur Transformation der Briefe
```xml
<?xml version="1.0" encoding="UTF-8"?>
<xsl:stylesheet xmlns:xsl="http://www.w3.org/1999/XSL/Transform"
    xmlns:xs="http://www.w3.org/2001/XMLSchema"
    xmlns:math="http://www.w3.org/2005/xpath-functions/math"
    xmlns:tei="http://www.tei-c.org/ns/1.0"
    xmlns="http://www.w3.org/1999/xhtml"
    exclude-result-prefixes="tei xs math"
    version="3.0">
    
    <xsl:template match="/">
        <div>
            <xsl:apply-templates select="tei:TEI/tei:text/tei:body/tei:div[@type='writingSession']" />
        </div>
    </xsl:template>
    
    <xsl:template match="tei:opener | tei:closer | tei:p">
        <p>
            <xsl:apply-templates/>
        </p>
    </xsl:template>
    
    <xsl:template match="tei:choice">
        <xsl:apply-templates select="tei:expan"/>
    </xsl:template>
    
    <xsl:template match="tei:abbr"/>
    
    <xsl:template match="tei:persName">
        <a href="{@ref}">
            <xsl:apply-templates/>
        </a>
    </xsl:template>
    

</xsl:stylesheet>
```

Some example XSL:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<xsl:stylesheet xmlns:xsl="http://www.w3.org/1999/XSL/Transform"
    xmlns:xs="http://www.w3.org/2001/XMLSchema"
    xmlns:xhtml="http://www.w3.org/1999/xhtml"
    xmlns:tei="http://www.tei-c.org/ns/1.0"
    exclude-result-prefixes="xs"
    version="2.0">
    
    <xsl:template match="tei:*">
        <xsl:element name="span" namespace="http://www.w3.org/1999/xhtml">
            <xsl:variable name="elemName" as="xs:string" select="'tei_' || local-name()"/>
            <xsl:variable name="attrs" as="xs:string*">
                <xsl:for-each select="@*">
                    <xsl:value-of select="local-name() || '_' || ."/>
                </xsl:for-each>
            </xsl:variable>
            <xsl:attribute name="class" select="($elemName, $attrs)"/>
            <xsl:apply-templates/>
        </xsl:element>
    </xsl:template>
    
</xsl:stylesheet>
```

The CSS could look like:

```css
.tei_teiHeader {
    display: none;
}

.tei_p {
    display: block;
    padding: .5em;
    text-align: justify;
}

.tei_closer {
    display: block;
    padding: .5em;
}

.tei_note {
    display: none;
}

.tei_settlement, .tei_persName {
    color: blue;
}
```

## Requests

eXist features a request module to capture various parameters from the HTTP 
request, such as headers and URL parameters.

```xquery
xquery version "3.1";

(:
 : Namespace declarations
 :)
declare namespace xhtml="http://www.w3.org/1999/xhtml";
declare namespace tei="http://www.tei-c.org/ns/1.0";
declare namespace output = "http://www.w3.org/2010/xslt-xquery-serialization";
declare namespace transform = "http://exist-db.org/xquery/transform";
declare namespace request = "http://exist-db.org/xquery/request";

(:
 : Serialization and output options
 :)
declare option output:media-type "text/html";
declare option output:method "xhtml";
declare option output:indent "yes";
declare option output:omit-xml-declaration "yes";

(:
 : Capture the parameter $id from the calling URL,
 : e.g. `http://localhost:8080/exist/apps/WeGA-data/tei2html.xq?id=A042323`
 : 
 : If no parameter is passed, the default (i.e. the second argument 
 : to `request:get-parameter#2`) will be taken
 :)
let $id := request:get-parameter("id", "A041627")
let $db-path := "/db/apps/WeGA-data/letters/" || replace($id, "..$", "xx/") || $id || ".xml"
return

<html xmlns="http://www.w3.org/1999/xhtml">
    <head>
        <title>Test page</title>
        <link rel="stylesheet" href="tei.css" media="screen"/>
    </head>
    <body>
        {
            transform:transform(
                doc($db-path),
                doc('/db/apps/WeGA-data/tei2html.xsl'),
                ()
            )
        }
    </body>
</html>
```

## Links

* [XQuery Wikibook](https://en.wikibooks.org/wiki/XQuery)
* Michael Kay, [Defining your own Functions in XQuery](http://www.stylusstudio.com/xquery/xquery-functions.html)
* [XQuery 3.1: An XML Query Language. W3C Recommendation 21 March 2017](https://www.w3.org/TR/xquery-31/) 
* xqDoc: <https://xqdoc.org/>
