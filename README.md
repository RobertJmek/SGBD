# LIBRARY MANAGEMENT

## TABLE OF CONTENTS

1. Briefly present the database (its utility)
2. Create the Entity-Relationship Diagram (ERD)
3. Based on the entity-relationship diagram, create the conceptual diagram of the proposed model, integrating all necessary attributes
4. Implement the created conceptual diagram in Oracle
5. Add coherent information to the created tables (minimum 5 records for each independent entity; minimum 10 records for each associative table)
6. Formulate in natural language a problem to be solved using an independent stored subprogram that utilizes all 3 types of collections studied
7. Formulate in natural language a problem to be solved using an independent stored subprogram that utilizes 2 different types of studied cursors, one of them being a parameterized cursor, dependent on the other cursor. Call the subprogram
8. Formulate in natural language a problem to be solved using an independent stored function-type subprogram that uses 3 of the created tables in a single SQL command. Handle all exceptions that may occur, including the predefined exceptions NO_DATA_FOUND and TOO_MANY_ROWS. Call the subprogram to highlight all handled cases
9. Formulate in natural language a problem to be solved using an independent stored procedure-type subprogram that has at least 2 parameters and uses 5 of the created tables in a single SQL command. Define at least 2 custom exceptions, other than the system-level predefined ones. Call the subprogram to highlight all defined and handled cases
10. Define a statement-level DML trigger. Fire the trigger
11. Define a row-level DML trigger. Fire the trigger
12. Define a DDL trigger. Fire the trigger
13. Formulate in natural language a problem to be solved using a package that includes complex data types and objects necessary for an integrated action flow, specific to the defined database (minimum 2 data types, minimum 2 functions, minimum 2 procedures)

---

The chosen theme for this project is the design and implementation of a complex informational system for managing the activity of a Library (both online and physically).

**Used Infrastructure:**
* **Database Management System (DBMS):** I used Oracle Database 23ai Free. 
* **Software Configuration:**
    * **OS:** The project was developed on macOS (M1 - Apple Silicon)
    * **Virtual Machines / Containerization Usage:** Since the Oracle Database server does not run natively on macOS, I used a container-based virtualization solution.
    * **Platform used:** Docker (managed via Colima) running the `gvenzl/oracle-free:23.6-slim-faststart`[1] image.
* The DBMS runs in an isolated Linux container, ensuring total compatibility with Oracle standards, without affecting the host OS.
* For writing and running SQL and PL/SQL code, I used Oracle SQL Developer version 24.3.1, connected to the Docker container via port 1521.

*[1] https://hub.docker.com/r/gvenzl/oracle-free*

---

## 1. Briefly present the database (its utility):

In a modern context, libraries are no longer just about storing books, but become complex informational nodes that require strict resource tracking.

**Model Utility:** Implementing this relational system brings major benefits over traditional (paper) management, solving the following critical aspects:

* **Granular stock management (Book-Copy Distinction):** The system makes a clear distinction between the intellectual work (the CARTE entity) and the physical object (the EXEMPLAR_DE_CARTE entity). This architecture allows the librarian to know the exact status of each copy.
* **Demographic data optimization (Normalization):** Complying with the 3rd Normal Form (3NF), addresses are managed in an independent entity (ADRESA). This eliminates data redundancy in frequent cases where multiple members of the same family live at the same address, optimizing storage space and ensuring geographical data consistency.
* **Transactional monitoring:** The model allows detailed tracking of reading history through the IMPRUMUT entity. The system automatically calculates due dates and can quickly identify users with overdue books.
* **Unified Loan Flow Management:** By centralizing operations in the IMPRUMUT table, the database manages the entire lifecycle of an interaction through a state mechanism (STATUS). This simplifies processes:
    * **Reservation:** The user blocks a copy online (Status: 'REZERVAT').
    * **Pickup:** The librarian hands over the book (Status becomes 'ACTIV').
    * **Return:** The book returns to stock (Status becomes 'RETURNAT').

---

## 2. Create the Entity-Relationship Diagram (ERD)[1]:

![ERD](ERD.svg)

---

## 3. Based on the entity-relationship diagram, create the conceptual diagram of the proposed model, integrating all necessary attributes[1]:

![CONCEPUTUALA](conceptuala.svg)

---

## 4. Implement the created conceptual diagram in Oracle:

I started with the independent tables to avoid any potential issues related to FKs (Foreign Keys):

```sql
CREATE TABLE ADRESA (
    id_adresa NUMBER(6) PRIMARY KEY,
    localitate      VARCHAR2(50) NOT NULL,
    strada    VARCHAR2(50) NOT NULL,
    numar     VARCHAR2(10) NOT NULL,
    cod_postal VARCHAR(6) NOT NULL,
    etaj NUMBER(2),
    bloc VARCHAR2(6),
    scara VARCHAR2(6),
    apartament NUMBER(4),
    CONSTRAINT chk_adresa_ap CHECK (apartament > 0),
    CONSTRAINT chk_adresa_zip CHECK (LENGTH(cod_postal) = 6)
);

CREATE TABLE ABONAMENT (
    id_abonament         NUMBER(5) PRIMARY KEY,
    tip                  VARCHAR2(30) UNIQUE NOT NULL,
    pret                 NUMBER(5,2),
    valabilitate_luni    NUMBER(3)
);

CREATE TABLE EDITURA (
    id_editura NUMBER(5) PRIMARY KEY,
    nume       VARCHAR2(50) UNIQUE NOT NULL,
    email      VARCHAR2(50) UNIQUE,
    numar_telefon VARCHAR2(15),
    website    VARCHAR2(100)
);

CREATE TABLE GEN (
    id_gen NUMBER(5) PRIMARY KEY,
    nume     VARCHAR2(50) UNIQUE NOT NULL,
    descriere    VARCHAR2(500),
    zona    VARCHAR2(50)
);

CREATE TABLE AUTOR (
    id_autor       NUMBER(6) PRIMARY KEY,
    nume           VARCHAR2(50) NOT NULL,
    prenume        VARCHAR2(50) NOT NULL,
    data_nasterii  DATE NOT NULL,
    biografie      VARCHAR2(1000),
    site_web       VARCHAR2(100)
);

CREATE TABLE CARTE (
    id_carte      NUMBER(9) PRIMARY KEY,
    titlu         VARCHAR2(150) NOT NULL,
    isbn          VARCHAR2(20) UNIQUE NOT NULL,
    an_aparitie   NUMBER(4),
    numar_pagini  NUMBER(5) NOT NULL,
    descriere     VARCHAR2(1000),
    varsta_minima NUMBER(2) DEFAULT 0,
    id_editura    NUMBER(5) NOT NULL,
    CONSTRAINT fk_carte_editura FOREIGN KEY (id_editura) REFERENCES EDITURA(id_editura)
);

CREATE TABLE UTILIZATOR (
    id_utilizator     NUMBER(5) PRIMARY KEY,
    nume              VARCHAR2(50) NOT NULL,
    prenume           VARCHAR2(50) NOT NULL,
    email             VARCHAR2(100) UNIQUE NOT NULL,
    parola            VARCHAR2(128) NOT NULL, 
    telefon           VARCHAR2(15),
    data_nasterii     DATE,
    id_abonament      NUMBER(5),
    data_expirare_abonament DATE,
    id_adresa         NUMBER(6),
    
    CONSTRAINT fk_util_abonament FOREIGN KEY (id_abonament) REFERENCES ABONAMENT(id_abonament),
    CONSTRAINT fk_util_adresa    FOREIGN KEY (id_adresa)    REFERENCES ADRESA(id_adresa)
);

CREATE TABLE EXEMPLAR_DE_CARTE (
    id_exemplar    NUMBER(10) PRIMARY KEY,
    id_carte       NUMBER(9) NOT NULL,
    barcode        VARCHAR2(20) UNIQUE NOT NULL,
    raft_locatie   VARCHAR2(20),
    observatii     VARCHAR2(200),
    
    CONSTRAINT fk_exemplar_carte FOREIGN KEY (id_carte) REFERENCES CARTE(id_carte)
);

CREATE TABLE IMRPTUMUT (
    id_imprumut   NUMBER(10) PRIMARY KEY,
    id_utilizator NUMBER(5) NOT NULL,
    id_exemplar   NUMBER(10) NOT NULL,
    data_start    DATE DEFAULT SYSDATE NOT NULL,
    data_scadenta DATE NOT NULL,
    data_retur    DATE,
    status        VARCHAR2(20) DEFAULT 'REZERVAT',
    observatii    VARCHAR2(200),
    CONSTRAINT fk_imp_utilizator FOREIGN KEY (id_utilizator) REFERENCES UTILIZATOR(id_utilizator),
    CONSTRAINT fk_imp_exemplar   FOREIGN KEY (id_exemplar)   REFERENCES EXEMPLAR_DE_CARTE(id_exemplar),
    CONSTRAINT chk_imp_status CHECK (status IN ('REZERVAT', 'ACTIV', 'RETURNAT', 'ANULAT', 'PIERDUT'))
);

CREATE TABLE CARTE_GEN (
    id_carte NUMBER(9) NOT NULL,
    id_gen NUMBER(5) NOT NULL,
    principal    NUMBER(1) DEFAULT 0,
    CONSTRAINT pk_carte_gen PRIMARY KEY (id_carte, id_gen),
    CONSTRAINT fk_cg_carte  FOREIGN KEY (id_carte) REFERENCES CARTE(id_carte),
    CONSTRAINT fk_cg_gen    FOREIGN KEY (id_gen)   REFERENCES GEN(id_gen)
);

CREATE TABLE CARTE_AUTOR (
    id_carte NUMBER(9) NOT NULL,
    id_autor NUMBER(6) NOT NULL,
    ordine NUMBER(2),
    CONSTRAINT pk_autor_carte PRIMARY KEY (id_carte, id_autor),
    CONSTRAINT fk_ac_carte FOREIGN KEY (id_carte) REFERENCES CARTE(id_carte),
    CONSTRAINT fk_ac_autor FOREIGN KEY (id_autor) REFERENCES AUTOR(id_autor)
);
```

