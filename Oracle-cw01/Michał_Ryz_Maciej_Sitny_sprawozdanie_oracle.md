# Oracle PL/Sql

widoki, funkcje, procedury, triggery
ćwiczenie

---

Imiona i nazwiska autorów : Michał Ryz i Maciej Sitny

---

<style>
  {
    font-size: 16pt;
  }
</style>

<style scoped>
 li, p {
    font-size: 14pt;
  }
</style>

<style scoped>
 pre {
    font-size: 10pt;
  }
</style>

# Tabele

![](_img/ora-trip1-0.png)

- `Trip` - wycieczki
  - `trip_id` - identyfikator, klucz główny
  - `trip_name` - nazwa wycieczki
  - `country` - nazwa kraju
  - `trip_date` - data
  - `max_no_places` - maksymalna liczba miejsc na wycieczkę
- `Person` - osoby

  - `person_id` - identyfikator, klucz główny
  - `firstname` - imię
  - `lastname` - nazwisko

- `Reservation` - rezerwacje/bilety na wycieczkę
  - `reservation_id` - identyfikator, klucz główny
  - `trip_id` - identyfikator wycieczki
  - `person_id` - identyfikator osoby
  - `status` - status rezerwacji
    - `N` – New - Nowa
    - `P` – Confirmed and Paid – Potwierdzona  i zapłacona
    - `C` – Canceled - Anulowana
- `Log` - dziennik zmian statusów rezerwacji
  - `log_id` - identyfikator, klucz główny
  - `reservation_id` - identyfikator rezerwacji
  - `log_date` - data zmiany
  - `status` - status

```sql
create sequence s_person_seq
   start with 1
   increment by 1;

create table person
(
  person_id int not null
      constraint pk_person
         primary key,
  firstname varchar(50),
  lastname varchar(50)
)

alter table person
    modify person_id int default s_person_seq.nextval;

```

```sql
create sequence s_trip_seq
   start with 1
   increment by 1;

create table trip
(
  trip_id int  not null
     constraint pk_trip
         primary key,
  trip_name varchar(100),
  country varchar(50),
  trip_date date,
  max_no_places int
);

alter table trip
    modify trip_id int default s_trip_seq.nextval;
```

```sql
create sequence s_reservation_seq
   start with 1
   increment by 1;

create table reservation
(
  reservation_id int not null
      constraint pk_reservation
         primary key,
  trip_id int,
  person_id int,
  status char(1)
);

alter table reservation
    modify reservation_id int default s_reservation_seq.nextval;


alter table reservation
add constraint reservation_fk1 foreign key
( person_id ) references person ( person_id );

alter table reservation
add constraint reservation_fk2 foreign key
( trip_id ) references trip ( trip_id );

alter table reservation
add constraint reservation_chk1 check
(status in ('N','P','C'));

```

```sql
create sequence s_log_seq
   start with 1
   increment by 1;


create table log
(
    log_id int not null
         constraint pk_log
         primary key,
    reservation_id int not null,
    log_date date not null,
    status char(1)
);

alter table log
    modify log_id int default s_log_seq.nextval;

alter table log
add constraint log_chk1 check
(status in ('N','P','C')) enable;

alter table log
add constraint log_fk1 foreign key
( reservation_id ) references reservation ( reservation_id );
```

---

# Dane

Należy wypełnić tabele przykładowymi danymi

- 4 wycieczki
- 10 osób
- 10 rezerwacji

Dane testowe powinny być różnorodne (wycieczki w przyszłości, wycieczki w przeszłości, rezerwacje o różnym statusie itp.) tak, żeby umożliwić testowanie napisanych procedur.

W razie potrzeby należy zmodyfikować dane tak żeby przetestować różne przypadki.

