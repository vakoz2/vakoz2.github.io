## Zadanie GEO
### Elasticsearch
Aby wygenerować poniższe geojsony należy uruchomić skrypt **el_geo.bat**, znajdujący się w głównym katalogu. Korzystam z bazy z zadania 1. Skrypt zapisuje pliki wynikowe do katalogu geojson.

Przestępstwa dokonane w promieniu kilometra od ratusza
```markdown
{
    "query": {
        "bool" : {
            "must" : {
                "match_all" : {}
            },
            "filter" : {
                "geo_distance" : {
                    "distance" : "1km",
                    "Location": [-87.631631, 41.88386]
                }
            }
        }
    }
}
```
<script src="https://embed.github.com/view/geojson/vakoz2/nosql/master/geojson/query1.geojson"></script>
Przestępstwa dokonane na zadanym obszarze (polygon)
```markdown
{
    "query": {
        "bool" : {
            "must" : {
                "match_all" : {}
            },
            "filter" : {
                "geo_polygon" : {
                    "Location" : {
                        "points" : [
                            [-87.63119101524353, 41.89085702404937],
                            [-87.62666344642639, 41.89085702404937],
                            [-87.62666344642639, 41.89322904173341],
                            [-87.63119101524353, 41.89322904173341]
                        ]
                    }
                }
            }
        }
    }
}
```
<script src="https://embed.github.com/view/geojson/vakoz2/nosql/master/geojson/query2.geojson"></script>
Kradzieże na terenie lotniska (bounding_box)
```markdown
{
    "query": {
        "bool" : {
            "must" : {
                "match" : {"Primary Type": "THEFT"}
            },
            "filter" : {
                "geo_bounding_box" : {
                    "Location": {
                      "top_left": [-87.9400634765625,42.00772369765501],
                      "bottom_right": [-87.86848068237305,41.956171100940026]
                    }
                }
            }
        }
    }
}

```
<script src="https://embed.github.com/view/geojson/vakoz2/nosql/master/geojson/query3.geojson"></script>

