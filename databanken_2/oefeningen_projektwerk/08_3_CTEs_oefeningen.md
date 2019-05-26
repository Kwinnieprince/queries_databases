# Oefeningen Common Table Expressions

## Oefening 1

Kijk naar de structuur van de tabel hemelobjecten in het schema public en naar dezelfde tabel in het schema ruimtereizen. 
Wat merk je op?

### Oplossing 

```sql

```

## Oefening 2

Geef de gepaste code waarmee je de tabel hemelobjecten in ruimtereizen 'beter' kan maken.

### Oplossing 

```sql

```

## Oefening 3

Er zijn twee (of meer..) manieren om deze tabel 'beter' te maken, met hetzelfde eindresultaat.
Welke van deze twee manieren zou jij kiezen en waarom?

### Oplossing

```sql

```

## Oefeningen in schema ruimtereizen met eventuele transacties

### Oefening 1

Voeg 1 rij toe aan een tabel naar keuze uit dit schema.

#### Oplossing 

```sql

```

### Oefening 2

Verander 1 rij in een tabel naar keuze uit dit schema.

#### Oplossing

```sql

```

### Oefening 3

Verwijder 1 rij uit een tabel naar keuze uit dit schema.

#### Oplossing

```sql

```

### Oefening 4

Schrijf een querie die alle satellieten van een zelf gekozen hemelobject toont. (bv de zon indien aanwezig in de tabel).
Zorg ervoor dat de hele hierarchie wordt getoond. (cf CTEs)

#### Oplossing

```sql 

```

### Oefening 5

Toon de boomstructuur voor elk hemelobject uit de vorige querie.

```sql
-- Voorbeeld op het schema ruimtereizen, alle satellieten van de zon en dieper:
(order by 2,1)
"Maan";"Aarde-Zon"
"Adrastea";"Jupiter-Zon"
..
"Venus";"Zon"
-- (70 rijen)
```

#### Oplossing
```sql

```

### Oefening 6

Probeer een stapsgewijze query naar te keuze te 'ontvlooien' gebruik makend van een tijdelijke tabel waar je tussenresultaten naar toe schrijft.
Maak hiervoor gebruik van CTEs.

#### Oplossing

```sql

```

### Oefening 7

**Werk binnen een transactie die je bij voorkeur niet bestendigd maar ongedaan maakt.**

* Voer eerste de querie oplossing van oefening 4.4 of 4.5 uit
* Verwijder nu alle afhankelijke hemelobjecten (in het voorbeeld van deze tekst is dat de zon; dit is afhankelijk van jouw vorige queries, namelijk van het wortel-element dat je hebt gekozen) van het wortel-element uit de vorige querie. Doe dit in 1 statement.
* Begrijp je wat onderstaande code betekent?

    ```sql 
    CONSTRAINT bezoeken_objectnaam_fkey FOREIGN KEY (objectnaam)
    REFERENCES ruimtereizen.hemelobjecten (objectnaam) MATCH SIMPLE
    ON UPDATE NO ACTION ON DELETE NO ACTION
    ```

* Voeg eventueel hemelobjecten toe die niet .. (vul aan)
* Keer terug op je stappen of (niet)

### Oefening 8 

Je kan zelf experimenteren zelf met het ruimtereizen of tennis script in probeer