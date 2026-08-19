# NYC Airbnb project

# NYC Airbnb price analysis and price prediction

This is a learning project on the 2019 New York City Airbnb dataset.
The goal was to walk through the whole path of a classic tabular problem:
data cleaning, exploratory analysis, feature engineering, and finally comparing
a few regression models on the nightly price of the listings.

The project consists of four notebooks. They should be run in order, because each
one continues with the CSV file saved by the previous one.

## Data

The dataset is the Kaggle "New York City Airbnb Open Data" (AB_NYC_2019.csv),
around 49 thousand listings from the five boroughs (Manhattan, Brooklyn, Queens,
Bronx, Staten Island).

The `data/` folder is not tracked by git, so the CSV has to be downloaded
separately and placed here:

```
data/AB_NYC_2019.csv
```

The cleaned and the feature engineered versions are saved by the notebooks
themselves into the same folder (`CLEAN_AB_NYC_2019.csv` and
`Features_AB_NYC_2019.csv`).

## Notebooks

**01_eda_clean.ipynb**
Loading, basic exploration, missing values. `reviews_per_month` is filled with
zeros, and two new columns are built from `last_review`:
`days_since_last_review` (with a negative marker when missing) and `has_reviews`.
The `id`, `host_name` and `last_review` columns are dropped, along with the rows
where the price is zero. A few boxplots and histograms to look at outliers, then
the cleaned CSV is saved.

**02_deep_eda.ipynb**
This one is only exploration, no modeling. Map like scatter plots based on
longitude and latitude, colored by borough, room type and price, distributions,
correlation heatmap, average price per borough, the ten most expensive and the ten
cheapest neighbourhoods. At the end an interactive Folium heatmap that is written
to `outputs/nyc_listings_heatmap.html`.

**03_features.ipynb**
Feature engineering. Three distances calculated with the Haversine formula:
Manhattan center, JFK airport, Brooklyn Bridge. OneHotEncoder on `room_type` and
`neighbourhood_group`. Review based derived features (`popularity_index`,
`reviews_per_listing`), and a few simple text features from the listing name:
length, word count, and keyword flags (luxury, cozy, view, central).
`host_id` and `name` are dropped at the end.

**04_modeling.ipynb**
Train test split, with the target transformed by `log1p`, because the price is
heavily right skewed. Outlier cutting is done on the training set only on purpose,
so no information leaks: price above the 99th percentile and `minimum_nights`
above the 95th percentile are removed (about 2.5 percent of the rows).

The `neighbourhood` column goes through a Target encoder, since it has more than
two hundred categories. Numeric columns are scaled for the linear model but not
for the tree models. Everything is wrapped into Pipeline objects, so there is no
leakage during cross validation.

Three models are compared, Ridge, RandomForest and XGBoost, first with five fold
cross validation, then on the test set. Finally XGBoost is tuned with
RandomizedSearchCV.

## Results

Cross validation on the training set, RMSE on the log scale:

| Model        | CV RMSE (log) |
| ------------ | ------------- |
| Ridge        | 0.4050        |
| RandomForest | 0.3826        |
| XGBoost      | 0.4067        |

Test set, converted back to dollars:

| Model         | Test RMSE | Test MAE |
| ------------- | --------- | -------- |
| Ridge         | $187.05   | $58.37   |
| RandomForest  | $181.88   | $54.74   |
| XGBoost       | $181.89   | $56.81   |
| XGBoost tuned | $181.52   | $53.99   |

The parameters found by the search: `max_depth=10`, `learning_rate=0.01`,
`n_estimators=900`, `subsample=0.9`, `colsample_bytree=0.8`,
`min_child_weight=1`.

## How to run

```bash
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
jupyter notebook
```

Then run the notebooks in the order 01 -> 02 -> 03 -> 04.
Notebook 02 and 04 can take a few minutes, especially the RandomizedSearchCV part.

# NYC Airbnb árelemzés és árbecslés

Ez egy tanulós adatelemző projekt a 2019-es New York City Airbnb adathalmazon.
A cél az volt, hogy végigcsináljam egy klasszikus tabuláris feladat teljes útját:
adattisztítás, felfedező elemzés, feature építés, majd néhány regressziós modell
összehasonlítása a hirdetések éjszakai árára.