### Posgresql
Korzystając z bazy z zadania pierwszego zainstalowałem rozszerzenie [PostGIS](http://postgis.net). Aby móc korzystać z zapytań geolokacyjnych należy wykonać następujące komendy:
```markdown
psql -d test -c "CREATE EXTENSION IF NOT EXISTS postgis"
psql -d test -c "ALTER TABLE crimes ADD COLUMN geom geometry(POINT,4326)"
psql -d test -c "UPDATE crimes SET geom = ST_SetSRID(ST_MakePoint(\"Longitude\",\"Latitude\"),4326)"
```
Zwróciło **UPDATE 9773**, czyli wszystkie wiersze zostały zaktualizowane.

Generowanie geojsonów już sobie odpuściełem (jest to oczywiście trywialne - wystarczy wywołać **sql2csv**(CSVKit) z odpowiednim zapytaniem i przy pomocy **csvjson** przerobić wynik na geojson). Na dowód, że wszystko jest ok, zadam pytanie o to samo, o co pytałem w ostatnim przykładzie w Elasticku (**jako, że niektóre przestępstwa mają dokładnie te same współrzędne liczba na mapie nie pokrywa się z tabelką**).
```markdown
SELECT "Date", "Primary Type", "Description", "Location Description", "Arrest", "Longitude", "Latitude"
FROM crimes 
WHERE "Primary Type" LIKE '%THEFT%' AND geom &&
ST_MakeEnvelope(-87.93834686279295, 41.95693703889415, -87.87483215332031, 42.0064481470799, 4326)
ORDER BY "Longitude", "Latitude", "Date"
```
Wynik:
```markdown
        Date         |    Primary Type     |   Description   |              Location Description              | Arrest |   Longitude   |   Latitude
---------------------+---------------------+-----------------+------------------------------------------------+--------+---------------+--------------
 2015-09-20 18:00:00 | THEFT               | RETAIL THEFT    | AIRPORT VENDING ESTABLISHMENT                  | f      | -87.906864845 | 41.979435886
 2013-03-28 15:30:00 | THEFT               | OVER $500       | AIRPORT TERMINAL LOWER LEVEL - NON-SECURE AREA | f      | -87.906463155 | 41.979006297
 2013-10-06 20:00:00 | THEFT               | $500 AND UNDER  | AIRPORT TERMINAL LOWER LEVEL - SECURE AREA     | f      | -87.906463155 | 41.979006297
 2016-01-22 14:00:00 | THEFT               | $500 AND UNDER  | AIRPORT TERMINAL LOWER LEVEL - SECURE AREA     | f      | -87.906463155 | 41.979006297
 2016-06-06 14:00:00 | THEFT               | PURSE-SNATCHING | CTA TRAIN                                      | f      | -87.906463155 | 41.979006297
 2016-08-21 14:09:00 | THEFT               | $500 AND UNDER  | AIRCRAFT                                       | f      | -87.906463155 | 41.979006297
 2014-07-30 16:00:00 | THEFT               | $500 AND UNDER  | AIRPORT TERMINAL UPPER LEVEL - NON-SECURE AREA | f      | -87.905227221 | 41.976290414
 2016-11-06 11:00:00 | THEFT               | FROM BUILDING   | AIRPORT TERMINAL UPPER LEVEL - SECURE AREA     | f      | -87.905227221 | 41.976290414
 2015-07-27 16:30:00 | THEFT               | OVER $500       | AIRPORT TERMINAL UPPER LEVEL - SECURE AREA     | f      | -87.904976266 |   41.9764212
 2013-03-14 17:00:00 | THEFT               | $500 AND UNDER  | AIRPORT TERMINAL LOWER LEVEL - SECURE AREA     | f      | -87.890372151 | 41.974861952
 2012-03-30 00:01:00 | THEFT               | OVER $500       | PARKING LOT/GARAGE(NON.RESID.)                 | f      | -87.883611316 | 41.980826277
 2014-03-21 08:00:00 | MOTOR VEHICLE THEFT | AUTOMOBILE      | AIRPORT VENDING ESTABLISHMENT                  | f      | -87.883611316 | 41.980826277
 2015-09-02 12:00:00 | THEFT               | $500 AND UNDER  | AIRPORT VENDING ESTABLISHMENT                  | f      | -87.882404127 | 41.983030629
(13 wierszy)
```
A tutaj wynik z Elastica (posortowany csvsort i przy użyciu csvtomd przerobiony na gitową tabelkę)
```
Date                 |  Primary Type         |  Description      |  Location Description                            |  Arrest  |  Longitude      |  Latitude
---------------------|-----------------------|-------------------|--------------------------------------------------|----------|-----------------|--------------
2015-09-20T18:00:00  |  THEFT                |  RETAIL THEFT     |  AIRPORT VENDING ESTABLISHMENT                   |  False   |  -87.906864845  |  41.979435886
2013-03-28T15:30:00  |  THEFT                |  OVER $500        |  AIRPORT TERMINAL LOWER LEVEL - NON-SECURE AREA  |  False   |  -87.906463155  |  41.979006297
2013-10-06T20:00:00  |  THEFT                |  $500 AND UNDER   |  AIRPORT TERMINAL LOWER LEVEL - SECURE AREA      |  False   |  -87.906463155  |  41.979006297
2016-01-22T14:00:00  |  THEFT                |  $500 AND UNDER   |  AIRPORT TERMINAL LOWER LEVEL - SECURE AREA      |  False   |  -87.906463155  |  41.979006297
2016-06-06T14:00:00  |  THEFT                |  PURSE-SNATCHING  |  CTA TRAIN                                       |  False   |  -87.906463155  |  41.979006297
2016-08-21T14:09:00  |  THEFT                |  $500 AND UNDER   |  AIRCRAFT                                        |  False   |  -87.906463155  |  41.979006297
2014-07-30T16:00:00  |  THEFT                |  $500 AND UNDER   |  AIRPORT TERMINAL UPPER LEVEL - NON-SECURE AREA  |  False   |  -87.905227221  |  41.976290414
2016-11-06T11:00:00  |  THEFT                |  FROM BUILDING    |  AIRPORT TERMINAL UPPER LEVEL - SECURE AREA      |  False   |  -87.905227221  |  41.976290414
2015-07-27T16:30:00  |  THEFT                |  OVER $500        |  AIRPORT TERMINAL UPPER LEVEL - SECURE AREA      |  False   |  -87.904976266  |  41.9764212
2013-03-14T17:00:00  |  THEFT                |  $500 AND UNDER   |  AIRPORT TERMINAL LOWER LEVEL - SECURE AREA      |  False   |  -87.890372151  |  41.974861952
2012-03-30T00:01:00  |  THEFT                |  OVER $500        |  PARKING LOT/GARAGE(NON.RESID.)                  |  False   |  -87.883611316  |  41.980826277
2014-03-21T08:00:00  |  MOTOR VEHICLE THEFT  |  AUTOMOBILE       |  AIRPORT VENDING ESTABLISHMENT                   |  False   |  -87.883611316  |  41.980826277
2015-09-02T12:00:00  |  THEFT                |  $500 AND UNDER   |  AIRPORT VENDING ESTABLISHMENT                   |  False   |  -87.882404127  |  41.983030629
```

Jak widać wszystkie punkty się zgadzają.
