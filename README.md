- ### Tabelle erstellen

    Zuerst erstellen wir eine Tabelle namens `Artikel`, die eine Spalte für die Artikel-ID und eine Spalte für das JSON-Dokument enthält.

    ```sql
    CREATE TABLE Artikel (
        artikel_id NUMBER GENERATED ALWAYS AS IDENTITY,
        artikel_json CLOB CHECK (artikel_json IS JSON),
        PRIMARY KEY (artikel_id)
    );
    ```

- ### Beispieldatensätze einfügen

    Als nächstes fügen wir 4 Beispieldatensätze in die Tabelle ein. Diese Datensätze enthalten dynamische Eigenschaften, verschachtelte Objekte, Booleans und Arrays.

    ```sql
    INSERT INTO Artikel (artikel_json) VALUES ('{
        "name": "Artikel 1",
        "beschreibung": "Beschreibung von Artikel 1",
        "preis": 10.99,
        "verfuegbar": true,
        "eigenschaften": {
            "farbe": "rot",
            "groesse": "M"
        },
        "tags": ["neu", "angebot"],
        "lagerbestand": 100
    }');

    INSERT INTO Artikel (artikel_json) VALUES ('{
        "name": "Artikel 2",
        "beschreibung": "Beschreibung von Artikel 2",
        "preis": 20.99,
        "verfuegbar": false,
        "eigenschaften": {
            "farbe": "blau",
            "groesse": "L"
        },
        "tags": ["neu"],
        "lagerbestand": 50
    }');

    INSERT INTO Artikel (artikel_json) VALUES ('{
        "name": "Artikel 3",
        "beschreibung": "Beschreibung von Artikel 3",
        "preis": 15.99,
        "verfuegbar": true,
        "eigenschaften": {
            "farbe": "gruen",
            "groesse": "S"
        },
        "tags": ["angebot"],
        "lagerbestand": 75
    }');

    INSERT INTO Artikel (artikel_json) VALUES ('{
        "name": "Artikel 4",
        "beschreibung": "Beschreibung von Artikel 4",
        "preis": 25.99,
        "verfuegbar": false,
        "eigenschaften": {
            "farbe": "gelb",
            "groesse": "XL"
        },
        "tags": ["neu", "angebot"],
        "lagerbestand": 25
    }');
    ```

---
- ### Abfragen durchführen

  - 1. **Finden eines Artikels anhand einer dynamischen Eigenschaft (aus der JSON-Spalte):**

    ```sql
    SELECT artikel_json
    FROM Artikel
    WHERE JSON_EXISTS(artikel_json, '$.eigenschaften.farbe?(@ == "rot")');
    ```

  - 2. **Selektieren (nur) einen Wert aus der JSON-Spalte eines Datensatzes:**

    ```sql
    SELECT JSON_VALUE(artikel_json, '$.name')
    FROM Artikel
    WHERE artikel_id = 1;
    ```

  - 3. **Selektieren Sie einen boolean-Wert, welcher nach 0/1 gemappt werden soll:**

    ```sql
    SELECT
        artikel_id,
        CASE
            WHEN JSON_VALUE(artikel_json, '$.verfuegbar') = 'true' THEN 1
            ELSE 0
        END AS verfuegbar
    FROM Artikel;
    ```

  - 4. **Eine Query, welche einen spezifischen Wert eines Arrays ausliest:**

    ```sql
    SELECT JSON_QUERY(artikel_json, '$.tags[0]')
    FROM Artikel
    WHERE artikel_id = 1;
    ```

  - 5. **Führen Sie eine Group-By Query auf Werte aus der JSON-Spalte aus:**

    ```sql
    SELECT
        JSON_VALUE(artikel_json, '$.eigenschaften.farbe') AS farbe,
        COUNT(*) AS anzahl
    FROM Artikel
    GROUP BY JSON_VALUE(artikel_json, '$.eigenschaften.farbe');
    ```

---
- ### Fremdschlüssel in JSON speichern und JOIN durchführen

    > Angenommen, wir haben eine weitere Tabelle `Kategorien`, die Kategorien für die Artikel enthält.

    ```sql
    CREATE TABLE Kategorien (
        kategorie_id NUMBER GENERATED ALWAYS AS IDENTITY,
        name VARCHAR2(100),
        PRIMARY KEY (kategorie_id)
    );

    INSERT INTO Kategorien (name) VALUES ('Elektronik');
    INSERT INTO Kategorien (name) VALUES ('Kleidung');
    INSERT INTO Kategorien (name) VALUES ('Haushalt');
    ```

    > Jetzt aktualisieren wir die Artikel, um eine Kategorie-ID in das JSON-Dokument einzufügen.

    ```sql
    UPDATE Artikel
    SET artikel_json = JSON_MERGEPATCH(artikel_json, '{"kategorie_id": 1}')
    WHERE artikel_id = 1;

    UPDATE Artikel
    SET artikel_json = JSON_MERGEPATCH(artikel_json, '{"kategorie_id": 2}')
    WHERE artikel_id = 2;

    UPDATE Artikel
    SET artikel_json = JSON_MERGEPATCH(artikel_json, '{"kategorie_id": 3}')
    WHERE artikel_id = 3;

    UPDATE Artikel
    SET artikel_json = JSON_MERGEPATCH(artikel_json, '{"kategorie_id": 1}')
    WHERE artikel_id = 4;
    ```

    > Jetzt führen wir einen JOIN durch, um die Kategorie-Informationen zu erhalten.

    ```sql
    SELECT
        a.artikel_id,
        JSON_VALUE(a.artikel_json, '$.name') AS artikel_name,
        k.name AS kategorie_name
    FROM Artikel a
    JOIN Kategorien k ON JSON_VALUE(a.artikel_json, '$.kategorie_id') = TO_CHAR(k.kategorie_id);
    ```

###### <p align="center"> by Jan Ritt </p>
