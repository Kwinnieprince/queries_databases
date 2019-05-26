# Oefeningen views, stored procedures en triggers

## Oefening 1

Zoals je misschien herinnerd, kan het tellen van al de rijen soms langs duren.
Dus het gebruik van de count() functie.

Maak op je lokale databank een tabel met minstens 1 miljoen rijen.

### Oplossing

```sql

```

## Oefening 2

Maak een view waarin de rijen van deze tabel telt. Hoe kan je ervoor zorgen dat deze view niet steeds opnieuw moet worden uitgevoerd, ie de onderliggende code.

Maak zo een view aan.

### Oplossing

```sql

```

## Oefening 3

Schrijf een trigger die bij een toevoeging of verwijdering van een rij in de tabel een teller in teltabel aanpast.

Het doel van deze teltabel is het aantal rijen van een andere tabel bij te houden.

### Oplossing

```sql

```

## Oefening 4

Welk van de 3 bovenstaande methode is het meest performant?

* rechstreeks tellen
* view (optie 1 en 2)
* teltabel (via trigger)

### Oplossing

```sql

```