```sql
insert into trip(trip_name, country, trip_date, max_no_places)
values ('Wycieczka do Paryza', 'Francja', to_date('2023-09-12', 'YYYY-MM-DD'), 3);

insert into trip(trip_name, country, trip_date,  max_no_places)
values ('Piekny Krakow', 'Polska', to_date('2025-05-03','YYYY-MM-DD'), 2);

insert into trip(trip_name, country, trip_date,  max_no_places)
values ('Znow do Francji', 'Francja', to_date('2025-05-01','YYYY-MM-DD'), 2);

insert into trip(trip_name, country, trip_date,  max_no_places)
values ('Hel', 'Polska', to_date('2025-05-01','YYYY-MM-DD'),  2);

-- person
insert into person(firstname, lastname)
values ('Jan', 'Nowak');

insert into person(firstname, lastname)
values ('Jan', 'Kowalski');

insert into person(firstname, lastname)
values ('Jan', 'Nowakowski');

insert into person(firstname, lastname)
values  ('Novak', 'Nowak');

insert into person(firstname, lastname)
values  ('Adam', 'Piec');

insert into person(firstname, lastname)
values  ('Barbara', 'Szesc');

insert into person(firstname, lastname)
values  ('Anna', 'Siedem');

insert into person(firstname, lastname)
values  ('Bóbr', 'Osiem');

insert into person(firstname, lastname)
values  ('Kamil', 'Dziewiec');

insert into person(firstname, lastname)
values  ('Jan', 'Dziesiaty');

-- reservation
-- trip1
insert  into reservation(trip_id, person_id, status)
values (1, 1, 'P');

insert into reservation(trip_id, person_id, status)
values (1, 2, 'N');

-- trip 2
insert into reservation(trip_id, person_id, status)
values (2, 1, 'P');

insert into reservation(trip_id, person_id, status)
values (2, 4, 'C');

-- trip 3
insert into reservation(trip_id, person_id, status)
values (2, 4, 'P');

insert into reservation(trip_id, person_id, status)
values (3, 4, 'P');

insert into reservation(trip_id, person_id, status)
values (1, 9, 'P');

insert into reservation(trip_id, person_id, status)
values (3, 6, 'C');

insert into reservation(trip_id, person_id, status)
values (2, 2, 'C');

insert into reservation(trip_id, person_id, status)
values (3, 2, 'P');

insert into reservation(trip_id, person_id, status)
values (3, 8, 'C');

insert into reservation(trip_id, person_id, status)
values (4, 5, 'P');

commit

delete from RESERVATION where RESERVATION_ID = 15;
```

proszę pamiętać o zatwierdzeniu transakcji

---

# Zadanie 0 - modyfikacja danych, transakcje

Należy zmodyfikować model danych tak żeby rezerwacja mogła dotyczyć kilku miejsc/biletów na wycieczkę

- do tabeli reservation należy dodać pole
  - no_tickets
- do tabeli log należy dodac pole
  - no_tickets

Należy zmodyfikować zestaw danych testowych

Należy przeprowadzić kilka eksperymentów związanych ze wstawianiem, modyfikacją i usuwaniem danych
oraz wykorzystaniem transakcji

Skomentuj dzialanie transakcji. Jak działa polecenie `commit`, `rollback`?.
Co się dzieje w przypadku wystąpienia błędów podczas wykonywania transakcji? Porównaj sposób programowania operacji wykorzystujących transakcje w Oracle PL/SQL ze znanym ci systemem/językiem MS Sqlserver T-SQL

pomocne mogą być materiały dostępne tu:
https://upel.agh.edu.pl/mod/folder/view.php?id=311899
w szczególności dokument: `1_ora_modyf.pdf`

```sql

alter table RESERVATION add no_tickets int
alter table log add no_tickets int

```

---

# Zadanie 1 - widoki

Tworzenie widoków. Należy przygotować kilka widoków ułatwiających dostęp do danych. Należy zwrócić uwagę na strukturę kodu (należy unikać powielania kodu)

Widoki:

