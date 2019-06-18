## Mondeling

1. Waarvoor staat ACID?,
waarom wordt het gebruikt bij RMDBS?,
Wat is het nut ervan bij NoSQL databses?

    * ACID is een acroniem dat staat voor: Atomair, consistent, isolatie en duurzaam

        * Atomair &rarr; dit wilt zeggen dat elke transactie helemaal of niet wordt uigevoerd, nooit gedeeltelijk

        * Consistent &rarr; na her uitvoeren van transactie bevindt de server zich steeds in een consistente staat, dit wilt zeggen dat de data voldoet aan de gedefinieerde regels.

        * Isolatie &rarr; hiermee verwijst men naar de gekijktijdigheid (concurrency) van de transacties. Het is alsof dat de transacties volledig los van elkaar uitgevoerd worden

        * Duurzaam &rarr; als een transactie is vastgelegd, dan blijft deze, ookal heeft het systeem een probleem, dit wilt zeggen dat een transactie persistent is

    * waarom wordt het gebruikt bij RMDBS?

        * Atomair geeft *Rolback* en *commit*

        * Consistent geeft integriteitsregels &rarr; referentiële integriteit door gebruik van vreemde sleutels, unieke sleutels, andere beperkingen (constraints) en triggers

        * Isolatie geeft *set Transaction Isolation Level Serializable* &rarr; zorgt dat andere mensen data niet kunnen aanpassen terwijl er een transactie gebeurt

        * Duurzaam geeft dat transacties weggeschreven worden naar een harde schijf (duurzaam medium, non volatile)

    * De meeste NOSQL producten ondersteunen geen ACID maar wel BASE &rarr; een zwakkere vorm waarbij de nadruk ligt op snelle beschikbaarheid van de data al dan niet consistent

        * BASE &rarr; Basically Available, Soft state en Eventually consistent

## Oefeningen

2. Schrijf een veilig script dat de in_uitgave tabel wijzigt door een nieuwe kolom bedrag_met_teken toe te voegen en dat de desbetreffende info dan bevat. Daarnaast heeft het nieuwe beleid een exra view nodig.

    * Veilig: gebruik een transactie waarbij de andere gebruikers met zo weinig mogelijk wachtproblemen da databank kunnen blijven gebruiken. Dus met de hoogst mogelijke gelijktijdigheid.

    ```sql
    -- niet werkende query
    Create or replace function create_table() returns void
    as $$
        DECLARE
            idColumn INTEGER;
            waarde double precision;
    begin
        set search_path to "ExamenTest";
        ALTER TABLE	"testTriggers"
        add bedrag_met_teken varchar(10);
        select min( bedrag_met_teken ) into idColumn from "testTriggers";

        loop
            exit when idColumn is null;
            select min(in_uitgave) into waarde from "testTriggers";
            insert into bedrag_met_teken(bedrag_met_teken) values(waarde);
            select min( bedrag_met_teken ) into idColumn from "testTriggers" where bedrag_met_teken > idColumn;
        end loop;
    end
    $$
    language plpgsql
    ```

    ```sql
    -- Juiste query
    alter table "testTriggers"
    add column bedrag_met_teken varchar(10);
    create TRIGGER set_bedrag_met_teken after insert on "ExamenTest"."testTriggers"
    for each row
    execute procedure set_bedrag();
    create or replace function set_bedrag()
    returns trigger as
    $$
    begin
        set search_path to "ExamenTest";
        update "ExamenTest"."testTriggers"
        set bedrag_met_teken =  concat(in_uitgave + 5);
        return NEW;
    end;
    $$ language 'plpgsql';

    set search_path to "ExamenTest";
    insert into "testTriggers"(in_uitgave) values(5); 
    ```

    * Wat is probleem van deze kolom op deze manier toe te voegen?

    * Deze extra view bevat de informatie van alle eigenaars en daarnaast de niet gerelateerde informatie van al de types (van rekeningen). Ze vragen niet wie welk type heeft, maar gewoon beide in 1 view.

    ```sql
    create or replace view in_uit_bedragen as
        select *
        from "testTriggers";
    ```