---

## 5. Add coherent information to the created tables (minimum 5 records for each independent entity; minimum 10 records for each associative table).

For ease of insertion, I added automatic indexing to generate and insert each ID using triggers.

```sql
INSERT INTO ABONAMENT VALUES (1, 'Standard', 20, 12);

CREATE SEQUENCE seq_adresa START WITH 1 INCREMENT BY 1 NOCACHE NOCYCLE;

CREATE OR REPLACE TRIGGER trg_adresa
BEFORE INSERT ON ADRESA FOR EACH ROW
BEGIN
    IF :NEW.id_adresa IS NULL THEN
        :NEW.id_adresa := seq_adresa.NEXTVAL;
    END IF;
END;
/

CREATE SEQUENCE seq_abonament START WITH 2 INCREMENT BY 1 NOCACHE NOCYCLE;

CREATE OR REPLACE TRIGGER trg_abonament
BEFORE INSERT ON ABONAMENT FOR EACH ROW
BEGIN
    IF :NEW.id_abonament IS NULL THEN
        :NEW.id_abonament := seq_abonament.NEXTVAL;
    END IF;
END;
/

CREATE SEQUENCE seq_editura START WITH 1 INCREMENT BY 1 NOCACHE NOCYCLE;

CREATE OR REPLACE TRIGGER trg_editura
BEFORE INSERT ON EDITURA FOR EACH ROW
BEGIN
    IF :NEW.id_editura IS NULL THEN
        :NEW.id_editura := seq_editura.NEXTVAL;
    END IF;
END;
/

CREATE SEQUENCE seq_gen START WITH 1 INCREMENT BY 1 NOCACHE NOCYCLE;

CREATE OR REPLACE TRIGGER trg_gen
BEFORE INSERT ON GEN FOR EACH ROW
BEGIN
    IF :NEW.id_gen IS NULL THEN
        :NEW.id_gen := seq_gen.NEXTVAL;
    END IF;
END;
/

CREATE SEQUENCE seq_autor START WITH 1 INCREMENT BY 1 NOCACHE NOCYCLE;

CREATE OR REPLACE TRIGGER trg_autor
BEFORE INSERT ON AUTOR FOR EACH ROW
BEGIN
    IF :NEW.id_autor IS NULL THEN
        :NEW.id_autor := seq_autor.NEXTVAL;
    END IF;
END;
/

CREATE SEQUENCE seq_carte START WITH 1 INCREMENT BY 1 NOCACHE NOCYCLE;

CREATE OR REPLACE TRIGGER trg_carte
BEFORE INSERT ON CARTE FOR EACH ROW
BEGIN
    IF :NEW.id_carte IS NULL THEN
        :NEW.id_carte := seq_carte.NEXTVAL;
    END IF;
END;
/

CREATE SEQUENCE seq_utilizator START WITH 1 INCREMENT BY 1 NOCACHE NOCYCLE;

CREATE OR REPLACE TRIGGER trg_utilizator
BEFORE INSERT ON UTILIZATOR FOR EACH ROW
BEGIN
    IF :NEW.id_utilizator IS NULL THEN
        :NEW.id_utilizator := seq_utilizator.NEXTVAL;
    END IF;
END;
/

CREATE SEQUENCE seq_exemplar START WITH 1 INCREMENT BY 1 NOCACHE NOCYCLE;

CREATE OR REPLACE TRIGGER trg_exemplar
BEFORE INSERT ON EXEMPLAR_DE_CARTE FOR EACH ROW
BEGIN
    IF :NEW.id_exemplar IS NULL THEN
        :NEW.id_exemplar := seq_exemplar.NEXTVAL;
    END IF;
END;
/

CREATE SEQUENCE seq_IMRPTUMUT START WITH 1 INCREMENT BY 1 NOCACHE NOCYCLE;

CREATE OR REPLACE TRIGGER trg_IMRPTUMUT
BEFORE INSERT ON IMRPTUMUT FOR EACH ROW
BEGIN
    IF :NEW.id_imprumut IS NULL THEN
        :NEW.id_imprumut := seq_IMRPTUMUT.NEXTVAL;
    END IF;
END;
/

SELECT * FROM ABONAMENT;

INSERT INTO EDITURA (nume, email, numar_telefon, website) VALUES ('Nemira', 'contact@nemira.ro', '0211112233', 'www.nemira.ro');
INSERT INTO EDITURA (nume, email, numar_telefon, website) VALUES ('Humanitas', 'office@humanitas.ro', '0212223344', 'www.humanitas.ro');
INSERT INTO EDITURA (nume, email, numar_telefon, website) VALUES ('Polirom', 'club@polirom.ro', '0232111222', NULL);
INSERT INTO EDITURA (nume, email, numar_telefon, website) VALUES ('RAO', NULL, '0213334455', '[www.raobooks.com](https://www.raobooks.com)');
INSERT INTO EDITURA (nume, email, numar_telefon, website) VALUES ('Arthur', 'hello@arthur.ro', NULL, 'www.editura-arthur.ro');
INSERT INTO EDITURA (nume, email, numar_telefon, website) VALUES ('Litera', 'contact@litera.ro', '0215556677', 'litera.ro'); -- ID 6
INSERT INTO EDITURA (nume, email, numar_telefon, website) VALUES ('Trei', 'office@edituratrei.ro', '0216667788', 'edituratrei.ro'); -- ID 7
INSERT INTO EDITURA (nume, email, numar_telefon, website) VALUES ('Corint', 'vanzari@corint.ro', '0217778899', 'edituracorint.ro'); -- ID 8

SELECT * FROM editura;

INSERT INTO GEN (nume, descriere, zona) VALUES ('SF', 'Science Fiction and Fantasy', 'Etaj 1');
INSERT INTO GEN (nume, descriere, zona) VALUES ('Roman', 'Literatura Universala', 'Parter');
INSERT INTO GEN (nume, descriere, zona) VALUES ('Politist', 'Thriller si Mistere', 'Etaj 1');
INSERT INTO GEN (nume, descriere, zona) VALUES ('Istorie', NULL, 'Etaj 2');
INSERT INTO GEN (nume, descriere, zona) VALUES ('Copii', 'Carti pentru cei mici', NULL);
INSERT INTO GEN (nume, descriere, zona) VALUES ('Horror', 'Carti de groaza', 'Etaj 1');
INSERT INTO GEN (nume, descriere, zona) VALUES ('Biografie', 'Vietile oamenilor celebri', 'Etaj 2'); 

SELECT * FROM GEN;

INSERT INTO AUTOR (nume, prenume, data_nasterii, biografie) VALUES ('Herbert', 'Frank', TO_DATE('08-10-1920', 'DD-MM-YYYY'), 'Creatorul Dune');
INSERT INTO AUTOR (nume, prenume, data_nasterii) VALUES ('Rowling', 'J.K.', TO_DATE('31-07-1965', 'DD-MM-YYYY'));
INSERT INTO AUTOR (nume, prenume, data_nasterii) VALUES ('Rebreanu', 'Liviu', TO_DATE('27-11-1885', 'DD-MM-YYYY'));
INSERT INTO AUTOR (nume, prenume, data_nasterii) VALUES ('Preda', 'Marin', TO_DATE('05-08-1922', 'DD-MM-YYYY'));
INSERT INTO AUTOR (nume, prenume, data_nasterii) VALUES ('King', 'Stephen', TO_DATE('21-09-1947', 'DD-MM-YYYY'));
INSERT INTO AUTOR (nume, prenume, data_nasterii) VALUES ('Christie', 'Agatha', TO_DATE('15-09-1890', 'DD-MM-YYYY')); -- ID 6
INSERT INTO AUTOR (nume, prenume, data_nasterii) VALUES ('Tolkien', 'J.R.R.', TO_DATE('03-01-1892', 'DD-MM-YYYY')); -- ID 7
INSERT INTO AUTOR (nume, prenume, data_nasterii) VALUES ('Martin', 'George R.R.', TO_DATE('20-09-1948', 'DD-MM-YYYY')); -- ID 8
INSERT INTO AUTOR (nume, prenume, data_nasterii) VALUES ('Asimov', 'Isaac', TO_DATE('02-01-1920', 'DD-MM-YYYY')); -- ID 9
INSERT INTO AUTOR (nume, prenume, data_nasterii) VALUES ('Eliade', 'Mircea', TO_DATE('13-03-1907', 'DD-MM-YYYY'));

SELECT * FROM AUTOR;

INSERT INTO ADRESA (localitate, strada, numar, cod_postal, etaj, bloc, scara, apartament) 
VALUES ('Bucuresti', 'Bd. Unirii', '10', '030123', 2, 'B1', 'A', 12);

INSERT INTO ADRESA (localitate, strada, numar, cod_postal) 
VALUES ('Iasi', 'Str. Pacurari', '22', '700505'); 

INSERT INTO ADRESA (localitate, strada, numar, cod_postal, etaj, bloc, scara, apartament) 
VALUES ('Cluj', 'Str. Horea', '5', '400111', 1, 'C2', 'B', 4);

INSERT INTO ADRESA (localitate, strada, numar, cod_postal, etaj, bloc, scara, apartament) 
VALUES ('Brasov', 'Calea Bucuresti', '100', '500300', 4, 'D1', 'A', 20);

INSERT INTO ADRESA (localitate, strada, numar, cod_postal, etaj, bloc, scara, apartament) 
VALUES ('Timisoara', 'Pta. Victoriei', '1', '300001', 3, 'T1', 'C', 15);

INSERT INTO ADRESA (localitate, strada, numar, cod_postal, bloc, scara, apartament) 
VALUES ('Craiova', 'Decebal', '50', '200300', 'F5', 'C', 12);

INSERT INTO ADRESA (localitate, strada, numar, cod_postal, bloc, scara, apartament) 
VALUES ('Galati', 'Domneasca', '11', '800200', 'G1', 'A', 3);

SELECT * FROM ADRESA;

INSERT INTO CARTE (titlu, isbn, an_aparitie, numar_pagini, id_editura, varsta_minima) 
VALUES ('Dune', '978-100', 1965, 700, 1, 12); -- Editura 1 (Nemira)

INSERT INTO CARTE (titlu, isbn, an_aparitie, numar_pagini, id_editura, varsta_minima) 
VALUES ('Ion', '978-200', 1920, 450, 2, 14); -- Editura 2 (Humanitas)

INSERT INTO CARTE (titlu, isbn, an_aparitie, numar_pagini, id_editura, varsta_minima) 
VALUES ('Morometii', '978-300', 1955, 500, 3, 14); -- Editura 3 (Polirom)

INSERT INTO CARTE (titlu, isbn, an_aparitie, numar_pagini, id_editura, varsta_minima) 
VALUES ('Harry Potter 1', '978-400', 1997, 300, 5, 10); -- Editura 5 (Arthur)

INSERT INTO CARTE (titlu, isbn, an_aparitie, numar_pagini, id_editura, varsta_minima) 
VALUES ('It', '978-500', 1986, 1100, 1, 18); -- Editura 1 (Nemira)

INSERT INTO CARTE (titlu, isbn, an_aparitie, numar_pagini, id_editura, varsta_minima) 
VALUES ('Hobbitul', '978-6', 1937, 350, 4, 10); -- RAO

INSERT INTO CARTE (titlu, isbn, an_aparitie, numar_pagini, id_editura, varsta_minima) 
VALUES ('Urzeala Tronurilor', '978-7', 1996, 800, 1, 18); -- Nemira

INSERT INTO CARTE (titlu, isbn, an_aparitie, numar_pagini, id_editura, varsta_minima) 
VALUES ('Fundatia', '978-8', 1951, 250, 3, 12); -- Polirom

INSERT INTO CARTE (titlu, isbn, an_aparitie, numar_pagini, id_editura, varsta_minima) 
VALUES ('Zece negri mititei', '978-9', 1939, 200, 4, 12); -- RAO

INSERT INTO CARTE (titlu, isbn, an_aparitie, numar_pagini, id_editura, varsta_minima) 
VALUES ('Maitreyi', '978-10', 1933, 180, 2, 14); -- Humanitas

INSERT INTO CARTE (titlu, isbn, an_aparitie, numar_pagini, id_editura, varsta_minima) 
VALUES ('Crima din Orient Express', '978-11', 1934, 220, 4, 12); -- RAO

INSERT INTO CARTE (titlu, isbn, an_aparitie, numar_pagini, id_editura, varsta_minima) 
VALUES ('Stralucirea', '978-12', 1977, 450, 1, 18); -- Nemira

SELECT * FROM CARTE;

INSERT INTO UTILIZATOR (nume, prenume, email, parola, telefon, data_nasterii, id_abonament, id_adresa, data_expirare_abonament) 
VALUES ('Popescu', 'Ion', 'ion@gmail.com', 'pass1', '0722111222', TO_DATE('01-05-1990','DD-MM-YYYY'), 4, 1, SYSDATE+300); -- Angajat, Adresa 1

INSERT INTO UTILIZATOR (nume, prenume, email, parola, telefon, data_nasterii, id_abonament, id_adresa, data_expirare_abonament) 
VALUES ('Ionescu', 'Maria', 'maria@yahoo.com', 'pass2', '0722333444', TO_DATE('15-08-2005','DD-MM-YYYY'), 3, 2, SYSDATE+365); -- Student, Adresa 2

INSERT INTO UTILIZATOR (nume, prenume, email, parola, telefon, data_nasterii, id_abonament, id_adresa, data_expirare_abonament) 
VALUES ('Georgescu', 'Vlad', 'vlad@stud.ro', 'pass3', '0744555666', TO_DATE('10-10-2010','DD-MM-YYYY'), 2, 3, SYSDATE+100); -- Elev, Adresa 3

INSERT INTO UTILIZATOR (nume, prenume, email, parola, telefon, data_nasterii, id_abonament, id_adresa, data_expirare_abonament) 
VALUES ('Dumitru', 'Elena', 'elena@gmail.com', 'pass4', '0755777888', TO_DATE('20-02-1980','DD-MM-YYYY'), 1, 4, SYSDATE-10); -- Standard, Expirat!

INSERT INTO UTILIZATOR (nume, prenume, email, parola, telefon, data_nasterii, id_abonament, id_adresa, data_expirare_abonament) 
VALUES ('Stan', 'Andrei', 'stan@outlook.com', 'pass5', '0766999000', TO_DATE('05-12-1955','DD-MM-YYYY'), 5, 5, SYSDATE+200); -- Pensionar

INSERT INTO UTILIZATOR (nume, prenume, email, parola, telefon, data_nasterii, id_abonament, id_adresa, data_expirare_abonament) 
VALUES ('Dinu', 'Elena', 'elena@y.com', 'p6', '0766', TO_DATE('1980-06-06','YYYY-MM-DD'), 1, 6, SYSDATE+50);

INSERT INTO UTILIZATOR (nume, prenume, email, parola, telefon, data_nasterii, id_abonament, id_adresa, data_expirare_abonament) 
VALUES ('Oprea', 'Gigel', 'gigel@g.com', 'p7', '0777', TO_DATE('1970-07-07','YYYY-MM-DD'), 3, 7, SYSDATE+20);

INSERT INTO UTILIZATOR (nume, prenume, email, parola, telefon, data_nasterii, id_abonament, id_adresa, data_expirare_abonament) 
VALUES ('Toma', 'Alina', 'alina@y.com', 'p8', '0788', TO_DATE('2005-08-08','YYYY-MM-DD'), 2, 6, SYSDATE+60);

SELECT * FROM UTILIZATOR;

INSERT INTO CARTE_AUTOR (id_carte, id_autor) VALUES (1, 1); -- Dune -> Herbert
INSERT INTO CARTE_AUTOR (id_carte, id_autor) VALUES (2, 2); -- HP -> Rowling
INSERT INTO CARTE_AUTOR (id_carte, id_autor) VALUES (3, 3); -- Ion -> Rebreanu
INSERT INTO CARTE_AUTOR (id_carte, id_autor) VALUES (4, 4); -- Morometii -> Preda
INSERT INTO CARTE_AUTOR (id_carte, id_autor) VALUES (5, 5); -- It -> King
INSERT INTO CARTE_AUTOR (id_carte, id_autor) VALUES (6, 7); -- Hobbit -> Tolkien
INSERT INTO CARTE_AUTOR (id_carte, id_autor) VALUES (7, 8); -- GoT -> Martin
INSERT INTO CARTE_AUTOR (id_carte, id_autor) VALUES (8, 9); -- Fundatia -> Asimov
INSERT INTO CARTE_AUTOR (id_carte, id_autor) VALUES (9, 6); -- 10 Negri -> Christie
INSERT INTO CARTE_AUTOR (id_carte, id_autor) VALUES (10, 10); -- Maitreyi -> Eliade
INSERT INTO CARTE_AUTOR (id_carte, id_autor) VALUES (11, 6); -- Orient Express -> Christie (A doua carte pt ea)
INSERT INTO CARTE_AUTOR (id_carte, id_autor) VALUES (12, 5); -- Shining -> King (A doua carte pt el)
INSERT INTO CARTE_AUTOR (id_carte, id_autor, ordine) VALUES (1, 9, 2); -- Dune (Prefata Asimov)
INSERT INTO CARTE_AUTOR (id_carte, id_autor, ordine) VALUES (6, 2, 2); -- Hobbit (Editie Rowling)
INSERT INTO CARTE_AUTOR (id_carte, id_autor, ordine) VALUES (5, 1, 2); -- It (Prefata Herbert)

SELECT * FROM CARTE_AUTOR;

INSERT INTO CARTE_GEN (id_carte, id_gen) VALUES (1, 1); -- Dune  SF
INSERT INTO CARTE_GEN (id_carte, id_gen, principal) VALUES (1, 2, 1); -- Dune Fantasy (Secundar)
INSERT INTO CARTE_GEN (id_carte, id_gen) VALUES (2, 2); -- HP Fantasy
INSERT INTO CARTE_GEN (id_carte, id_gen) VALUES (2, 6); -- HP Copii
INSERT INTO CARTE_GEN (id_carte, id_gen) VALUES (3, 4); -- Ion Dragoste (fortat)
INSERT INTO CARTE_GEN (id_carte, id_gen) VALUES (3, 6); -- Ion Biografie (a taranului)
INSERT INTO CARTE_GEN (id_carte, id_gen) VALUES (4, 5); -- Morometii Istorie
INSERT INTO CARTE_GEN (id_carte, id_gen) VALUES (5, 7); -- It Horror
INSERT INTO CARTE_GEN (id_carte, id_gen) VALUES (5, 3); -- It Politist/Thriller
INSERT INTO CARTE_GEN (id_carte, id_gen) VALUES (6, 2); -- Hobbit Fantasy
INSERT INTO CARTE_GEN (id_carte, id_gen) VALUES (7, 2); -- GoT Fantasy
INSERT INTO CARTE_GEN (id_carte, id_gen) VALUES (7, 3); -- GoT Politic/Mistere
INSERT INTO CARTE_GEN (id_carte, id_gen) VALUES (8, 1); -- Fundatia SF
INSERT INTO CARTE_GEN (id_carte, id_gen) VALUES (9, 3); -- 10 Negri Politist
INSERT INTO CARTE_GEN (id_carte, id_gen) VALUES (10, 4); -- Maitreyi Dragoste
INSERT INTO CARTE_GEN (id_carte, id_gen) VALUES (11, 3); -- Orient Politist

SELECT * FROM CARTE_GEN;

INSERT INTO EXEMPLAR_DE_CARTE (id_carte, barcode, raft_locatie) VALUES (1, 'B001', 'A1');
INSERT INTO EXEMPLAR_DE_CARTE (id_carte, barcode, raft_locatie) VALUES (1, 'B002', 'A1');
INSERT INTO EXEMPLAR_DE_CARTE (id_carte, barcode, raft_locatie) VALUES (2, 'B003', 'B1');
INSERT INTO EXEMPLAR_DE_CARTE (id_carte, barcode, raft_locatie) VALUES (2, 'B004', 'B1');
INSERT INTO EXEMPLAR_DE_CARTE (id_carte, barcode, raft_locatie) VALUES (3, 'B005', 'C2');
INSERT INTO EXEMPLAR_DE_CARTE (id_carte, barcode, raft_locatie) VALUES (5, 'B006', 'D3');
INSERT INTO EXEMPLAR_DE_CARTE (id_carte, barcode, raft_locatie) VALUES (6, 'B007', 'E1');
INSERT INTO EXEMPLAR_DE_CARTE (id_carte, barcode, raft_locatie) VALUES (8, 'B008', 'A2');
INSERT INTO EXEMPLAR_DE_CARTE (id_carte, barcode, raft_locatie) VALUES (10, 'B009', 'F5');
INSERT INTO EXEMPLAR_DE_CARTE (id_carte, barcode, raft_locatie) VALUES (11, 'B010', 'G1');

SELECT * FROM EXEMPLAR_DE_CARTE;

INSERT INTO IMRPTUMUT (id_utilizator, id_exemplar, data_scadenta, status) 
VALUES (1, 1, SYSDATE+10, 'ACTIV');
INSERT INTO IMRPTUMUT (id_utilizator, id_exemplar, data_scadenta, status) 
VALUES (2, 3, SYSDATE+5, 'ACTIV');
INSERT INTO IMRPTUMUT (id_utilizator, id_exemplar, data_scadenta, data_retur, status) 
VALUES (3, 5, SYSDATE-5, SYSDATE-2, 'RETURNAT');
INSERT INTO IMRPTUMUT (id_utilizator, id_exemplar, data_scadenta, data_retur, status) 
VALUES (4, 6, SYSDATE-10, SYSDATE-8, 'RETURNAT');
INSERT INTO IMRPTUMUT (id_utilizator, id_exemplar, data_scadenta, status)
VALUES (5, 8, SYSDATE+20, 'ACTIV');
INSERT INTO IMRPTUMUT (id_utilizator, id_exemplar, data_scadenta, status)
VALUES (6, 10, SYSDATE+2, 'REZERVAT');
INSERT INTO IMRPTUMUT (id_utilizator, id_exemplar, data_scadenta, status) 
VALUES (1, 4, SYSDATE-1, 'ACTIV'); -- Intarziat
INSERT INTO IMRPTUMUT (id_utilizator, id_exemplar, data_scadenta, status) 
VALUES (9, 2, SYSDATE-20, 'PIERDUT');
INSERT INTO IMRPTUMUT (id_utilizator, id_exemplar, data_scadenta, status) 
VALUES (2, 7, SYSDATE+14, 'ACTIV');
INSERT INTO IMRPTUMUT (id_utilizator, id_exemplar, data_start, data_scadenta, data_retur, status) 
VALUES (3, 5, SYSDATE-30, SYSDATE-16, SYSDATE-15, 'RETURNAT');
INSERT INTO IMRPTUMUT (id_utilizator, id_exemplar, data_start, data_scadenta, status) 
VALUES (4, 8, SYSDATE-20, SYSDATE-6, 'ACTIV');
INSERT INTO IMRPTUMUT (id_utilizator, id_exemplar, data_scadenta, status) 
VALUES (1, 4, SYSDATE+3, 'REZERVAT');
INSERT INTO IMRPTUMUT (id_utilizator, id_exemplar, data_start, data_scadenta, status) 
VALUES (5, 6, SYSDATE-5, SYSDATE-3, 'ANULAT');

SELECT * FROM IMRPTUMUT;
```