- `vw_reservation`
  - widok łączy dane z tabel: `trip`, `person`, `reservation`
  - zwracane dane: `reservation_id`, `country`, `trip_date`, `trip_name`, `firstname`, `lastname`, `status`, `trip_id`, `person_id`, `no_tickets`
- `vw_trip`
  - widok pokazuje liczbę wolnych miejsc na każdą wycieczkę
  - zwracane dane: `trip_id`, `country`, `trip_date`, `trip_name`, `max_no_places`, `no_available_places` (liczba wolnych miejsc)
- `vw_available_trip`
  - podobnie jak w poprzednim punkcie, z tym że widok pokazuje jedynie dostępne wycieczki (takie które są w przyszłości i są na nie wolne miejsca)

Proponowany zestaw widoków można rozbudować wedle uznania/potrzeb

- np. można dodać nowe/pomocnicze widoki, funkcje
- np. można zmienić def. widoków, dodając nowe/potrzebne pola

# Zadanie 1 - rozwiązanie

```sql

CREATE VIEW vw_reservation AS
SELECT
    r.reservation_id,
    t.country,
    t.trip_date,
    t.trip_name,
    p.firstname,
    p.lastname,
    r.status,
    r.trip_id,
    r.person_id,
    r.no_tickets
FROM reservation r
JOIN trip t ON r.trip_id = t.trip_id
JOIN person p ON r.person_id = p.person_id;


CREATE OR REPLACE VIEW VW_TRIP AS
SELECT
    t.trip_id,
    t.country,
    t.trip_date,
    t.trip_name,
    t.max_no_places,
    (t.max_no_places - COALESCE(SUM(r.no_tickets), 0)) AS no_available_places
FROM trip t
LEFT JOIN reservation r ON t.trip_id = r.trip_id AND (r.STATUS IS NULL OR r.STATUS != 'C')
GROUP BY t.trip_id, t.country, t.trip_date, t.trip_name, t.max_no_places;


CREATE  VIEW vw_available_trip AS
SELECT *
FROM vw_trip
WHERE trip_date > SYSDATE AND no_available_places > 0;
```

---

# Zadanie 2 - funkcje

Tworzenie funkcji pobierających dane/tabele. Podobnie jak w poprzednim przykładzie należy przygotować kilka funkcji ułatwiających dostęp do danych

Procedury:

- `f_trip_participants`
  - zadaniem funkcji jest zwrócenie listy uczestników wskazanej wycieczki
  - parametry funkcji: `trip_id`
  - funkcja zwraca podobny zestaw danych jak widok `vw_eservation`
- `f_person_reservations`
  - zadaniem funkcji jest zwrócenie listy rezerwacji danej osoby
  - parametry funkcji: `person_id`
  - funkcja zwraca podobny zestaw danych jak widok `vw_reservation`
- `f_available_trips_to`
  - zadaniem funkcji jest zwrócenie listy wycieczek do wskazanego kraju, dostępnych w zadanym okresie czasu (od `date_from` do `date_to`)
  - parametry funkcji: `country`, `date_from`, `date_to`

Funkcje powinny zwracać tabelę/zbiór wynikowy. Należy rozważyć dodanie kontroli parametrów, (np. jeśli parametrem jest `trip_id` to można sprawdzić czy taka wycieczka istnieje). Podobnie jak w przypadku widoków należy zwrócić uwagę na strukturę kodu

Czy kontrola parametrów w przypadku funkcji ma sens?

- jakie są zalety/wady takiego rozwiązania?

Proponowany zestaw funkcji można rozbudować wedle uznania/potrzeb

- np. można dodać nowe/pomocnicze funkcje/procedury

# Zadanie 2 - rozwiązanie

