# Getting Started with CHAP-Core
**For master students working with forecasting, modeling and DHIS2**

Denne tutorialen gir deg en praktisk introduksjon til CHAP-Core – motoren som kjører modeller for epidemiologisk prediksjon, forecasting og evaluering i DHIS2-økosystemet.

Målet er at du skal:

- Installere og kjøre CHAP-Core lokalt
- Forstå hvordan CHAP bruker modeller, datasets og metadata
- Kjøre din første modell og tolke resultatene
- Navigere prosjektstrukturen du skal jobbe med i masteroppgaven
- Få en base du kan bygge videre på (egne modeller, egne datasett, integrasjon med DHIS2)



---

#  1. Hva er CHAP-Core?

CHAP-Core er en Python-basert modellmotor som:

- leser inn data (DHIS2 eller CSV)
- kjører modeller (regresjon, tidsserier, ML)
- beregner usikkerhet / konfidensintervaller
- produserer output som kan integreres i DHIS2 Modeling App

Kort sagt: **CHAP-Core = backend-hjernen i modellering for DHIS2**.

---

# 2. Forutsetninger


Du trenger:

- Python 3.9 eller nyere
- pip
- Et fungerende virtuelt miljø anbefales

--- 

# 3. Installer CHAP-core

pip install chap-core

sjekk at kommandoen fungerer: 
    chap --help

Hvis du får en liste med kommandoer -> alt fungerer. 

---

# 4. Opprett prosjektstruktur 

getting-started-chap/
│
├── data/
│   ├── example_dataset.csv
│   └── metadata.json
│
├── model/
│   └── example_model.yaml
│
└── output/


---
# 5. Lag et eksempel-dataset
Opprett filen: 
    data/example_dataset.cvs

region,period,rainfall,temperature,population,incidence_rate
UG-01,2020Q1,123,27,200000,14.2
UG-01,2020Q2,98,25,200000,13.7
UG-01,2020Q3,76,24,200000,11.5
UG-01,2020Q4,90,26,200000,12.8

Opprett metadatafilen:
    data/metadata.json

{
  "dataset_name": "uganda_tutorial_dataset",
  "country": "uga",
  "period_type": "quarterly",
  "geography_level": "district",
  "target_variable": "incidence_rate"
}


# 6. Definer en enkel modell

model:
  name: example_model
  type: regression
  parameters:
    n_estimators: 100
    max_depth: 3

training:
  target: incidence_rate
  features:
    - rainfall
    - temperature
    - population

validation:
  metrics:
    - rmse
    - mae

Dette er en enkel modell – men den demonstrerer hele CHAP-pipelinen.

--- 

# 7. Kjør modellen med CHAP-Core

chap evaluate \
    --model model/example_model.yaml \
    --dataset data/example_dataset.csv \
    --metadata data/metadata.json \
    --output output/
