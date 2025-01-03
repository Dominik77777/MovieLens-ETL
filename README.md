# ETL Proces pre Dataset MovieLens

Tento projekt dokumentuje implementáciu ETL procesu pre dataset MovieLens v Snowflake. Cieľom je spracovať dáta o filmoch, hodnoteniach a používateľoch a vytvoriť dátový model vhodný na multidimenzionálnu analýzu. Analáza sa zameriava na získanie dát o filmoch z požívateľských preferencií na základe hodnotenia jednotlivých filmov. Výsledný dátový model umožňuje multidimenzionálnu analýzu a grafickú vizualizáciu dát.

## 1. Úvod a popis zdrojových dát
Dataset MovieLens obsahuje údaje o filmoch, užívateľoch a hodnoteniach. Cieľom analýzy je identifikovať najpopulárnejšie filmy, žánre a takisto informácie o používateľoch ako vek alebo zamestanie.
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

# 2 Dimenzionálny model
Navrhnutý bol hviezdicový model (star schema), pre efektívnu analýzu kde centrálny bod predstavuje faktová tabuľka fact_ratings, ktorá je prepojená s nasledujúcimi dimenziami:

- [dim_movies](dim_movies): Obsahuje podrobné informácie o filomch (názov, rok vydania, žáner).
- [dim_users](dim_users): Obsahuje demografické údaje o používateľoch, ako sú vekové kategórie, pohlavie, povolanie a poštové smerovacie číslo.
- [dim_date](dim_date): Zahrňuje informácie o dátumoch hodnotení (deň, mesiac, rok, štvrťrok).
- [dim_time](dim_time): Obsahuje podrobné časové údaje (hodina, minúty).
Štruktúra hviezdicového modelu je znázornená na diagrame nižšie. Diagram ukazuje prepojenia medzi faktovou tabuľkou a dimenziami, čo zjednodušuje pochopenie a implementáciu modelu.