---

## 6. Formulate in natural language a problem to be solved using an independent stored subprogram that utilizes all 3 types of collections studied. Call the subprogram.

**Problem:** Create a stored procedure named `analiza_cititor` (reader_analysis) that receives a user ID as a parameter and generates a report including:
* Genre Catalog (Index-By Table / Associative Array): A list of all literary genres available in the library, loaded into memory for quick reference.
* Reader's Books (Nested Table): A list of all titles the user currently has actively borrowed.
* Account Status (Vector/Varray): A fixed list of up to 3 informational labels about the account (e.g., subscription type, if there are penalties, account age).

```sql
CREATE OR REPLACE PROCEDURE analiza_cititor(p_id_utilizator IN NUMBER) IS
    TYPE t_map_genuri IS TABLE OF VARCHAR2(50) INDEX BY PLS_INTEGER;
    TYPE t_lista_titluri IS TABLE OF VARCHAR2(150);
    TYPE t_etichete IS VARRAY(3) OF VARCHAR2(50);

    v_catalog_genuri t_map_genuri;        
    v_carti_active   t_lista_titluri;      
    v_status_cont    t_etichete;           --
    
    v_nume_user      VARCHAR2(50);
    v_tip_abo        VARCHAR2(30);

BEGIN

    FOR r IN (SELECT id_gen, nume FROM GEN) LOOP
        v_catalog_genuri(r.id_gen) := r.nume;
    END LOOP;

    SELECT c.titlu
    BULK COLLECT INTO v_carti_active
    FROM CARTE c
    JOIN EXEMPLAR_DE_CARTE e ON c.id_carte = e.id_carte
    JOIN IMPRUMUT i ON e.id_exemplar = i.id_exemplar
    WHERE i.id_utilizator = p_id_utilizator AND i.status = 'ACTIV';

    v_status_cont := t_etichete(); 
    v_status_cont.EXTEND(3);       
    
    SELECT u.nume, a.tip 
    INTO v_nume_user, v_tip_abo
    FROM UTILIZATOR u JOIN ABONAMENT a ON u.id_abonament = a.id_abonament
    WHERE u.id_utilizator = p_id_utilizator;

    v_status_cont(1) := 'Client: ' || v_nume_user;
    v_status_cont(2) := 'Abonament: ' || v_tip_abo;
    
    IF v_carti_active.COUNT > 0 THEN
        v_status_cont(3) := 'Status: ARE IMRPTUMUT';
    ELSE
        v_status_cont(3) := 'Status: FARA CARTI';
    END IF;

    DBMS_OUTPUT.PUT_LINE('-------------------------------------------');
    DBMS_OUTPUT.PUT_LINE('RAPORT ANALIZA PENTRU ID: ' || p_id_utilizator);
    DBMS_OUTPUT.PUT_LINE('-------------------------------------------');

    DBMS_OUTPUT.PUT_LINE('[ INFO CONT ]');
    FOR i IN 1..v_status_cont.COUNT LOOP
        DBMS_OUTPUT.PUT_LINE('  * ' || v_status_cont(i));
    END LOOP;
    
    DBMS_OUTPUT.PUT_LINE(CHR(10) || '[ CARTI IMPRUMUTATE ACUM ]');
    IF v_carti_active.COUNT > 0 THEN
        FOR i IN 1..v_carti_active.COUNT LOOP
            DBMS_OUTPUT.PUT_LINE('  -> ' || v_carti_active(i));
        END LOOP;
    ELSE
        DBMS_OUTPUT.PUT_LINE('  (Nicio carte activa)');
    END IF;

    DBMS_OUTPUT.PUT_LINE(CHR(10) || '[ CATALOG GENURI DISPONIBILE - Extras ]');
    DBMS_OUTPUT.PUT_LINE('  Genul cu ID 1 este: ' || v_catalog_genuri(1));
    DBMS_OUTPUT.PUT_LINE('-------------------------------------------');
EXCEPTION
    WHEN NO_DATA_FOUND THEN
        DBMS_OUTPUT.PUT_LINE('Eroare: Utilizatorul nu exista.');
    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('Eroare: ' || SQLERRM);
END;
/

SET SERVEROUTPUT ON;

BEGIN
    analiza_cititor(1);
    DBMS_OUTPUT.PUT_LINE(''); 
    analiza_cititor(4);
END;
/
```

