# Sustav-za-upravljanje-skladistem
projekt baze podataka 2
## Ivan Maurović

## ER dijagram

<img src="https://cdn.discordapp.com/attachments/1190245422741061764/1196412358332010557/ERD_baze_2.png"/>

**EER dijagram**

<img src="https://cdn.discordapp.com/attachments/1164449330988580864/1196141423733510196/EER_diagram_-_skladiste.png"/>

**Opis dijagrama**

Skup entiteta **firma** povezan je s skupom entiteta **adresa** više naprema jedan, jer možemo imati da na jednoj **adresi** bude više **firma**, dok svaka **firma** može biti stacionirat na samo jednoj **adresi**.

Skup entiteta **adresa** povezan je s skupom entiteta **država** više naprema jedan, jer možemo imati da u jednoj **državi** se nalazimo na više **adresa**, dok **adresa** može biti samo u jednoj **državi**.

Skup entiteta **račun** povezan je s skupom entiteta **firma** više naprema jedan, jer **firma**  može izdati  više **računa**, dok **račun** može biti izdan samo u jednoj **firmi**.

Skup entiteta **račun** povezan je s skupom entiteta **zaposlenik** više naprema jedan, jer možemo imati da  **zaposlenik** izdaje više **računa**, dok jedan **račun** može biti izdati samo od jednog **zaposlenika**.

Skup entiteta **zaposlenik** povezan je s skupom entiteta **skladište** više naprema jedan, jer možemo imati da u jednom **skladištu** radi više **zaposlenika**, dok **zaposlenik** može raditi samo u jednom **skladištu**.

Skup entiteta **zaposlenik** povezan je s skupom entiteta **vrsta_posla** više naprema jedan, jer jednu **vrsta_posla** može raditi više **zaposlenika**, dok **zaposlenik** može raditi najviše jednu **vrstu_posla**.

Skup entiteta **izdatnica** povezan je s skupom entiteta **zaposlenik** više naprema jedan, jer možemo imati da jedan **zaposlenik** izdaje više **izdatnica**, dok **izdatnica** može biti izdana od samo jednog **zaposlnika**.

Skup entiteta **izdatnica** povezan je s skupom entiteta **kupac** više naprema jedan, jer možemo imati da jedan **kupac** kreira više **izdatnica**, dok **izdatnica** može biti kreirana od samo u jednog **kupca**.

Skup entiteta **kupac** povezan je s skupom entiteta **adresa** više naprema jedan, jer možemo imati da na jednoj **adresi** se nalazi više **kupaca**, dok **kupac** može biti samo prijavljen na jednoj **adresi**.

Skup entiteta **povratno** povezan je s skupom entiteta **izdatnica** više naprema jedan, jer na jednoj **izdatnici** možemo imati više **povrata**, dok **povrat* može biti najviše na jednoj **izdatnici**.

Skup entiteta **povratno** povezan je s skupom entiteta **dobavljac** više naprema jedan, jer možemo imati da jedan **dobavljac** ima više **povrata**, dok **povrat** može biti samo kod jednog **dobavljača**.

Skup entiteta **skladište** povezan je s skupom entiteta **firma** jedan naprema više, jer možemo imati da jedna **firma** posjeduje više **skladišta**, dok **skladište** može biti u posjedu najviše jedne **firme**.

Skup entiteta **skladište** povezan je s skupom entiteta **adresa** više naprema jedan, jer možemo na jednoj **adresi** posjedovati više **skladišta**, dok **skladište** može biti najviše na jednoj **adresi**.

Veza s atributom **artikli_u_skladištu** povezan je s skupom entiteta **skladište** više naprema više, jer **skladište** može sadržati više vrsta **artikala_u_skladištu**, dok **artikl_u_skladištu** može biti u više **skladišta** ne treba biti samo u jednom.

Veza s atributom **artikli_u_skldaištu** povezan je s skupom entiteta **artikl** više naprema više, jer možemo imati jednu vrstu **artikla** u više **artikla_u_skladišta**, dok **artikl_u_skladištu** može biti više **artikla**.