```sql

create FUNCTION f_available_trips_to(
  p_country VARCHAR2,
  p_date_from DATE,
  p_date_to DATE
) RETURN SYS_REFCURSOR
AS
  v_cursor SYS_REFCURSOR;
BEGIN
  OPEN v_cursor FOR
    SELECT * FROM vw_available_trip
    WHERE country = p_country
    AND trip_date BETWEEN p_date_from AND p_date_to;
  RETURN v_cursor;
END;
/

create FUNCTION f_person_reservation (p_person_id INT)
RETURN SYS_REFCURSOR
AS
  v_cursor SYS_REFCURSOR;
BEGIN
  OPEN v_cursor FOR
    SELECT * FROM VW_RESERVATION WHERE person_id = p_person_id;
  RETURN v_cursor;
END;
/

create FUNCTION f_trip_participants (p_trip_id INT)
RETURN SYS_REFCURSOR
AS
  v_cursor SYS_REFCURSOR;
BEGIN
  OPEN v_cursor FOR
    SELECT * FROM VW_RESERVATION WHERE trip_id = p_trip_id;
  RETURN v_cursor;
END;
/


```

---

# Zadanie 3 - procedury

Tworzenie procedur modyfikujących dane. Należy przygotować zestaw procedur pozwalających na modyfikację danych oraz kontrolę poprawności ich wprowadzania

Procedury

- `p_add_reservation`
  - zadaniem procedury jest dopisanie nowej rezerwacji
  - parametry: `trip_id`, `person_id`, `no_tickets`
  - procedura powinna kontrolować czy wycieczka jeszcze się nie odbyła, i czy sa wolne miejsca
  - procedura powinna również dopisywać inf. do tabeli `log`
- `p_modify_reservation_status
  - zadaniem procedury jest zmiana statusu rezerwacji
  - parametry: `reservation_id`, `status`
  - procedura powinna kontrolować czy możliwa jest zmiana statusu, np. zmiana statusu już anulowanej wycieczki (przywrócenie do stanu aktywnego nie zawsze jest możliwa – może już nie być miejsc)
  - procedura powinna również dopisywać inf. do tabeli `log`
- `p_modify_reservation
  - zadaniem procedury jest zmiana statusu rezerwacji
  - parametry: `reservation_id`, `no_iickets`
  - procedura powinna kontrolować czy możliwa jest zmiana liczby sprzedanych/zarezerwowanych biletów – może już nie być miejsc
  - procedura powinna również dopisywać inf. do tabeli `log`
- `p_modify_max_no_places`
  - zadaniem procedury jest zmiana maksymalnej liczby miejsc na daną wycieczkę
  - parametry: `trip_id`, `max_no_places`
  - nie wszystkie zmiany liczby miejsc są dozwolone, nie można zmniejszyć liczby miejsc na wartość poniżej liczby zarezerwowanych miejsc

Należy rozważyć użycie transakcji

Należy zwrócić uwagę na kontrolę parametrów (np. jeśli parametrem jest trip_id to należy sprawdzić czy taka wycieczka istnieje, jeśli robimy rezerwację to należy sprawdzać czy są wolne miejsca itp..)

Proponowany zestaw procedur można rozbudować wedle uznania/potrzeb

- np. można dodać nowe/pomocnicze funkcje/procedury

# Zadanie 3 - rozwiązanie