---

## 7. Formulate in natural language a problem to be solved using an independent stored subprogram that utilizes 2 different types of studied cursors, one of them being a parameterized cursor, dependent on the other cursor. Call the subprogram.

**Problem:** Create an independent stored procedure named `catalog_autori_opere` (authors_works_catalog) that generates a complete bibliographic report. The procedure must meet the following requirements:
* Use a classic explicit cursor to iterate through the list of all authors in the database (alphabetically ordered).
* For each processed author, use a parameterized explicit cursor (dependent on the current author's ID) to extract and display the list of titles written by them.
* Display a distinct message if an author has no books registered in the library.

```sql
CREATE OR REPLACE PROCEDURE catalog_autori_opere IS
    CURSOR c_autori IS
        SELECT id_autor, nume, prenume
        FROM AUTOR
        ORDER BY nume;
    CURSOR c_opere (p_id_aut NUMBER) IS
        SELECT c.titlu, c.an_aparitie
        FROM CARTE c
        JOIN CARTE_AUTOR ca ON c.id_carte = ca.id_carte
        WHERE ca.id_autor = p_id_aut
        ORDER BY c.an_aparitie;

    v_id_autor   AUTOR.id_autor%TYPE;
    v_nume       AUTOR.nume%TYPE;
    v_prenume    AUTOR.prenume%TYPE;
    
    v_titlu      CARTE.titlu%TYPE;
    v_an         CARTE.an_aparitie%TYPE;
    
    v_contor_carti INTEGER;

BEGIN
    DBMS_OUTPUT.PUT_LINE('====== CATALOG BIBLIOGRAFIC ======');
    OPEN c_autori;
    LOOP
        FETCH c_autori INTO v_id_autor, v_nume, v_prenume;
        EXIT WHEN c_autori%NOTFOUND;

        DBMS_OUTPUT.PUT_LINE(CHR(10) || '>> AUTOR: ' || v_nume || ' ' || v_prenume);
        
        v_contor_carti := 0;
        OPEN c_opere(v_id_autor);
        LOOP
            FETCH c_opere INTO v_titlu, v_an;
            EXIT WHEN c_opere%NOTFOUND;
            v_contor_carti := v_contor_carti + 1;
            DBMS_OUTPUT.PUT_LINE('      ' || v_contor_carti || '. ' || v_titlu || ' (' || v_an || ')');
        END LOOP;

        IF v_contor_carti = 0 THEN
            DBMS_OUTPUT.PUT_LINE('      (Nu exista opere inregistrate)');
        END IF;

        CLOSE c_opere;

    END LOOP;
    
    CLOSE c_autori;

    DBMS_OUTPUT.PUT_LINE(CHR(10) || '====== SFARSIT CATALOG ======');
END;
/

BEGIN
    catalog_autori_opere;
END;
/
```

---

## 8. Formulate in natural language a problem to be solved using an independent stored function-type subprogram that uses 3 of the created tables in a single SQL command. Handle all exceptions that may occur, including the predefined exceptions NO_DATA_FOUND and TOO_MANY_ROWS. Call the subprogram to highlight all handled cases.

**Problem:** Create a stored function named `verifica_status_cititor` (check_reader_status) that receives a user's Last Name as a parameter.
The function must query the database through a single SQL command that joins the UTILIZATOR, IMRPTUMUT, EXEMPLAR_DE_CARTE, and CARTE tables, to return the Title of the book the user currently has actively borrowed (status 'ACTIV').

* Ideal case: The user has exactly one borrowed book -> Returns the book title.
* Exceptions:
    * `NO_DATA_FOUND`: The user does not exist or has no active loans -> Returns an informational message.
    * `TOO_MANY_ROWS`: The user has multiple active books simultaneously (which prevents returning a single title in a simple variable) -> Returns a warning that there are multiple loans.

```sql
CREATE OR REPLACE FUNCTION verifica_status_cititor(p_nume_user IN VARCHAR2) 
RETURN VARCHAR2 IS
    v_titlu_carte VARCHAR2(150);
BEGIN
    SELECT c.titlu
    INTO v_titlu_carte
    FROM UTILIZATOR u
    JOIN IMRPTUMUT i ON u.id_utilizator = i.id_utilizator
    JOIN EXEMPLAR_DE_CARTE e ON i.id_exemplar = e.id_exemplar
    JOIN CARTE c ON e.id_carte = c.id_carte
    WHERE u.nume = p_nume_user 
      AND i.status = 'ACTIV';

    RETURN 'Utilizatorul citeste: ' || v_titlu_carte;

EXCEPTION
    WHEN NO_DATA_FOUND THEN
        RETURN 'Info: Utilizatorul ' || p_nume_user || ' nu are niciun imprumut activ sau nu exista';
        
    WHEN TOO_MANY_ROWS THEN
        RETURN 'Atentie: Utilizatorul ' || p_nume_user || ' are MAI MULTE carti imprumutate simultan!';
    WHEN OTHERS THEN
        RETURN 'Eroare : ' || SQLERRM;
END;
/

BEGIN
    DBMS_OUTPUT.PUT_LINE('--- TESTAREA FUNCTIEI ---');
    
    -- CAZ 1: TOO_MANY_ROWS (Popescu are acum 2 carti active)
    DBMS_OUTPUT.PUT_LINE('1. Popescu: ' || verifica_status_cititor('Popescu'));
    
    -- CAZ 2: SUCCESS (Stan are fix 1 carte activa)
    DBMS_OUTPUT.PUT_LINE('2. Stan:    ' || verifica_status_cititor('Stan'));
    
    -- CAZ 3: NO_DATA_FOUND (Georgescu nu are imprumuturi active)
    DBMS_OUTPUT.PUT_LINE('3. Georgescu: ' || verifica_status_cititor('Georgescu'));
    
    -- CAZ BONUS: Utilizator inexistent
    DBMS_OUTPUT.PUT_LINE('4. Superman:  ' || verifica_status_cititor('Superman'));
END;
/
```

---

## 9. Formulate in natural language a problem to be solved using an independent stored procedure-type subprogram that has at least 2 parameters and uses 5 of the created tables in a single SQL command. Define at least 2 custom exceptions, other than the system-level predefined ones. Call the subprogram to highlight all defined and handled cases.

**Problem:** Create a stored procedure named `verificare_eligibilitate` (check_eligibility) that simulates the moment a librarian scans the user's permit and a book's barcode to see if the loan is allowed. The procedure receives the User ID and the Copy's Barcode as parameters.

Through a single SQL command joining 5 tables (UTILIZATOR, ABONAMENT, EXEMPLAR_DE_CARTE, CARTE, EDITURA), the information needed for validation will be extracted.

The procedure must check the rules and raise the following custom exceptions:
* `exc_abonament_expirat`: If the user's subscription expiration date is prior to the current day.
* `exc_restrictie_varsta`: If the user is younger than the minimum age required by the book.
* The predefined exception `NO_DATA_FOUND` must be handled (if the user or book does not exist in the database).

```sql
CREATE OR REPLACE PROCEDURE verificare_eligibilitate(
    p_id_user IN NUMBER, 
    p_barcode IN VARCHAR2
) IS
    v_nume_user   VARCHAR2(50);
    v_data_nastere DATE;
    v_data_exp    DATE;
    v_tip_abo     VARCHAR2(30);
    
    v_titlu_carte VARCHAR2(150);
    v_varsta_min  NUMBER(2);
    v_nume_editura VARCHAR2(50);
    
    v_varsta_user NUMBER(3);

    exc_abonament_expirat EXCEPTION;
    exc_restrictie_varsta EXCEPTION;
    
BEGIN
    SELECT u.nume, u.data_nasterii, u.data_expirare_abonament, a.tip,
           c.titlu, c.varsta_minima, ed.nume
    INTO v_nume_user, v_data_nastere, v_data_exp, v_tip_abo,
         v_titlu_carte, v_varsta_min, v_nume_editura
    FROM UTILIZATOR u
    JOIN ABONAMENT a ON u.id_abonament = a.id_abonament
    CROSS JOIN EXEMPLAR_DE_CARTE e
    JOIN CARTE c ON e.id_carte = c.id_carte
    JOIN EDITURA ed ON c.id_editura = ed.id_editura
    WHERE u.id_utilizator = p_id_user 
      AND e.barcode = p_barcode;

    v_varsta_user := TRUNC(MONTHS_BETWEEN(SYSDATE, v_data_nastere) / 12);

    IF v_data_exp < SYSDATE THEN
        RAISE exc_abonament_expirat;
    END IF;

    IF v_varsta_user < v_varsta_min THEN
        RAISE exc_restrictie_varsta;
    END IF;

    DBMS_OUTPUT.PUT_LINE('----------------------------------------');
    DBMS_OUTPUT.PUT_LINE('SUCCES: Imprumutul este eligibil!');
    DBMS_OUTPUT.PUT_LINE('Client: ' || v_nume_user || ' (' || v_tip_abo || ')');
    DBMS_OUTPUT.PUT_LINE('Carte:  ' || v_titlu_carte || ' (Ed. ' || v_nume_editura || ')');
    DBMS_OUTPUT.PUT_LINE('----------------------------------------');

EXCEPTION
    WHEN exc_abonament_expirat THEN
        DBMS_OUTPUT.PUT_LINE('EROARE: Utilizatorul ' || v_nume_user || ' are abonamentul EXPIRAT din ' || v_data_exp);
        
    WHEN exc_restrictie_varsta THEN
        DBMS_OUTPUT.PUT_LINE('EROARE: Restrictie varsta! Cartea "' || v_titlu_carte || '" cere ' || v_varsta_min || ' ani.');
        DBMS_OUTPUT.PUT_LINE('        Utilizatorul are doar ' || v_varsta_user || ' ani.');

    WHEN NO_DATA_FOUND THEN
        DBMS_OUTPUT.PUT_LINE('EROARE: Nu s-au gasit date. Verificati ID-ul utilizatorului sau Codul de Bare.');
        
    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('Alta eroare: ' || SQLERRM);
END;
/

BEGIN
    -- Popescu (ID 1) vrea Hobbitul (B007)
    verificare_eligibilitate(1, 'B007');
    DBMS_OUTPUT.PUT_LINE('');

    -- Dumitru Elena (ID 4) vrea Hobbitul (B007)
    verificare_eligibilitate(4, 'B007');
    DBMS_OUTPUT.PUT_LINE('');

    -- Georgescu Vlad (ID 3, ~14 ani) vrea IT (B006, interzis sub 18)
    verificare_eligibilitate(3, 'B006');
    DBMS_OUTPUT.PUT_LINE('');

    -- ID gresit sau Barcode inexistent
    verificare_eligibilitate(999, 'B_INEXISTENT');
END;
/
```

---

## 10. Define a statement-level DML trigger. Fire the trigger.

I will create a logging table (`JURNAL_ACTIVITATE`). I will define a trigger to write to this log every time someone makes changes (INSERT, UPDATE, or DELETE) to the IMPRUMUT table.

```sql
CREATE TABLE JURNAL_ACTIVITATE (
    utilizator   VARCHAR2(50),
    data_actiune TIMESTAMP,
    operatiune   VARCHAR2(100)
);

CREATE OR REPLACE TRIGGER trg_audit_global_IMPRUMUT
    AFTER INSERT OR UPDATE OR DELETE ON IMRPTUMUT
DECLARE
    v_actiune VARCHAR2(20);
BEGIN
    
    IF INSERTING THEN
        v_actiune := 'INSERARE';
    ELSIF UPDATING THEN
        v_actiune := 'ACTUALIZARE';
    ELSIF DELETING THEN
        v_actiune := 'STERGERE';
    END IF;
    INSERT INTO JURNAL_ACTIVITATE (utilizator, data_actiune, operatiune)
    VALUES (USER, SYSTIMESTAMP, 'S-a executat o comanda de ' || v_actiune || ' pe tabela IMRPTUMUT');
END;
/

UPDATE IMPRUMUT 
SET data_scadenta = data_scadenta + 1 
WHERE status = 'ACTIV';

COMMIT;

SELECT * FROM JURNAL_ACTIVITATE;
```

---

## 11. Define a row-level DML trigger. Fire the trigger.

When a loan is marked as 'RETURNAT' (RETURNED), the trigger checks if the current date (SYSDATE) is greater than the `data_scadenta` (due date). If so, it automatically appends the text "[RETUR INTARZIAT]" in the observations column.

```sql
CREATE OR REPLACE TRIGGER trg_notificare_intarziere
    BEFORE UPDATE ON IMPRUMUT
    FOR EACH ROW
BEGIN
    IF :NEW.status = 'RETURNAT' AND :OLD.status != 'RETURNAT' THEN
        
        IF SYSDATE > :NEW.data_scadenta THEN
            :NEW.observatii := NVL(:OLD.observatii, '') || ' [RETUR INTARZIAT!]';
        END IF;
        IF :NEW.data_retur IS NULL THEN
             :NEW.data_retur := SYSDATE;
        END IF;
        
    END IF;
END;
/

UPDATE IMRPTUMUT
SET status = 'RETURNAT'
WHERE id_utilizator = 4 
  AND id_exemplar = 8 
  AND status = 'ACTIV';

COMMIT;

select * from IMRPTUMUT;
```

---

## 12. Define a DDL trigger. Fire the trigger.

I chose to make a trigger for the structural modification history (ON SCHEMA).

```sql
CREATE TABLE ISTORIC_DDL (
    utilizator   VARCHAR2(50),
    data_eveniment DATE,
    tip_eveniment VARCHAR2(50), 
    nume_obiect   VARCHAR2(50), 
    tip_obiect    VARCHAR2(50)  
);

CREATE OR REPLACE TRIGGER trg_audit_schema
    AFTER DDL ON SCHEMA
BEGIN
    INSERT INTO ISTORIC_DDL (
        utilizator, 
        data_eveniment, 
        tip_eveniment, 
        nume_obiect, 
        tip_obiect
    )
    VALUES (
        USER, 
        SYSDATE, 
        ORA_SYSEVENT, 
        ORA_DICT_OBJ_NAME, 
        ORA_DICT_OBJ_TYPE
    );
END;
/

CREATE TABLE TABEL_TEST_TRIGGER (
    id NUMBER
);
```

---

## 13. Formulate in natural language a problem to be solved using a package that includes complex data types and objects necessary for an integrated action flow, specific to the defined database (minimum 2 data types, minimum 2 functions, minimum 2 procedures).

The library's public relations department wants to implement a PL/SQL package named `pkg_gestiune_clienti` (client_management_pkg) to centralize operations for reader loyalty and auditing.

**Implementing the functions:**
* A `get_total_penalitati` function to dynamically calculate the user's current debt, considering a penalty of 1 RON for each day of delay for active books that have passed their due date.
* A `get_carti_active` function to return the collection with the exact titles of the books the user currently has borrowed (status 'ACTIV' or 'REZERVAT').

**Implementing the procedures:**
* An `extinde_abonament_gratuit` procedure that checks the user's eligibility: if they have no debts (penalties = 0), their subscription validity will automatically be extended by 1 month (using the ADD_MONTHS function). Otherwise, an error message will be displayed.
* An `afiseaza_raport_complet` procedure that integrates all the elements above to print the client's complete file to the console, including the detailed list of held books.

```sql
CREATE OR REPLACE PACKAGE pkg_gestiune_clienti IS

    TYPE t_profil_financiar IS RECORD (
        id_utilizator NUMBER,
        nume_complet  VARCHAR2(100),
        total_datorie NUMBER,
        status_abo    VARCHAR2(30)
    );

    TYPE t_lista_carti IS TABLE OF VARCHAR2(150);

    FUNCTION get_total_penalitati(p_id_user NUMBER) RETURN NUMBER;
    FUNCTION get_carti_active(p_id_user NUMBER) RETURN t_lista_carti;

    PROCEDURE extinde_abonament_gratuit(p_id_user NUMBER);
    PROCEDURE afiseaza_raport_complet(p_id_user NUMBER);

END pkg_gestiune_clienti;
/


CREATE OR REPLACE PACKAGE BODY pkg_gestiune_clienti IS

    FUNCTION get_total_penalitati(p_id_user NUMBER) RETURN NUMBER IS
        v_zile_intarziere NUMBER := 0;
    BEGIN
        SELECT NVL(SUM(TRUNC(SYSDATE) - TRUNC(data_scadenta)), 0)
        INTO v_zile_intarziere
        FROM IMRPTUMUT
        WHERE id_utilizator = p_id_user
          AND status = 'ACTIV'
          AND data_scadenta < SYSDATE;
        
        RETURN v_zile_intarziere * 1; -- 1 leu pe zi
    END get_total_penalitati;

    FUNCTION get_carti_active(p_id_user NUMBER) RETURN t_lista_carti IS
        v_lista t_lista_carti := t_lista_carti(); 
    BEGIN
        SELECT c.titlu
        BULK COLLECT INTO v_lista
        FROM CARTE c
        JOIN EXEMPLAR_DE_CARTE e ON c.id_carte = e.id_carte
        JOIN IMRPTUMUT i ON e.id_exemplar = i.id_exemplar
        WHERE i.id_utilizator = p_id_user 
          AND i.status IN ('ACTIV', 'REZERVAT');
        
        RETURN v_lista;
    END get_carti_active;

    PROCEDURE extinde_abonament_gratuit(p_id_user NUMBER) IS
        v_datorie   NUMBER;
        v_nume      VARCHAR2(50);
        v_data_veche DATE;
        v_data_noua  DATE;
    BEGIN

        v_datorie := get_total_penalitati(p_id_user);
        
        SELECT nume, data_expirare_abonament 
        INTO v_nume, v_data_veche 
        FROM UTILIZATOR 
        WHERE id_utilizator = p_id_user;

        IF v_datorie = 0 THEN
            UPDATE UTILIZATOR 
            SET data_expirare_abonament = ADD_MONTHS(data_expirare_abonament, 1)
            WHERE id_utilizator = p_id_user
            RETURNING data_expirare_abonament INTO v_data_noua;
            
            DBMS_OUTPUT.PUT_LINE('SUCCES: Abonamentul utilizatorului ' || v_nume || ' a fost prelungit!');
            DBMS_OUTPUT.PUT_LINE('   Vechea data: ' || TO_CHAR(v_data_veche, 'DD-MM-YYYY'));
            DBMS_OUTPUT.PUT_LINE('   Noua data:   ' || TO_CHAR(v_data_noua, 'DD-MM-YYYY'));
        ELSE
            DBMS_OUTPUT.PUT_LINE('ESEC: Utilizatorul ' || v_nume || ' are intarzieri (' || v_datorie || ' RON). Nu primeste bonus.');
        END IF;
    EXCEPTION
        WHEN NO_DATA_FOUND THEN
            DBMS_OUTPUT.PUT_LINE('Eroare: Utilizatorul nu exista.');
    END extinde_abonament_gratuit;


PROCEDURE afiseaza_raport_complet(p_id_user NUMBER) IS
        v_profil  t_profil_financiar; 
        v_carti   t_lista_carti;      
    BEGIN
        DBMS_OUTPUT.PUT_LINE('---------------------------------');
        DBMS_OUTPUT.PUT_LINE('RAPORT GESTIUNE CLIENT #' || p_id_user);
        
        -- Date profil
        SELECT u.id_utilizator, u.nume || ' ' || u.prenume, 
               get_total_penalitati(u.id_utilizator),
               a.tip || ' (Exp: ' || TO_CHAR(u.data_expirare_abonament, 'DD-MM-YYYY') || ')'
        INTO v_profil
        FROM UTILIZATOR u 
        JOIN ABONAMENT a ON u.id_abonament = a.id_abonament
        WHERE u.id_utilizator = p_id_user;

        DBMS_OUTPUT.PUT_LINE('Nume: ' || v_profil.nume_complet);
        DBMS_OUTPUT.PUT_LINE('Detalii Abo: ' || v_profil.status_abo);
        DBMS_OUTPUT.PUT_LINE('Penalitati Active: ' || v_profil.total_datorie || ' RON');

   
        v_carti := get_carti_active(p_id_user);

        DBMS_OUTPUT.PUT_LINE('Carti Active (' || v_carti.COUNT || '):');
        
        IF v_carti.COUNT > 0 THEN
            FOR i IN 1..v_carti.COUNT LOOP
                DBMS_OUTPUT.PUT_LINE('  -> ' || v_carti(i));
            END LOOP;
        ELSE
            DBMS_OUTPUT.PUT_LINE('  (Niciun imprumut activ)');
        END IF;
        
        DBMS_OUTPUT.PUT_LINE('---------------------------------');
        EXCEPTION
        WHEN NO_DATA_FOUND THEN
            DBMS_OUTPUT.PUT_LINE('Eroare: Utilizatorul nu exista.');
    END afiseaza_raport_complet;
END pkg_gestiune_clienti;
/


BEGIN
    DBMS_OUTPUT.PUT_LINE('=== CAZ 1: Utilizator fara probleme (Georgescu - ID 3) ===');
    
    pkg_gestiune_clienti.afiseaza_raport_complet(3);
    
    DBMS_OUTPUT.PUT_LINE('>> Aplicare bonus...');
    pkg_gestiune_clienti.extinde_abonament_gratuit(3);
    
    pkg_gestiune_clienti.afiseaza_raport_complet(3);
    
    DBMS_OUTPUT.PUT_LINE('');
    
    DBMS_OUTPUT.PUT_LINE('=== CAZ 2: Utilizator cu probleme (Popescu - ID 1) ===');
    pkg_gestiune_clienti.extinde_abonament_gratuit(1);
    
    pkg_gestiune_clienti.afiseaza_raport_complet(1);
END;
/
```

---