3. Zorg ervoor dat alle uitgaven van Griet automatisch naar inkomsten worden omgezet wanneer Griet een nieuwe uitgave doet. Boodschappen:

    * Toon hierbij een gepaste boodschap "normal people dont understand this, but this is legal and correct, trust us, its really complicated to explain this; bug fixed" als er zich een nieuwe uitgave voordoet.

   ```sql
    -- Trigger for the query
    create TRIGGER set_uitgave after insert on "ExamenTest".in_uitgaves
    for each row when (NEW.inkomsten is null)
    execute procedure insert_uitgave();

    -- functie
    create or replace function insert_uitgave() returns trigger as 
    $$
        begin
            set search_path to "ExamenTest";
            raise notice 'normal people dont understand this, but this is legal and correct, trust us, its really complicated to explain this; bug fixed';
            update "ExamenTest".in_uitgaves
            set uitgaven = inkomsten;
            
            update "ExamenTest".in_uitgaves
            set uitgaven = 0;
            return NEW;
        end;
    $$
    language 'plpgsql'

    -- to test the trigger
    set search_path to "ExamenTest";
    insert into in_uitgaves(uitgaven, usernames) values(4, 'Griet');
    ```

    * Bij een nieuwe ingave/inkomst wordt de boodschap "are you sure" getoond.

    ```sql
    -- Function for the trigger later
    create or replace function insert_ingave() returns trigger as
    $$
        begin
            raise notice 'are you sure?';
            return new;
        end;
    $$
    language 'plpgsql'

    -- Trigger
    create trigger set_inkomst_message after insert on "ExamenTest".in_uitgaves for each row when (NEW.uitgaven is null) execute procedure insert_ingave()

    -- Data to test the trigger
    set search_path to "ExamenTest";
    insert into in_uitgaves(inkomsten, usernames) values(6, 'Griet');
    ```

## Theorie

4. Geef conceptueel de volgorde van de bewerkingen in de select instruktie met alle komponenten die we gezien hebben.

    * Geef aan of je akkoord bent of niet. En leg duidelijk en bondig uit waarom. De punten staan op de uitleg bij je keuze, niet op de keuze zelf. Je kan in je uitleg een voorbeel/afbeelding gebruiken. Stelling: Bij het uitnormaliseren moeten de vreemde sleutels altijd in de primaire sleutel zitten.

    ```sql
    neen,
    bij uitnormaliseren proberen we redundantie te voorkomen, het gebruik van foreign keys zorgt ervoor dat er enkel naar geldige waardes verwezen kan worden.
    dit maakt het uitnormaliseren eenvoudiger en overzichtelijker
    ```

    * Gegeven
    ```sql
    Gegeven: <Een CREATE statement en 3 querries>
	Elke querrie heeft dezelfde SELECT en FROM
	Q1:	WHERE id = 123456789;
	Q2:	WHERE passhash = 'erygufhvsbldjkn5bgsfd85bguyrhvnl'; (passhash is een charachter(32)
	Q3:	WHERE registered = <een tijdsmoment>; (het tijdsmoment is het tijdsmoment van de eerste registratie)
    ```
    We zijn op zoek naar de snelste querie. Welk van de volgende antwoorden is/zijn waar op de meeste RDBMSn?
    Elk antwoord is hier juist of fout.

        * Q1 waarschijnlijk niet want het id is zeer hoog, het staat waarschijnlijk fysisch niet in het begin van de tabel.
        * Q1 waarschijnlijk wel want het id is uniek. Er is er zo maar 1 rij.
        * Q2 waarschijnlijk niet passhash is wel een hash, maar daarom niet uniek, ook niet door een constraint in de tabel.
        * Q2 waarschijnlijk wel want door middel van hashes worden data sneller gevonden. Zoals in een hsh tabel of lijst.
        * Q3 waarschijnlijk niet want een tijdsmoment is een ingewikkeld samengesteld datatype.
        * Q3 waarschijnlijk wel want de eerste gebruiker werd toen geregistreerd, het staat waarschijnlijk fysisch in het begin van de tabel.

# voorbeeldexamen

1. Wat is redundantie? Hoe gaan we hiermee om in een RDBMS? Wat is de aangewezen manier van werken? Leg uit dat je duidelijk 'de kracht' van het juist gebruik hiervan begrijpt.

    >   * Redundantie is het meer dan benodigd voorkomen van data, dit wilt      zeggen dat data meerdere keren is opgeslagen.
    >  * In RDBMS kunnen we data gaan uitnoemaliseren. Dit wilt zeggen dat we regels gaan implementeren zodat data maar 1 keer opgeslagen zal worden en dat de database duidelijk blijft.