Skup entiteta **artikla** povezan je s skupom entiteta **racun** više naprema jedan, jer možemo imati na jednom **racunu**  više **artikala**, dok **artikl** može biti samo na jednom **racunu** Iz ovog odnosa više na jedan dolazimo do stvaranja nove tablice **stavka_racun**. Iz ovog odnosa možemo izvući količinu pojedinog **artikla** na svakom **racunu**.

Veza s atributom **stavka_izdatnice** povezan je s skupom entiteta **izdatnica** više naprema jedan, jer **izdatnica** može imati više **stavaka_izdatnice**, dok **stavka_izdatnice** može biti najviše na jednoj **izdatnica**.

Veza s atributom **stavka_izdatnice** povezan je s veza s atributom **artikli_u_skladištu** jedan naprema jedan, jer možemo imati jednu vrstu  **artikla_u_skladistu** na jednoj **stavci_izdatnice**, dok **stavka_izdatnice** može sadržati najviše jedan **artikl_u_skladištu**.


## Okidaći

**1. Ažuriranje količine artikala u skladištu nakon izdavanja**

Nakon novoga izdavanja dolazi do ažuriranja podataka u tablici artikli_u_skladištu korištenjem UPDATE argumenta te dolazi do oduzima broja koliko je izdano od broja artikala koji su u skladištu.

```mysql
DROP TRIGGER azuriranje_kolicine_artikla_nakon_izdavanja;
DELIMITER //
CREATE TRIGGER azuriranje_kolicine_artikla_nakon_izdavanja
AFTER INSERT ON stavka_izdatnica
FOR EACH ROW
BEGIN
    UPDATE artikli_u_skladistu
    SET kolicina = kolicina - NEW.kolicina
    WHERE ID = NEW.ID_artikli_u_skladistu;
END //
DELIMITER ;
INSERT INTO stavka_izdatnica VALUES (17,5,1,3 );
SELECT * FROM stavka_izdatnica;
SELECT * FROM artikli_u_skladistu;
```

**2. Prilikom postavljanja novih računa postavi datum izdavanja na trenutni datum**

Prilikom unosa novoga računa naredbom INSERT automatski se datum izdavanja računa  postavlja na trenutni datum (NOW()).

```mysql
DROP TRIGGER dv_racun;
 DELIMITER //
CREATE TRIGGER dv_racun
 BEFORE INSERT ON racun
 FOR EACH ROW
BEGIN
 SET new.datum_izdavanja = NOW();
END//
DELIMITER ;

INSERT INTO racun (ID, ID_zaposlenik, datum_izdavanja, ID_firma) VALUES (34, 12, CURRENT_DATE, 1);
Select * From racun;
```
**3. Zabranjuje da količina novoga artikla u skladištu bude manja od 1 te javlja da količina mora biti 1 ili vise**

Kada želimo dodavati novi artikl u skladište naredbom INSERT ako nam količina toga artikla nije 1 ili više dolazi do zabrane unosa tog artikla te nam javlja grešku 'Količina mora biti 1 ili više', a ako je količina 1 ili više onda dozvoljava unos novoga artikla ü skladište.

```mysql
DROP TRIGGER negativna_kolicina;
DELIMITER //
CREATE TRIGGER negativna_kolicina
BEFORE INSERT ON artikli_u_skladistu
FOR EACH ROW
BEGIN
    IF NEW.kolicina < 1 THEN
        SIGNAL SQLSTATE '40000'
        SET MESSAGE_TEXT = 'Količina mora biti 1 ili više';
    END IF;
END //
DELIMITER ;

INSERT INTO artikli_u_skladistu (ID, ID_skladiste, ID_artikl, kolicina) VALUES (21, 1, 1, 0);
INSERT INTO artikli_u_skladistu (ID, ID_skladiste, ID_artikl, kolicina) VALUES (22, 1, 1, 1);
Select* from artikli_u_skladistu;
```

**4. Prije unosa novog računa provjerava se ID zaposlenika ako ID ne postaji izlazi greška 'Zaposlenik ne postoji'**

Kod unosa novoga računa naredbom INSERT dolazi do provjeri ID_zaposlenika  te ako taj ID ne postoji onda nije moguć unos tog računa te izbacuje se greška 'Zaposlenik ne postoji' , a ako zaposlenik postoji novi se račun unosi.