```sql

create PROCEDURE p_add_reservation(
  p_trip_id INT,
  p_person_id INT,
  p_no_tickets INT
) AS
  v_trip_date DATE;
  v_available_places INT;
BEGIN
  -- Czy wycieczka jest valid
  SELECT trip_date INTO v_trip_date FROM trip WHERE trip_id = p_trip_id;
  IF v_trip_date <= SYSDATE THEN
    RAISE_APPLICATION_ERROR(-20001, 'Wycieczka już się odbyła.');
  END IF;

  -- dostępność miejsc
  SELECT no_available_places INTO v_available_places FROM vw_trip WHERE trip_id = p_trip_id;
  IF p_no_tickets > v_available_places THEN
    RAISE_APPLICATION_ERROR(-20002, 'Brak wystarczającej liczby miejsc.');
  END IF;

  -- Dodawanie rezerwacji
  INSERT INTO reservation(trip_id, person_id, no_tickets, status)
  VALUES (p_trip_id, p_person_id, p_no_tickets, 'N');

  --wpis do log
  INSERT INTO log(reservation_id, log_date, status, no_tickets)
  VALUES (s_reservation_seq.CURRVAL, SYSDATE, 'N', p_no_tickets);

  COMMIT;
END;
/

create PROCEDURE p_modify_reservation_status(
  p_reservation_id INT,
  p_status CHAR
) AS
  v_old_status CHAR;
  v_trip_id INT;
  v_no_tickets INT;
BEGIN
  SELECT status, trip_id, no_tickets INTO v_old_status, v_trip_id, v_no_tickets
  FROM reservation WHERE reservation_id = p_reservation_id;


  IF v_old_status = 'C' AND p_status != 'C' THEN
    RAISE_APPLICATION_ERROR(-20003, 'Nie można przywrócić anulowanej rezerwacji.');
  END IF;

  UPDATE reservation SET status = p_status WHERE reservation_id = p_reservation_id;

  INSERT INTO log(reservation_id, log_date, status, no_tickets)
  VALUES (p_reservation_id, SYSDATE, p_status, v_no_tickets);

  COMMIT;
END;
/

create PROCEDURE p_modify_reservation(
    p_reservation_id INT,
    p_new_no_tickets INT
) AS
    v_trip_id INT;
    v_old_no_tickets INT;
    v_max_places INT;
    v_used_places INT;
BEGIN
    SELECT trip_id, no_tickets INTO v_trip_id, v_old_no_tickets
    FROM reservation
    WHERE reservation_id = p_reservation_id;

    SELECT max_no_places INTO v_max_places FROM trip WHERE trip_id = v_trip_id;
    SELECT NVL(SUM(no_tickets), 0) INTO v_used_places
    FROM reservation
    WHERE trip_id = v_trip_id AND status IN ('N', 'P');

    IF (v_used_places - v_old_no_tickets + p_new_no_tickets) > v_max_places THEN
        RAISE_APPLICATION_ERROR(-20003, 'Brak miejsc po zmianie liczby biletów!');
    END IF;

    UPDATE reservation
    SET no_tickets = p_new_no_tickets
    WHERE reservation_id = p_reservation_id;

    INSERT INTO log (reservation_id, log_date, status, no_tickets)
    VALUES (p_reservation_id, SYSDATE,
            (SELECT status FROM reservation WHERE reservation_id = p_reservation_id),
            p_new_no_tickets);

    COMMIT;
    DBMS_OUTPUT.PUT_LINE('Zmodyfikowano liczbę biletów dla rezerwacji ID: ' || p_reservation_id);
EXCEPTION
    WHEN OTHERS THEN
        ROLLBACK;
        DBMS_OUTPUT.PUT_LINE('Błąd: ' || SQLERRM);
END;
/



```

---

# Zadanie 4 - triggery

Zmiana strategii zapisywania do dziennika rezerwacji. Realizacja przy pomocy triggerów

Należy wprowadzić zmianę, która spowoduje, że zapis do dziennika będzie realizowany przy pomocy trigerów

Triggery:

- trigger/triggery obsługujące
  - dodanie rezerwacji
  - zmianę statusu
  - zmianę liczby zarezerwowanych/kupionych biletów
- trigger zabraniający usunięcia rezerwacji

Oczywiście po wprowadzeniu tej zmiany należy "uaktualnić" procedury modyfikujące dane.

> UWAGA
> Należy stworzyć nowe wersje tych procedur (dodając do nazwy dopisek 4 - od numeru zadania). Poprzednie wersje procedur należy pozostawić w celu umożliwienia weryfikacji ich poprawności

Należy przygotować procedury: `p_add_reservation_4`, `p_modify_reservation_status_4` , `p_modify_reservation_4`

