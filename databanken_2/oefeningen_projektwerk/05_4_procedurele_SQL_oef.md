# Procedurele sql

## Voorbeeld
```sql
--$1 $2 .. or named parameters
--' ' blok of $$ $$ blok of ..
CREATE OR REPLACE FUNCTION geef_klant(numeric)
RETURNS character varying AS
'
SELECT naam
FROM klanten
WHERE klantnr = $1;
'
LANGUAGE sql;
--standaard heb je alleen sql als taal
--if u need other languages, they need to be installed once
--this is database dependent (we have a cluster of databases..)
SELECT *
FROMpg_language;
--CREATE LANGUAGE 'plpgsql';
--random() is een functie die we kunnen gebruiken
SELECT *
FROMpg_proc
WHEREproname='random';
--mind the returns vs return thas has to be done
CREATE OR REPLACE FUNCTION random_bigint(lengte integer)
RETURNS bigint AS
$$
DECLARE woord bigint;
BEGIN
woord = round(random()*10^(lengte));
RETURN woord;
END
$$
LANGUAGE plpgsql;
--execute a function (stored procedure)
SELECT geef_klant(122);
SELECT random_bigint(10);
```


## Oefening
Schrijf een functie waarbij je gegevens uit een transactie tabel verwijdert die ouder zijn dan 10 dagen (of een andere “archiverings” functie op 1 van jou eigen tabellen).

## Oplossing
```sql
CREATE FUNCTION delete_after_10_days() returns void AS
    '
    delete
    from fordeletingthings.deleteafter10days
    where days > 10;
    '
LANGUAGE sql;

select delete_after_10_days();
```

## Oefening
Schrijf een functie die drie getallen optelt en teruggeeft.

## Oplossing



## Oefening
Geef 3 mogelijke procedurale talen op postgresql en vind een overeenkomst met een ander software pakket (dbsoftware) op minstens 1 van die procedurale talen.

## Oplossing



## Oefening
Zou het mogelijk zijn een aggregatie functie te schrijven?
Hoe zou je dit dan moeten doen en is dit sql standaard?

## Oplossing



## Oefening
Schrijf een functie die nuttig is voor jouw persoonlijke databank.

## Oplossing



## Oefening
Schrijf een functie met als input parameter een tabel, die als output een overzicht geeft van alle kolommen op de volgende wijze:
```sql
table A (x text, y int, z char(4)) -- tabel A met 3 kolommen als input
output: drie lijnen met een opsomming van de kolommen en hun datatypes.
x_value text;
y_value int;
z_value char;
-- tip: check table information about pg_attribute
```

## Oplossing



## Oefening
Schrijf een functie met dezelfde functionaliteit als een bestaande php functie uit je persoonlijk project.

## Oplossing



## Oefening
Probeer 1 van de volgende aan te maken (je mag kiezen welke, maar je mag het nog niet gedaan hebben eg: table) :

```sql
AGGREGATE, DATABASE, GROUP, OPERATOR, SEQUENCE, TRIGGER, USER, CAST, DOMAIN, INDEX, RULE, TABLE, TYPE, VIEW, CONVERSION, FUNCTION, LANGUAGE , SCHEMA, TEMP, UNIQUE
```

Controleer nu of deze functie of een SQL functie is en niet postgresql specifiek.

## Oplossing
