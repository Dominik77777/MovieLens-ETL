# ETL Proces pre Dataset MovieLens

Tento projekt dokumentuje implementáciu ETL procesu pre dataset MovieLens v Snowflake. Cieľom je spracovať dáta o filmoch, hodnoteniach a používateľoch a vytvoriť dátový model vhodný na multidimenzionálnu analýzu. Analáza sa zameriava na získanie dát o filmoch z požívateľských preferencií na základe hodnotenia jednotlivých filmov. Výsledný dátový model umožňuje multidimenzionálnu analýzu a grafickú vizualizáciu dát.

## 1. Úvod a popis zdrojových dát
Dataset MovieLens obsahuje údaje o filmoch, používateľoch a hodnoteniach. Cieľom analýzy je identifikovať najpopulárnejšie filmy, žánre a takisto informácie o používateľoch ako vek alebo zamestanie.
Zdrojové dáta pochádzajú z grouplens datasetu dostupného tu - https://grouplens.org/datasets/movielens/. Dataset obsahuje 6 hlavných tabuliek:

- `Movies`
- `Users`
- `Ratings`
- `Genres`
- `Occupations`
- `Age_group`

 ## 1.1 Dátová architektúra

 ## ERD diagram
 Surové dáta sú usporiadané v relačnom modeli, ktorý je znázornený na __entitno-relačnom diagrame (ERD)__:
 
![Obrázok 1 Entitno-relačná schéma MovieLens](https://github.com/user-attachments/assets/b35ae0ce-a0df-479f-a020-f2819d54e47e)

## 2. Dimenzionálny model
Navrhnutý bol __hviezdicový model (star schema__), pre efektívnu analýzu kde centrálny bod predstavuje faktová tabuľka __fact_ratings__, ktorá je prepojená s nasledujúcimi dimenziami:

- `dim_movies`: Obsahuje podrobné informácie o filomch (názov, rok vydania, žáner).
- `dim_users`: Obsahuje demografické údaje o používateľoch, ako sú vekové kategórie, pohlavie, povolanie a poštové smerovacie číslo.
- `dim_date`: Zahrňuje informácie o dátumoch hodnotení (deň, mesiac, rok, štvrťrok).
- `dim_time`: Obsahuje podrobné časové údaje (hodina, minúty).

  
Štruktúra hviezdicového modelu je znázornená na diagrame nižšie. Diagram ukazuje prepojenia medzi faktovou tabuľkou a dimenziami, čo zjednodušuje pochopenie a implementáciu modelu.


![star_chema](https://github.com/user-attachments/assets/68fbbe13-00d4-4d2c-9944-b9c544a55f0d)



## 3. ETL proces v Snowflake
ETL proces pozostával z troch hlavných fáz: __extrahovanie (Extract), transformácia (Transform) a načítanie (Load)__. Tento proces bol implementovaný v Snowflake s cieľom pripraviť zdrojové dáta zo staging vrstvy do viacdimenzionálneho modelu vhodného na analýzu a vizualizáciu.

## 3.1 Extract (Extrahovanie dát)
Dáta zo zdrojového datasetu (formát .csv) boli najprv nahraté do Snowflake prostredníctvom interného stage úložiska s názvom my_stage. Stage v Snowflake slúži ako dočasné úložisko na import alebo export dát. Vytvorenie stage bolo zabezpečené príkazom:

## 3.3 Load (Načítanie dát)
Po úspešnom vytvorení dimenzií a faktovej tabuľky boli dáta nahraté do finálnej štruktúry. Na záver boli staging tabuľky odstránené, aby sa optimalizovalo využitie úložiska:
``` sql
DROP TABLE IF EXISTS books_staging;
DROP TABLE IF EXISTS education_levels_staging;
DROP TABLE IF EXISTS occupations_staging;
DROP TABLE IF EXISTS ratings_staging;
DROP TABLE IF EXISTS users_staging;
```
ETL proces v Snowflake umožnil spracovanie pôvodných dát z .csv formátu do viacdimenzionálneho modelu typu hviezda. Tento proces zahŕňal čistenie, obohacovanie a reorganizáciu údajov. Výsledný model umožňuje analýzu filmov a používateľských preferencií pričom poskytuje základ pre vizualizáciu.
## 4 Vizualizácia dát
Dashboard obsahuje 5 vizualizácií, ktoré zobrazujú základný prehľad o filmoch, ich hodnotení a používateľských preferenciách. Tieto vizualizácie zodpovedajú otázky ohľadom popularity filmov a informácií o používaťeľoch.

![MovieLens_dashboard](https://github.com/user-attachments/assets/62c46063-c9a2-4d22-b836-c9b8402ea360)

## Graf 1: 10 najlepšie hodnotených filmov v roku 1999
Táto vizualizácia zobrazuje 10 filmov s najlepším priemerným hodnotením v roku 1999 s minimálne 1000 hodnoteniami. Umožňuje nám identifikovať aké filmy používatelia považujúza najlepšie v jednotlivom roku. Zistili sme napríklad, že najlepšie hodnoteným filmom je Matrix s priemerným hodnotením 4.42.

``` sql
SELECT 
    m.title, 
    AVG(r.rating) AS avg_rating,
    COUNT(r.rating) AS rating_count
FROM FACT_RATINGS r
JOIN DIM_MOVIES m ON r.movieID = m.dim_movieId
WHERE m.release_year = 1999
GROUP BY m.title
HAVING COUNT(r.rating) >= 1000
ORDER BY avg_rating DESC
LIMIT 10;
```
## Graf 2: Počet hodnotení filmov podľa času

``` sql
SELECT
    t.hour AS hour,
    COUNT(r.rating) AS total_ratings
FROM fact_ratings r
JOIN dim_time t ON r.timeId = t.dim_timeId
GROUP BY t.hour
ORDER BY hour;
```

## Graf 3: Rozdelenie žánrov podľa vekovej skupiny používateľov
``` sql
SELECT
    m.genres AS genre,
    u.age_group AS age_group,
    COUNT(r.rating) AS total_ratings
FROM fact_ratings r
JOIN dim_movies m ON r.movieId = m.dim_movieId
JOIN dim_users u ON r.userId = u.dim_userId
GROUP BY m.genres, u.age_group
ORDER BY total_ratings DESC;
```

## Graf 4: Počet hodnotení podľa mesiaca 
``` sql
SELECT
    d.month AS month,
    COUNT(r.rating) AS total_ratings
FROM fact_ratings r
JOIN dim_date d ON r.dateId = d.dim_dateId
GROUP BY d.month
ORDER BY month;
```
## Graf 5: Počet hodnotení podľa povolania a pohlavia
``` sql
SELECT
    u.occupation AS occupation,
    u.gender AS gender,
    COUNT(r.rating) AS total_ratings
FROM fact_ratings r
JOIN dim_users u ON r.userId = u.dim_userId
GROUP BY u.occupation, u.gender
ORDER BY total_ratings DESC;
```
Dashboard MovieLens je interaktívna vizualizácia, ktorá slúži na analýzu a prehľad o rôznych aspektoch dát z databázy MovieLens, ktorá obsahuje informácie o filmoch, hodnoteniach a používateľoch. Tento dashboard poskytuje užívateľom a analytikom nástroje na rýchlu analýzu trendov v dátach, ktoré sú často využívané v oblasti odporúčacích systémov a analýzy filmových preferencií.

__Autor:__ Dominik Čibik