```mysql
DROP TRIGGER prije_unosa_racuna;
DELIMITER //
CREATE TRIGGER prije_unosa_racuna
BEFORE INSERT ON racun
FOR EACH ROW
BEGIN
    DECLARE zaposlenik_id INT;

    SELECT COUNT(*) INTO zaposlenik_id
    FROM zaposlenik
    WHERE ID = NEW.ID_zaposlenik;

    IF zaposlenik_id = 0 THEN
        SIGNAL SQLSTATE '40000'
        SET MESSAGE_TEXT = 'Zaposlenik ne postoji';
    END IF;
END//
DELIMITER ;

INSERT INTO racun (ID, ID_zaposlenik, datum_izdavanja, ID_firma) VALUES (NULL, 100, STR_TO_DATE('01.01.2024.', '%d.%m.%Y.'), 1);
```

**5. Prikaz ukupne cijene računa u dodatnoj koloni ukupna_cijena**

Prilikom unosa novih stavaka računa izračunava se ukupna cijena umnoškom cijene i količine iz tablice artikl te se onda ukupna cijena ažurira u tablici račun. U tablici račun dodan je atribut ukupna_cijena tipa DECIMAL (10,2) pomoću ALTER TABLE račun tako da kada želimo prikazati tablicu račun imamo dodatni stupac u kojemu piše ukupna_cijena.

```mysql
DROP TRIGGER azuriranje_uk_cijene_racuna;
DELIMITER //
CREATE TRIGGER azuriranje_uk_cijene_racuna
BEFORE INSERT ON stavka_racun
FOR EACH ROW
BEGIN
    DECLARE total DECIMAL(10, 2);

    SELECT SUM(a.cijena * NEW.kolicina)
    INTO total
    FROM artikl a
    WHERE a.ID = NEW.ID_artikl;

    UPDATE racun
    SET ukupna_cijena = total
    WHERE ID = NEW.ID_racun;
END //
DELIMITER ;

ALTER TABLE racun ADD COLUMN ukupna_cijena DECIMAL(10, 2);
INSERT INTO racun (ID, ID_zaposlenik, datum_izdavanja, ID_firma)
VALUES (11, 1, CURRENT_DATE, 1);

INSERT INTO artikl (ID, cijena, naziv, vrsta_artikla)
VALUES (18, 299.99, 'Pisaći stol', 'Namještaj');

INSERT INTO stavka_racun (ID, ID_racun, ID_artikl, kolicina)
VALUES (11, 11, 18, 3);
SELECT * FROM racun;
```
## Toni Mijić
## Tablice

**Relacija drzava**:
sve drzave koje se koriste za skladiste

- id podataka tipa integer,primarni kljuc
- skracenica podatak tipa char limitiran na 3 znaka
- naziv podatak tipa varchar limitiran na 20 znakova
- glavni_grad podatak tipa varchar limitiran na 30 znakova

```mysql
CREATE TABLE drzava(
id INTEGER NOT NULL,
skracenica CHAR(3) NOT NULL,
naziv VARCHAR(20) NOT NULL,
glavni_grad VARCHAR(30) NOT NULL,
PRIMARY KEY(id)
);
Drop table drzava;
```

**Relacija adresa**:
prikazuje adrese koje se koriste za skladiste

- primarni kljuc je id tipa integer
- id_drzava tipa integer referencira se na drzava(id) kao FOREIGN KEY
- naziv,grad su tipa varchar
- postanski_broj tipa integer i ima check koji provjerava da broj ne ide u nedogled vec do 100 000

```mysql
CREATE TABLE adresa(
ID INTEGER NOT NULL,
naziv VARCHAR(50) NOT NULL,
id_drzava INTEGER NOT NULL, 
grad VARCHAR(30) NOT NULL,
postanski_broj INT NOT NULL CHECK (postanski_broj<100000), 
PRIMARY KEY (ID),
FOREIGN KEY (id_drzava) REFERENCES drzava(id)
);
Drop table adresa;
```

**Relacija vrsta_posla**:
Vrste posla u skladistima

- primarni kljuc je id tipa integer
- naziv,duznost,radno_vrijeme ,opis su sve tipa varchar

