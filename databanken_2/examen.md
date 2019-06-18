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

* Redundantie is het meer dan benodigd voorkomen van data, dit wilt zeggen dat data meerdere keren is opgeslagen.
* In RDBMS kunnen we data gaan uitnoemaliseren. Dit wilt zeggen dat we regels gaan implementeren zodat data maar 1 keer opgeslagen zal worden en dat de database duidelijk blijft.