# Zadanie 4 - rozwiązanie

```sql

CREATE OR REPLACE TRIGGER trg_log_insert_reservation
AFTER INSERT ON reservation
FOR EACH ROW
BEGIN
    INSERT INTO log (reservation_id, log_date, status, no_tickets)
    VALUES (:NEW.reservation_id, SYSDATE, :NEW.status, :NEW.no_tickets);
END;
/

CREATE OR REPLACE TRIGGER trg_log_update_reservation
AFTER UPDATE OF status, no_tickets ON reservation
FOR EACH ROW
BEGIN
    INSERT INTO log (reservation_id, log_date, status, no_tickets)
    VALUES (:NEW.reservation_id, SYSDATE, :NEW.status, :NEW.no_tickets);
END;
/

CREATE OR REPLACE TRIGGER trg_prevent_delete_reservation
BEFORE DELETE ON reservation
FOR EACH ROW
BEGIN
    RAISE_APPLICATION_ERROR(-20004, 'Nie można usunąć rezerwacji! Użyj anulowania (status=C).');
END;
/


--Procedury z tego zadania wygladaja tak samo jak poprzednie procedury,
--jedyna roznica to brak logowania w procedurze

```

---

# Zadanie 5 - triggery

Zmiana strategii kontroli dostępności miejsc. Realizacja przy pomocy triggerów

Należy wprowadzić zmianę, która spowoduje, że kontrola dostępności miejsc na wycieczki (przy dodawaniu nowej rezerwacji, zmianie statusu) będzie realizowana przy pomocy trigerów

Triggery:

- Trigger/triggery obsługujące:
  - dodanie rezerwacji
  - zmianę statusu
  - zmianę liczby zakupionych/zarezerwowanych miejsc/biletów

Oczywiście po wprowadzeniu tej zmiany należy "uaktualnić" procedury modyfikujące dane.

> UWAGA
> Należy stworzyć nowe wersje tych procedur (np. dodając do nazwy dopisek 5 - od numeru zadania). Poprzednie wersje procedur należy pozostawić w celu umożliwienia weryfikacji ich poprawności.

Należy przygotować procedury: `p_add_reservation_5`, `p_modify_reservation_status_5`, `p_modify_reservation_status_5`

# Zadanie 5 - rozwiązanie

```sql

-- Trigger sprawdzający dostępność miejsc przed dodaniem/zmianą rezerwacji
CREATE OR REPLACE TRIGGER trg_reservation_availability
BEFORE INSERT OR UPDATE OF no_tickets, status ON reservation
FOR EACH ROW
DECLARE
  v_trip_date      DATE;
  v_max_places     INT;
  v_reserved_places INT;
BEGIN
  -- Pobierz datę wycieczki i maksymalną liczbę miejsc
  SELECT trip_date, max_no_places
  INTO v_trip_date, v_max_places
  FROM trip
  WHERE trip_id = :NEW.trip_id;

  -- Sprawdź, czy wycieczka już się odbyła
  IF v_trip_date <= SYSDATE THEN
    RAISE_APPLICATION_ERROR(-20010, 'Wycieczka już się odbyła.');
  END IF;

  -- Oblicz zajęte miejsca (bez anulowanych rezerwacji)
  SELECT NVL(SUM(no_tickets), 0)
  INTO v_reserved_places
  FROM reservation
  WHERE trip_id = :NEW.trip_id AND status != 'C';

  -- Sprawdź dostępność miejsc
  IF (:NEW.status != 'C' AND (v_reserved_places + :NEW.no_tickets) > v_max_places) THEN
    RAISE_APPLICATION_ERROR(-20011, 'Brak wystarczającej liczby miejsc.');
  END IF;
END;
/

CREATE OR REPLACE PROCEDURE p_add_reservation_5(
  p_trip_id    INT,
  p_person_id  INT,
  p_no_tickets INT
) AS
BEGIN
  INSERT INTO reservation (trip_id, person_id, no_tickets, status)
  VALUES (p_trip_id, p_person_id, p_no_tickets, 'N');
  COMMIT;
END;
/

CREATE OR REPLACE PROCEDURE p_modify_reservation_status_5(
  p_reservation_id INT,
  p_status         CHAR
) AS
BEGIN
  UPDATE reservation SET status = p_status
  WHERE reservation_id = p_reservation_id;
  COMMIT;
END;
/

CREATE OR REPLACE PROCEDURE p_modify_reservation_5(
  p_reservation_id INT,
  p_no_tickets     INT
) AS
BEGIN
  UPDATE reservation SET no_tickets = p_no_tickets
  WHERE reservation_id = p_reservation_id;
  COMMIT;
END;
/


```