```mysql
CREATE TABLE vrsta_posla(
ID INTEGER NOT NULL,
naziv VARCHAR(20) NOT NULL,
duznost VARCHAR(50) NOT NULL,
radno_vrijeme VARCHAR(50) NOT NULL,
opis VARCHAR(100) NOT NULL,
PRIMARY KEY (ID)
);
Drop table vrsta_posla;
```

**Relacija firma**:
sve firme koje rade sa skladistem tj. dostavljaju u skladiste

- primarni kljuc je id tipa integer
- oib je tipa char limitiran na 11 te smo napravili check koji provjerava da li je duzina oiba 11 
- id_adresa tipa integer referencira se na adresa(id) kao FOREIGN KEY

```mysql
CREATE TABLE firma(
ID INTEGER NOT NULL,
naziv VARCHAR(50) NOT NULL,
OIB CHAR(11) NOT NULL UNIQUE, 
ID_adresa INT, 
PRIMARY KEY (ID),
check (length(oib) = 11),
FOREIGN KEY  (id_adresa) REFERENCES adresa (id)
);
Drop table firma;
```
**Relacija skladiste**:
lokacija i detalji svakog zasebnog skladista

- id_firma tipa integer referencira se na firma(id) kao FOREIGN KEY
- id_adresa tipa integer referencira se na adresa(id) kao FOREIGN KEY
- primarni kljuc je id tipa integer
- naziv je tipa varchar


```mysql
CREATE TABLE skladiste(
ID INTEGER NOT NULL,
ID_firma INT NOT NULL, 
naziv VARCHAR(20) NOT NULL,
ID_adresa INT NOT NULL, 
PRIMARY KEY (ID),
FOREIGN KEY (id_firma) REFERENCES firma(id),
FOREIGN KEY  (id_adresa) REFERENCES adresa (id)
);
Drop table skladiste;
```

**Relacija zaposlenik**:
prikazuje detalje o zaposlenicima

- primarni kljuc je id tipa integer
- oib je unique i limitiran na 11 znakova te ima isti check da provjerava da je duzina oiba 11 
- datum_zaposlenja je tipa datum
- id_skladiste tipa integer referencira se na skladiste(id) kao FOREIGN KEY
- id_vrsta_posla tipa integer referencira se na vrsta_posla(id) kao FOREIGN KEY

```mysql
CREATE TABLE zaposlenik(
ID INTEGER NOT NULL,
ime VARCHAR(30),
prezime VARCHAR(30),
OIB CHAR(11) UNIQUE NOT NULL,
datum_zaposlenja DATE NOT NULL,
ID_skladiste INT ,
ID_vrsta_posla INT, 
PRIMARY KEY (ID),
check (length(oib) = 11),
FOREIGN KEY (id_skladiste) REFERENCES  skladiste(id),
FOREIGN KEY (id_vrsta_posla) REFERENCES vrsta_posla(id)
);
Drop table zaposlenik;
```

**Relacija racun**:
prikaz racuna izdanih sa odredenom firmom

- primarni kljuc je id tipa integer
- id_zaposlenik tipa integer referencira se na zaposlenik(id) kao FOREIGN KEY
- id_firma tipa integer referencira se na firma(id) kao FOREIGN KEY

```mysql
CREATE TABLE racun(
ID INTEGER NOT NULL,
ID_zaposlenik INT , 
datum_izdavanja DATE NOT NULL, 
ID_firma INT, 
PRIMARY KEY (ID),
FOREIGN KEY (id_zaposlenik) REFERENCES zaposlenik(id),
FOREIGN KEY (id_firma) REFERENCES firma(id)
);
Drop table racun;
```

**Relacija artikl**:
detalji artikala

- primarni kljuc je id tipa integer
- cijena je decimal i ima check koji provjerava da cijena ne prelazi 5000 i da ne ide ispod nule

```mysql
CREATE TABLE artikl(
ID INTEGER NOT NULL,
cijena DECIMAL(10,2) NOT NULL, 
naziv VARCHAR(20) NOT NULL,
vrsta_artikla VARCHAR(30) NOT NULL, 
PRIMARY KEY (ID),
check (cijena < 5000 AND cijena >= 0 )
);
Drop table artikl;
```
**Relacija stavka_racun**:
prikaz artikala koji su bili na racun