A projekt négy notebookból áll, ezeket sorrendben érdemes futtatni, mert
mindegyik az előző által kimentett CSV fájllal dolgozik tovább.

## Adatok

Az adathalmaz a Kaggle "New York City Airbnb Open Data" (AB_NYC_2019.csv),
kb. 49 ezer hirdetés az öt kerületből (Manhattan, Brooklyn, Queens, Bronx,
Staten Island).

A `data/` mappa nincs verziókezelve, szóval a CSV fájlt külön le kell tölteni
és ide bemásolni:

```
data/AB_NYC_2019.csv
```

A tisztított és a feature építéssel bővített változatot a notebookok maguk mentik ki
ugyanide (`CLEAN_AB_NYC_2019.csv` és `Features_AB_NYC_2019.csv`).

## Notebookok

**01_eda_clean.ipynb**
Beolvasás, alap felderítés, hiányzó értékek. A `reviews_per_month` nullákkal lett
feltöltve, a `last_review` dátumból pedig két új oszlop készült:
`days_since_last_review` (ahol nincs értékelés, ott negatív jelölést kap) és
`has_reviews`. Kidobtam az `id`, `host_name` és `last_review` oszlopokat, valamint
a nulla árú sorokat. Néhány boxplot és hisztogram az outlierek megnézésére,
a végén a tisztított CSV mentése.

**02_deep_eda.ipynb**
Itt már csak nézelődés van, modellezés nélkül. Térképszerű scatter plotok
hosszúság és szélesség alapján kerület, szobatípus és ár szerint színezve,
eloszlások, korrelációs heatmap, átlagár kerületenként, a tíz legdrágább és tíz
legolcsóbb környék. A végén egy interaktív Folium hőtérkép, ami az
`outputs/nyc_listings_heatmap.html` fájlba kerül.

**03_features.ipynb**
Feature építés. Haversine képlettel három távolság: Manhattan központ, JFK
repülőtér, Brooklyn híd. OneHotEncoder a `room_type` és `neighbourhood_group`
oszlopokra. Értékelés alapú származtatott jellemzők (`popularity_index`,
`reviews_per_listing`), és a hirdetés nevéből néhány egyszerű szöveges feature:
hossz, szavak száma, illetve kulcsszó flagek (luxury, cozy, view, central).
A `host_id` és a `name` a végén kiesik.

**04_modeling.ipynb**
Train test split, a célváltozó `log1p` transzformálva, mert az ár erősen jobbra
ferde. Az outlier vágás szándékosan csak a tanuló halmazon történik, hogy ne
szivárogjon információ: ár a 99. percentilis felett és `minimum_nights` a 95.
percentilis felett kiesik (ez kb. 2,5 százalék).

A `neighbourhood` oszlop Target encoderrel megy be, mert több mint kétszáz
kategória van benne. A numerikus oszlopok a lineáris modellnél skálázva vannak,
a fáknál nem. Minden Pipeline objektumba van csomagolva, így a
keresztvalidációnál nincs adatszivárgás.

Három modell került összehasonlításra Ridge, RandomForest és XGBoost, először
öt szeletes keresztvalidációval, aztán a teszt halmazon. Végül az XGBoost
hangolása RandomizedSearchCV segítségével.

## Eredmények

Keresztvalidáció a tanuló halmazon, RMSE log skálán:

| Modell       | CV RMSE (log) |
| ------------ | ------------- |
| Ridge        | 0.4050        |
| RandomForest | 0.3826        |
| XGBoost      | 0.4067        |

Teszt halmaz, dollárban visszaalakítva:

| Modell           | Test RMSE | Test MAE |
| ---------------- | --------- | -------- |
| Ridge            | $187.05   | $58.37   |
| RandomForest     | $181.88   | $54.74   |
| XGBoost          | $181.89   | $56.81   |
| XGBoost hangolva | $181.52   | $53.99   |

A hangolás után talált paraméterek: `max_depth=10`, `learning_rate=0.01`,
`n_estimators=900`, `subsample=0.9`, `colsample_bytree=0.8`,
`min_child_weight=1`.

## Futtatás

```bash
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
jupyter notebook
```

Ezután a notebookok 01 -> 02 -> 03 -> 04 sorrendben futtathatók.
A 02 és a 04 notebook futása eltarthat pár percig, főleg a RandomizedSearchCV rész.

---
