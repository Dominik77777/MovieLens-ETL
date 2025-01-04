# ETL Proces pre Dataset MovieLens

Tento projekt dokumentuje implementáciu ETL procesu pre dataset MovieLens v Snowflake. Cieľom je spracovať dáta o filmoch, hodnoteniach a používateľoch a vytvoriť dátový model vhodný na multidimenzionálnu analýzu. Analáza sa zameriava na získanie dát o filmoch z požívateľských preferencií na základe hodnotenia jednotlivých filmov. Výsledný dátový model umožňuje multidimenzionálnu analýzu a grafickú vizualizáciu dát.

## 1. Úvod a popis zdrojových dát
Dataset MovieLens obsahuje údaje o filmoch, používateľoch a hodnoteniach. Cieľom analýzy je identifikovať najpopulárnejšie filmy, žánre a takisto informácie o používateľoch ako vek alebo zamestanie.
Zdrojové dáta pochádzajú z grouplens datasetu dostupného tu - https://grouplens.org/datasets/movielens/. Dataset obsahuje 6 hlavných tabuliek:

- [Movies](#movies)
- [Users](#users)
- [Ratings](#ratings)
- [Genres](#genres)
- [Occupations](#occupations)
- [Age_group](#age_group)

 ## 1.1 Dátová architektúra

 ## ERD diagram
 Surové dáta sú usporiadané v relačnom modeli, ktorý je znázornený na entitno-relačnom diagrame (ERD):
 
![Obrázok 1 Entitno-relačná schéma MovieLens](https://github.com/user-attachments/assets/b35ae0ce-a0df-479f-a020-f2819d54e47e)

## 2. Dimenzionálny model
Navrhnutý bol hviezdicový model (star schema), pre efektívnu analýzu kde centrálny bod predstavuje faktová tabuľka fact_ratings, ktorá je prepojená s nasledujúcimi dimenziami:

- [dim_movies](dim_movies): Obsahuje podrobné informácie o filomch (názov, rok vydania, žáner).
- [dim_users](dim_users): Obsahuje demografické údaje o používateľoch, ako sú vekové kategórie, pohlavie, povolanie a poštové smerovacie číslo.
- [dim_date](dim_date): Zahrňuje informácie o dátumoch hodnotení (deň, mesiac, rok, štvrťrok).
- [dim_time](dim_time): Obsahuje podrobné časové údaje (hodina, minúty).

  
Štruktúra hviezdicového modelu je znázornená na diagrame nižšie. Diagram ukazuje prepojenia medzi faktovou tabuľkou a dimenziami, čo zjednodušuje pochopenie a implementáciu modelu.


![star_chema](https://github.com/user-attachments/assets/68fbbe13-00d4-4d2c-9944-b9c544a55f0d)



## 3. ETL proces v Snowflake
ETL proces pozostával z troch hlavných fáz: extrahovanie (Extract), transformácia (Transform) a načítanie (Load). Tento proces bol implementovaný v Snowflake s cieľom pripraviť zdrojové dáta zo staging vrstvy do viacdimenzionálneho modelu vhodného na analýzu a vizualizáciu.

## 3.1 Extract (Extrahovanie dát)
Dáta zo zdrojového datasetu (formát .csv) boli najprv nahraté do Snowflake prostredníctvom interného stage úložiska s názvom my_stage. Stage v Snowflake slúži ako dočasné úložisko na import alebo export dát. Vytvorenie stage bolo zabezpečené príkazom:

## 4 Vizualizácia dát
Dashboard obsahuje 5 vizualizácií, ktoré zobrazujú základný prehľad o filmoch, ich hodnotení a používateľských preferenciách. Tieto vizualizácie zodpovedajú otázky ohľadom popularity filmov a informácií o používaťeľoch.

![MovieLens_dashboard](https://github.com/user-attachments/assets/62c46063-c9a2-4d22-b836-c9b8402ea360)

## Graf 1: 10 najlepšie hodnotených filmov v roku 1999
Táto vizualizácia zobrazuje 10 filmov s najlepším priemerným hodnotením v roku 1999 s minimálne 1000 hodnoteniami. Umožňuje nám identifikovať aké filmy používatelia považujúza najlepšie v jednotlivom roku. Zistili sme napríklad, že najlepšie hodnoteným filmom je Matrix s priemerným hodnotením 4.42.

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

## Graf 2: Počet hodnotení filmov podľa času