- primarni kljuc je id tipa integer
- id_racun tipa integer referencira se na racun(id) kao FOREIGN KEY
- id_artikl tipa integer referencira se na artikl(id) kao FOREIGN KEY
- kolicina tipa integer i ima check koji provjerava da kolicina ne prelazi 50 i da ne ide ispod nule

 ```mysql
CREATE TABLE stavka_racun(
ID INTEGER NOT NULL,
ID_racun INT , 
ID_artikl INT , 
kolicina INT NOT NULL CHECK (kolicina < 50 AND kolicina >= 0), 
PRIMARY KEY (ID),
FOREIGN KEY (id_racun) REFERENCES racun(id) ON DELETE CASCADE,
FOREIGN KEY (id_artikl) REFERENCES artikl(id)
);
Drop table stavka_racun;
```

**Relacija artikli_u_skladistu**:
svi artikli koji dolaze se nalaze na ovom popisu

- primarni kljuc je id tipa integer
- id_skladiste tipa integer referencira se na skladiste(id) kao FOREIGN KEY
- id_artikl tipa integer referencira se na artikl(id) kao FOREIGN KEY
- kolicina tipa integer i ima check koji provjerava da kolicina ne prelazi 1000 i da ne ide ispod nule

 ```mysql
CREATE TABLE artikli_u_skladistu(
ID INTEGER NOT NULL,
ID_skladiste INT , 
ID_artikl INT ,
kolicina INT CHECK (kolicina < 1000 AND kolicina >= 0), 
PRIMARY KEY (ID),
FOREIGN KEY (id_skladiste) REFERENCES skladiste(id),
FOREIGN KEY (id_artikl) REFERENCES artikl(id)
);
Drop table artikli_u_skladistu;
 ```

**Relacija kupac**:
detalji o kupcu

- ime,prezime,telefon tipa varchar
- primarni kljuc je id tipa integer
- id_adresa tipa integer referencira se na adresa(id) kao FOREIGN KEY

 ```mysql
CREATE TABLE kupac(
ID INTEGER NOT NULL,
ime VARCHAR(40),
prezime VARCHAR(40),
ID_adresa INTEGER,
telefon VARCHAR(20),
PRIMARY KEY (ID),
FOREIGN KEY (id_adresa) REFERENCES adresa(id)
);
Drop table kupac;
 ```
**Relacija izdatnica**:
racun koji prikazuje kupca i sta je kupljeno

- primarni kljuc je id tipa integer
- datum i vrijeme dostavljanja su tipa date time
- id_zaposlenik tipa integer referencira se na zaposlenik(id) kao FOREIGN KEY
- id_kupac tipa integer referencira se na kupac(id) kao FOREIGN KEY

 ```mysql
CREATE TABLE izdatnica (
ID INTEGER NOT NULL,
datum_dostavljanja DATE NOT NULL,
vrijeme_dostavljanja TIME NOT NULL,
ID_zaposlenik INT ,
ID_kupac INT , 
PRIMARY KEY (ID),
FOREIGN KEY (id_zaposlenik) REFERENCES zaposlenik(id),
FOREIGN KEY (id_kupac) REFERENCES kupac(id)
);
Drop table izdatnica;
 ```
**Relacija stavka_izdatnica**:
artikli koji se nalaze na izdatnici

- primarni kljuc je id tipa integer
- id_izdatnica tipa integer referencira se na izdatnica(id) kao FOREIGN KEY
- id_artikli_u_skladistu tipa integer referencira se na artikli_u_skladistu(id) kao FOREIGN KEY
- kolicina tipa integer i ima check koji provjerava da kolicina ne prelazi 25 i da ne ide ispod nule

```mysql
CREATE TABLE stavka_izdatnica(
ID INTEGER NOT NULL,
ID_izdatnica INTEGER ,
ID_artikli_u_skladistu INTEGER, 
kolicina INTEGER CHECK (kolicina < 25 AND kolicina >= 0), 
PRIMARY KEY (ID),
FOREIGN KEY (id_izdatnica) REFERENCES izdatnica(id),
FOREIGN KEY (id_artikli_u_skladistu) REFERENCES artikli_u_skladistu(id)
);
Drop table stavka_izdatnica
```

**Relacija dobavljac**:
bavi se izdatnicom