---

# Zadanie 6

Zmiana struktury bazy danych. W tabeli `trip` należy dodać redundantne pole `no_available_places`. Dodanie redundantnego pola uprości kontrolę dostępnych miejsc, ale nieco skomplikuje procedury dodawania rezerwacji, zmiany statusu czy też zmiany maksymalnej liczby miejsc na wycieczki.

Należy przygotować polecenie/procedurę przeliczającą wartość pola `no_available_places` dla wszystkich wycieczek (do jednorazowego wykonania)

Obsługę pola `no_available_places` można zrealizować przy pomocy procedur lub triggerów

Należy zwrócić uwagę na spójność rozwiązania.

> UWAGA
> Należy stworzyć nowe wersje tych widoków/procedur/triggerów (np. dodając do nazwy dopisek 6 - od numeru zadania). Poprzednie wersje procedur należy pozostawić w celu umożliwienia weryfikacji ich poprawności.

- zmiana struktury tabeli

```sql
alter table trip add
    no_available_places int null
```

- polecenie przeliczające wartość `no_available_places`
  - należy wykonać operację "przeliczenia" liczby wolnych miejsc i aktualizacji pola `no_available_places`

# Zadanie 6 - rozwiązanie

```sql

CREATE OR REPLACE PROCEDURE p_calculate_initial_availability AS
BEGIN
  FOR trip_rec IN (SELECT trip_id, max_no_places FROM trip) LOOP
    UPDATE trip t
    SET no_available_places = trip_rec.max_no_places - (
      SELECT NVL(SUM(no_tickets), 0)
      FROM reservation r
      WHERE r.trip_id = trip_rec.trip_id AND r.status != 'C'
    )
    WHERE t.trip_id = trip_rec.trip_id;
  END LOOP;
  COMMIT;
END;
/



```

---

# Zadanie 6a - procedury

Obsługę pola `no_available_places` należy zrealizować przy pomocy procedur

- procedura dodająca rezerwację powinna aktualizować pole `no_available_places` w tabeli trip
- podobnie procedury odpowiedzialne za zmianę statusu oraz zmianę maksymalnej liczby miejsc na wycieczkę
- należy przygotować procedury oraz jeśli jest to potrzebne, zaktualizować triggery oraz widoki

> UWAGA
> Należy stworzyć nowe wersje tych widoków/procedur/triggerów (np. dodając do nazwy dopisek 6a - od numeru zadania). Poprzednie wersje procedur należy pozostawić w celu umożliwienia weryfikacji ich poprawności.

- może być potrzebne wyłączenie 'poprzednich wersji' triggerów

# Zadanie 6a - rozwiązanie