2. Wat is een cartesisch product? Wat moet je meestal doen om dit te vermijden?

    >   Een caresisch product of de productverzameling van 2 verzamelingen: de verzameling van alle koppels of geordende paren waarvan het eerste element uit de eerste verzameling en het tweede uit de tweede verzameling komt.
    >
    >   * Om dit te vermijden gebruiken we expliciete joins of aangeven als we excliciet een carthesis  product willen  met CROSS JOIN

3. Wat is een niet-gecorreleerde subquery?

    > Een niet-gecorreleerde query wordt exact 1 keer uitgevoerd en het resultaat kan gebruikt worden in de bovenliggende query

4. Wat is het verschil tussen een gecorreleerde subquery en een niet gecorreleerde subquery?

    > Een niet-gecorreleerde query wordt exact 1 keer uitgevoerd en het resultaat kan gebruikt worden in de bovenliggende query. Een gecorreleerde subquery wordt meerdere keren uitgevoerd en geeft telkens 1 rij terug aan de boveliggende query

5. Wat is het verschil tussen condities in de WHERE en de FROM

    > In de FROM component wordt geselecteerd welke tabellen er geraadpleegd moeten worden. In de WHERE component kunnen er regels op de data gesteld worden die getoond zal worden.

6. Geef een zinnig voorbeeld van een theta join en leg uit.

    > Algemene of theta join is de verzameling van equi- en non-equi-joins
    > ```sql
    > SELECT *
    > FROM spelers, teams
    > WHERE spelers.spelersnr ? teams.spelersnr

7. Welke set-operatoren ken je en wat doen ze (kort)?

    > UNION
    > * plakt rijen onder elkaar
    >>
    > INTERSECT
    > * Deze geeft alleen de resultaten weer die in beide tabelexpressies voorkomen &rarr; dit is ookwel de *doorsnede* genoemd
    >>
    > EXCEPT
    > * Deze geeft waarden terug die **wel** in de eerste tabelexpressie voorkomen maar **niet** in de tweede &rarr; dit wordt ookwel het verschil genoemd
    >>
    > UNION ALL
    > * Zelfde als union maar behoudt ook alle dubbele waarden
    >>
    >  INTERSECT ALL
    > * zelfde als intersect maar behoudt ook alle dubbele waarden
    > >
    > EXCEPT ALL
    > * Zelfde als except maar alle dubbels worden behouden

8. Waar dient LC_COLLATE voor en waarom kan dit belangrijk zijn?

    > er wordt gesorteerd eerst op uppercase en dan lowercase

9. Hoe kan je een constraint van de tabel verwijderen?

    > Met een alter table en dan drop constraint
    >
    > ```sql
    > ALTER TABLE <TABLE_NAME> 
    > DROP CONSTRAINT <FOREIGN_KEY_NAME>
    >```

10. Geef een alternatief voor de LIKE operator en geef enkele voorbeelden van wat dit alternatief meer kan.

    > ILIKE &rarr; dit kan regex en heeft geen invloed op hoofdletters. ILIKE is niet hoofdlettergevoelig

11. Leg uit: "connection pooling client side" met tekst en tekening.

12. Welke types van connecties kan je maken naar database vanuit een programmeertaal?

    > Een database connectie

13. Waarom kan het gebruik van een genereerde serial id gevaarlijk zijn?

    > Deze kan een getal zijn dat niet gewenst / gekend is door het bovenliggende programma.

14. Een querie met de volgende elementen:(S F W G H en 1 of meer subqueries).

15. Wat is een segmenttype?

16. Wat typeert een netwerk databank?

    > Dat deze een API heeft waarnaar gequeried kan worden

17. Geef een voorbeeld van een CTE met recursie.(zie slides)

18. Geef een alternatief met gecorreleerde subqueries om een beperkt deel van de resultaten te tonen (cf het limiteren van resultaten)

    > ```sql
    > SELECT spelersnr, naam, geboortedaatum
    > FROM spelers p
    > ORDER BY geboortedatum DESC
    > FETCH FIRST 3 ROWS ONLY;
    >```

19. Wat is het verschil tussen een string literal en een identifier?

    > Een string literal is een string die hetzelde is als een functiestring

20. Geef een alternatieve oplossing voor deze query (GROUP BY GROUPING SETS..)

21. Waarom maken we niet op elke kolom per definitie een INDEX?

    > Soms zijn indexen  overbodig als de kolom heel klein is of als de queries die uitgevoerd zullen worden meestal zonder index uitgevoerd worden. Dan is het maken van een index pure plaatsverspilling op de schijf van de databank

22. Stelling: "Het gebruik van JOINS is steeds beter dan het gebruik van subqueries". Kies een standpunt en verdedig helder.

23. Waarvoor kan je VIEWS gebruiken?

    > Het lijkt net alsof dat de gebruiker op een tabel aan het werken is maar dit is in werkelijkheid niet zo. Hierdoor is het makkelijk om bepaalde gegevens af te schermen en te beviligen

24. Wat is SQL injectie (injection) en hoe kan je dit vermijden?

    > Dat er SQL code uitgevoerd wordt op de server dat niet de bedoeling was. Je kan dit in programma's tegengaan door prepared statements te maken.

25. Wat zijn transacties?

    > Een transactie propageert 1 of meerdere veranderingen aan de database. Bijvoorbeeld het verwijderen of updaten van een kolom. Het is dus belangrijk dat de data niet verloren gaat of dat andere mensen deze op hetzelfde moment aanpassen. Door een transactie wordt een *lock* geplaatst waardoor niemand anders de data kan aanpassen op het moment dat de transactie uitgevoerd wordt

26. Geef een voorbeeld van embedded sql.

    >```c++
    >int main() {  
    >  EXEC SQL INCLUDE SQLCA;  
    > EXEC SQL BEGIN DECLARE SECTION;  
    >  int OrderID;         /* Employee ID (from user)         */  
    >  int CustID;            /* Retrieved customer ID         */  
    >  char SalesPerson[10]   /* Retrieved salesperson name      */  
    >  char Status[6]         /* Retrieved order status        */  
    >EXEC SQL END DECLARE SECTION;  
    >
    > /* Set up error processing */  
    >EXEC SQL WHENEVER SQLERROR GOTO query_error;  
    >EXEC SQL WHENEVER NOT FOUND GOTO bad_number;  
    >
    >/* Prompt the user for order number */  
    >printf ("Enter order number: ");  
    >scanf_s("%d", &OrderID);  
    >
    >/* Execute the SQL query */  
    >EXEC SQL SELECT CustID, SalesPerson, Status  
    >  FROM Orders  
    >  WHERE OrderID = :OrderID  
    >  INTO :CustID, :SalesPerson, :Status;  
    >
    >/* Display the results */  
    >printf ("Customer number:  %d\n", CustID);  
    >printf ("Salesperson: %s\n", SalesPerson);  
    >printf ("Status: %s\n", Status);  
    >exit();  
    >
    >query_error:  
    >printf ("SQL error: %ld\n", sqlca->sqlcode);  
    >exit();  
    >
    >bad_number:  
    >printf ("Invalid order number.\n");  
    >exit();  
    >}
    >```

27. Waarvoor kunnen we TRIGGERS gebruiken?

    > Om integriteitsregels te controleren of informatie in de database aan te passen als er iets geüpdate of ge-insert wordt

28. Geef de typische kenmerken van een ODMS.

29. Wat zijn RULES? Geef een voorbeeld.

30. Stelling: "We hebben geen databanken nodig, we kunnen alles met XML."Kies een standpunt en verdedig helder.

31. Geef een voorbeeld van een NoSQL product en filosofie.

    > MongoDB &rarr; NoSQL is makkelijk te schalen en leveren betere performantie dan traditionele relationele databases

32. Stelling:"PostgreSQL is de relationele database in alle omstandigheden, we kunne ze voor alles gebruiken" Kies een standpunt en verdedig helder.

33. Wat is het verschil tussen een venster en een groepering?

    > Groepering &rarr; doet de ordering in de groep
    > Venster &rarr; de group by functie wordt uitgevoerd voor de vensterfunctie

34. Geef een voorbeeld van een cumulatieve frequentietabel met een theta join enerzijds en een venster anderzijds. Welke is het meest performant, hoe kan je dit controleren?