- primarni kljuc je id tipa integer
- id_zaposlenik tipa integer referencira se na zaposlenik(id) kao FOREIGN KEY
- godine iskustva tipa integer i ima check koji provjerava da nema vise od 25 godina iskustva

```mysql
CREATE TABLE dobavljac(
ID INTEGER NOT NULL,
ID_zaposlenik INT,
vozacka VARCHAR(5),
godine_iskustva INTEGER CHECK (godine_iskustva < 25 ),
PRIMARY KEY (ID),
FOREIGN KEY (id_zaposlenik) REFERENCES zaposlenik(id)
);
Drop table dobavljac;
```
**Relacija povratno**:
sve izdatnice koje nisu iz prve dostavljene

- primarni kljuc je id tipa integer
- id_dobavljac tipa integer referencira se na dobavljac(id) kao FOREIGN KEY
- id_izdatnica tipa integer referencira se na izdatnica(id) kao FOREIGN KEY

```mysql
CREATE TABLE povratno(
ID INTEGER NOT NULL,
id_dobavljac INT,
id_izdatnica INT,
razlog_povratka VARCHAR(50),
datum_ponovnog_dostavljanja DATE,
PRIMARY KEY(ID),
FOREIGN KEY (id_dobavljac) REFERENCES dobavljac(id),
FOREIGN KEY (id_izdatnica) REFERENCES izdatnica(id)
);
Drop table povratno;
```

### PROCEDURE

##### Procedura 1

 Prva procedura je jednostavni primjer koja vraca firmu s kojom se najvise posluje do sada i ima jedan izlazni parametar
 procedura nema nijedan ulazni parametar

```mysql
DELIMITER //

CREATE PROCEDURE najvaznija_firma(OUT najvise_poslujuca_firma VARCHAR(50))
BEGIN
    SELECT firma.naziv
    INTO najvise_poslujuca_firma
    FROM firma
    JOIN racun ON firma.id = racun.id_firma
    GROUP BY firma.naziv
    ORDER BY COUNT(racun.id) DESC
    LIMIT 1;
END //

DELIMITER ;

CALL najvaznija_firma(@rezultat);
SELECT @rezultat AS 'najvaznija_firma';
```

##### Procedura 2

Druga procedura je jednostavni  prikaz skladista s najvecom kolicinom artikala tako da bismo znali koje skladiste treba izbjegavati puniti artiklima ako je potrebno
nema ulaznih ni izlaznih parametara

```mysql
DELIMITER //

CREATE PROCEDURE prikaz_najpunijeg_skladista()
BEGIN

    DECLARE skl_naziv VARCHAR(50);


    SELECT naziv INTO skl_naziv
    FROM (
        SELECT s.naziv, SUM(sa.kolicina) AS sveukupna_kolicina
        FROM skladiste s
        JOIN artikli_u_skladistu sa ON s.id = sa.id_skladiste
        GROUP BY s.id
        ORDER BY sveukupna_kolicina DESC
        LIMIT 1
    ) AS skladista;

    SELECT skl_naziv AS naziv_skladista;
END //

DELIMITER ;

CALL prikaz_najpunijeg_skladista();
```

##### Procedura 3


Treca prcoedura pronalazi najvrijednijeg radnika i dodjeljuje mu vip ulogu ima jedna ulazno/izlazni parametar 
Koristimo 2 pomocne varijable u koje cemo spremiti zaposlenika s najvise racuna te u drugu spremamo vip naziv

```mysql
DELIMITER //
CREATE PROCEDURE dodaj_nagradu_zaposleniku(INOUT vip_id INT)
BEGIN
    DECLARE najaktivniji_id INT;
    DECLARE vip_ime VARCHAR(60);

    SELECT id_zaposlenik INTO najaktivniji_id
    FROM (
        SELECT id_zaposlenik, COUNT(*) AS broj_racuna
        FROM racun
        WHERE YEAR(datum_izdavanja) = YEAR(CURRENT_DATE - INTERVAL 1 YEAR)
        GROUP BY id_zaposlenik
        ORDER BY broj_racuna DESC
        LIMIT 1
    ) AS najaktivniji;

    SET vip_id = najaktivniji_id;

    SELECT CONCAT('VIP ', ime) INTO vip_ime
    FROM zaposlenik
    WHERE ID = vip_id;

    UPDATE zaposlenik
    SET ime = vip_ime
    WHERE ID = vip_id;
END //
DELIMITER ;

SET @zaposlenik_id = 0;
CALL dodaj_nagradu_zaposleniku(@zaposlenik_id);
SELECT @zaposlenik_id;
```