```sql

CREATE OR REPLACE PROCEDURE p_add_reservation_6a(
  p_trip_id    INT,
  p_person_id  INT,
  p_no_tickets INT
) AS
BEGIN
  INSERT INTO reservation (trip_id, person_id, no_tickets, status)
  VALUES (p_trip_id, p_person_id, p_no_tickets, 'N');

  UPDATE trip
  SET no_available_places = no_available_places - p_no_tickets
  WHERE trip_id = p_trip_id;

  COMMIT;
END;
/

CREATE OR REPLACE PROCEDURE p_modify_reservation_status_6a(
  p_reservation_id INT,
  p_status         CHAR
) AS
  v_trip_id      INT;
  v_old_status   CHAR;
  v_no_tickets   INT;
BEGIN
  SELECT trip_id, status, no_tickets
  INTO v_trip_id, v_old_status, v_no_tickets
  FROM reservation
  WHERE reservation_id = p_reservation_id;

  UPDATE reservation SET status = p_status
  WHERE reservation_id = p_reservation_id;

  IF v_old_status != 'C' AND p_status = 'C' THEN
    UPDATE trip
    SET no_available_places = no_available_places + v_no_tickets
    WHERE trip_id = v_trip_id;
  ELSIF v_old_status = 'C' AND p_status != 'C' THEN
    UPDATE trip
    SET no_available_places = no_available_places - v_no_tickets
    WHERE trip_id = v_trip_id;
  END IF;

  COMMIT;
END;
/

CREATE OR REPLACE PROCEDURE p_modify_max_no_places_6a(
  p_trip_id        INT,
  p_max_no_places  INT
) AS
BEGIN
  UPDATE trip
  SET max_no_places = p_max_no_places,
      no_available_places = no_available_places + (p_max_no_places - max_no_places)
  WHERE trip_id = p_trip_id;
  COMMIT;
END;
/

```

---

# Zadanie 6b - triggery

Obsługę pola `no_available_places` należy zrealizować przy pomocy triggerów

- podczas dodawania rezerwacji trigger powinien aktualizować pole `no_available_places` w tabeli trip
- podobnie, podczas zmiany statusu rezerwacji
- należy przygotować trigger/triggery oraz jeśli jest to potrzebne, zaktualizować procedury modyfikujące dane oraz widoki

> UWAGA
> Należy stworzyć nowe wersje tych widoków/procedur/triggerów (np. dodając do nazwy dopisek 6b - od numeru zadania). Poprzednie wersje procedur należy pozostawić w celu umożliwienia weryfikacji ich poprawności.

- może być potrzebne wyłączenie 'poprzednich wersji' triggerów

# Zadanie 6b - rozwiązanie

```sql

CREATE OR REPLACE TRIGGER trg_update_available_places
AFTER INSERT OR UPDATE OR DELETE ON reservation
FOR EACH ROW
BEGIN
  IF INSERTING THEN
    UPDATE trip
    SET no_available_places = no_available_places - :NEW.no_tickets
    WHERE trip_id = :NEW.trip_id;
  ELSIF UPDATING THEN
    UPDATE trip
    SET no_available_places = no_available_places + :OLD.no_tickets - :NEW.no_tickets
    WHERE trip_id = :NEW.trip_id;
  ELSIF DELETING THEN
    UPDATE trip
    SET no_available_places = no_available_places + :OLD.no_tickets
    WHERE trip_id = :OLD.trip_id;
  END IF;
END;
/

--zmodyfikowane procedury
CREATE OR REPLACE PROCEDURE p_add_reservation_6b(
  p_trip_id    INT,
  p_person_id  INT,
  p_no_tickets INT
) AS
BEGIN
  INSERT INTO reservation (trip_id, person_id, no_tickets, status)
  VALUES (p_trip_id, p_person_id, p_no_tickets, 'N');
  COMMIT;
END;
/

```

# Zadanie 7 - podsumowanie

Porównaj sposób programowania w systemie Oracle PL/SQL ze znanym ci systemem/językiem MS Sqlserver T-SQL

```sql

Najwiekszą roznicą miedzy Oraclem a MS Sqlserver jest uzywanie commita,
 gdzie na sqlserverze jest to automatyczne. Ogólnie skladnia obu
 technologii jest bardzo podobna co sprawia ze osoba znajaca jedna technologie jest w stanie zrozumiec kod Oracla bądź w drugą stronę.

```