##### Procedura 4

Cetvrta procedura je jednostavna procedura koja nam provjerava da li je odredeni artikl u odredenom skladistu postojeci i koja je kolicina
ima 2 ulazna i 3 izlazna parametra 

```mysql
DELIMITER //
CREATE PROCEDURE Provjera_artikala_u_skladistu(IN skladiste_id INT, IN artikl_id INT,OUT naziv_artikla VARCHAR(50), OUT artiklPostoji INT, OUT broj_artikala INT)
BEGIN

    SELECT a.naziv, aus.kolicina INTO naziv_artikla, broj_artikala
    FROM artikl a
    LEFT JOIN artikli_u_skladistu AS aus ON a.id = aus.id_artikl
    WHERE aus.id_skladiste = skladiste_id AND aus.id_artikl = artikl_id;


    IF broj_artikala > 0 THEN
        SET artiklPostoji = 1;
    ELSE
        SET artiklPostoji = 0 ;
    END IF;
END //

DELIMITER ;

CALL Provjera_artikala_u_skladistu(1, 10,@naziv_artikala ,@artiklPostoji, @broj_artikala);
SELECT @naziv_artikala AS NazivArtikla ,@artiklPostoji AS ArtiklPostoji,  @broj_artikala AS Kolicina;
```


##### Procedura 5 

Peta procedura je jednostavna procedura koja provjerava koje se skladiste najcesce koristi da vidimo u koje treba najbolja kvaliteta ljudi radi najvise posla
nema ulaznih ni izlaznih parametara

```mysql
DROP PROCEDURE IF EXISTS NajcesceKoristenoSkladiste;

DELIMITER //

CREATE PROCEDURE NajcesceKoristenoSkladiste ()
BEGIN
  SELECT
    s.ID AS ID_skladista,
    s.naziv AS Naziv_skladista,
    COUNT(i.ID) AS Broj_izdatnica
  FROM
    skladiste s
    LEFT JOIN zaposlenik z ON s.ID = z.ID_skladiste
    LEFT JOIN izdatnica i ON z.ID = i.ID_zaposlenik
  GROUP BY
    s.ID
  ORDER BY
    Broj_izdatnica DESC
  LIMIT 1;
END //

DELIMITER ;

CALL NajcesceKoristenoSkladiste();
```


##### Procedura 6
Sesta procedura salje iste artikle u skladista u koja mi odaberemo radi lakseg kontroliranja artikala po skladistima 
ima 2 ulazna parametra 

```mysql
DELIMITER //
CREATE PROCEDURE prijenos_artikala(IN p_id_artikl INT, IN p_id_odrediste INT)
BEGIN

    UPDATE artikli_u_skladistu AS a
    JOIN skladiste AS s ON a.id_skladiste = s.id
    SET a.id_skladiste = p_id_odrediste
    WHERE a.id_artikl = p_id_artikl AND s.id != p_id_odrediste;
END //
DELIMITER ;

CALL prijenos_artikala(1, 1);
SELECT * FROM artikli_u_skladistu WHERE id_skladiste = 1;
```

##### Procedura 7

Sedma procedura  zbraja odredene iste artikle nakon prebacivanja u skladiste tako da se sve svrstava pod jedno 
ima jedan ulazni parametar 

```mysql
DELIMITER //
CREATE PROCEDURE spoji_artikle(p_id_skladiste INT)
BEGIN

    UPDATE artikli_u_skladistu AS a1
    JOIN (
        SELECT id_artikl,id_skladiste, SUM(kolicina) AS ukupna_kolicina
        FROM artikli_u_skladistu
        WHERE id_skladiste = p_id_skladiste
        GROUP BY id_artikl, id_skladiste)
        AS a2 ON a1.id_artikl = a2.id_artikl AND a1.id_skladiste = a2.id_skladiste
    SET a1.kolicina = a2.ukupna_kolicina
    WHERE a1.id_artikl IS NOT NULL AND a1.id_skladiste IS NOT NULL AND a1.id_skladiste = p_id_skladiste;

    DELETE a1 FROM artikli_u_skladistu a1
    INNER JOIN artikli_u_skladistu a2
    WHERE a1.id_artikl = a2.id_artikl AND a1.id_skladiste = a2.id_skladiste AND a1.id< a2.ID AND a1.id_skladiste = p_id_skladiste;
END //
DELIMITER ;


CALL spoji_artikle(1);

SELECT aus.ID, aus.ID_skladiste, s.naziv AS skladiste, aus.ID_artikl, a.naziv AS artikl, aus.kolicina
FROM artikli_u_skladistu aus
JOIN skladiste s ON aus.ID_skladiste = s.ID
JOIN artikl a ON aus.ID_artikl = a.ID
WHERE aus.ID_skladiste = 1;
```

##### Procedura 8 
Osma procedura stavlja popust od 20% na sve proizvode  koji se nikada nisu prodali
nema ulaznih niti izlaznih parametara

```mysql
DELIMITER //
CREATE PROCEDURE popust_ne_prodani()
BEGIN
    UPDATE artikl
    SET cijena = cijena * 0.8
    WHERE id NOT IN (SELECT DISTINCT id_artikl FROM stavka_racun);
END //
DELIMITER ;

CALL popust_ne_prodani();
SELECT * FROM artikl;
```


##### Procedura 9

Deveta procedura provjerava da li je dovoljno zaposlenika u odredenim skladistima i ako nije javlja gresku
ima jedan ulazni parametar 

```mysql
DROP PROCEDURE IF EXISTS provjeri_radnike;

DELIMITER //

CREATE PROCEDURE provjeri_radnike (IN p_id_skladiste INT)
BEGIN
  DECLARE broj_radnika INT;

  -- Provjeri broj radnika u određenom skladištu
  SELECT COUNT(*) INTO broj_radnika
  FROM zaposlenik
  WHERE id_skladiste = p_id_skladiste;

  -- Ako ima manje od 5 radnika, generiraj grešku
  IF broj_radnika < 5 THEN
    SIGNAL SQLSTATE '45000'
    SET MESSAGE_TEXT = 'Skladište ima manje od 5 radnika, potrebno je dodati radnike';
  -- Ako ima više od 10 radnika, generiraj grešku
  ELSEIF broj_radnika > 10 THEN
    SIGNAL SQLSTATE '45000'
    SET MESSAGE_TEXT = 'Skladište ima više od 10 radnika, previše radnika u skladištu';
  END IF;
END //

DELIMITER ;

CALL provjeri_radnike(5);
```

##### Procedura 10 

Deseta procedura  prebacuje zaposlenike u druga skladista ovisno o potrebama firme tj. nadopunjujemo proslu proceduru koja nam radi provjeru
ima dva ulazna parametra 

```mysql
DROP PROCEDURE IF EXISTS promjeni_radno_mjesto;


DELIMITER //
CREATE PROCEDURE promjeni_radno_mjesto (
  IN p_id_zaposlenik INT,
  IN p_novo_skladiste INT
)
BEGIN
  -- Provjeri postoji li zaposlenik s zadanim ID-om
  IF NOT EXISTS (SELECT 1 FROM zaposlenik WHERE ID = p_id_zaposlenik) THEN
    SIGNAL SQLSTATE '45000'
    SET MESSAGE_TEXT = 'Zaposlenik s zadanim ID-om ne postoji';

  END IF;

  -- Provjeri postoji li skladište s zadanim ID-om
  IF NOT EXISTS (SELECT 1 FROM skladiste WHERE ID = p_novo_skladiste) THEN
    SIGNAL SQLSTATE '45000'
    SET MESSAGE_TEXT = 'Skladište s zadanim ID-om ne postoji';

  END IF;

  -- Ažuriraj skladište zaposlenika
  UPDATE zaposlenik
  SET ID_skladiste = p_novo_skladiste
  WHERE ID = p_id_zaposlenik;
END //

DELIMITER ;

CALL promjeni_radno_mjesto(1, 3);

select* from zaposlenik;
```
