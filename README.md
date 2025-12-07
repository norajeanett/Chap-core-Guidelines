# Getting Started with CHAP-Core – For Master Students

Denne tutorialen gir en strukturert og pedagogisk introduksjon til hvordan du:
1. Først jobber med en modell og metrikk isolert (uten CHAP)
2. Deretter integrerer den samme logikken i CHAP-Core
3. (Valgfritt senere) kobler mot DHIS2 Modeling App

Målet er at du **skal forstå hva som skjer**, ikke bare kjøre kode.



## Når du er ferdig med denne tutorialen skal du kunne:

- Forklare hva CHAP-Core er og hva det brukes til
- Kjøre en modell og en metrikk isolert uten CHAP
- Forstå hvordan prediksjoner og observasjoner representeres
- Implementere en enkel evalueringsmetrikk i pandas
- Forstå hvordan samme metrikk kan gjøres CHAP-kompatibel
- Forberede en modell for integrasjon i CHAP



##  Forutsetninger

Du trenger kun:

- Python 3.9 eller nyere
- Ingen forhåndsinstallasjon av biblioteker
- Ingen forkunnskap om CHAP eller DHIS2



## 1. Oppsett på helt blank PC (med `uv` + `requirements.txt`)

I denne tutorialen bruker vi:

- **`uv`** til å:
  - lage og håndtere virtuelt miljø
  - installere avhengigheter raskt
  - låse versjoner (reproduserbart oppsett)
- **`requirements.txt`** til å:
  - definere hvilke Python-pakker prosjektet trenger
  - sikre at alle studenter bruker samme versjoner

Poenget er at **studentene slipper å tenke på “pip-kaos”**, og bare trenger noen få, tydelige steg.




### 1.1 Hvorfor bruker vi `requirements.txt`?

`requirements.txt`:

- er en **liste over alle avhengigheter** (f.eks. `pandas`, `chap-core`, `numpy` …)
- gjør det mulig å:
  - sette opp nøyaktig samme miljø på alle maskiner
  - reprodusere resultater senere
  - unngå “det funker hos meg, men ikke hos deg”

Eksempel på en enkel `requirements.txt` for denne tutorialen:

```txt
chap-core
pandas
numpy
pyyaml
scikit-learn
```

### 1.2 Hvorfor bruker vi uv sync i stedet for pip install?

```uv```:

- lager virtuelt miljø automatisk for prosjektet ditt

- installerer alt som står i ```requirements.txt``` med én kommando

- låser versjoner i en låsefil (```uv.lock```), slik at miljøet kan gjenskapes senere

Det betyr at:

- studenter trenger kun én kommando for å sette opp alt

- dere slipper å forklare ```python -m venv```, ```pip install```, osv.

- alle får samme versjoner av pakkene → mindre feilsøking

### 3 Steg-for-steg for studenter

Fra en helt blank maskin:

#### 1.3.1 Installer uv (gjøres bare én gang per maskin)

**macOS / Linux:**

```curl -LsSf https://astral.sh/uv/install.sh | sh```


**Windows (PowerShell):**

```irm https://astral.sh/uv/install.ps1 | iex```


Sjekk at uv er installert:

```uv --version```

#### 1.3.2 Sett opp prosjektet med uv sync

Gå til prosjektmappen der ```requirements.txt``` ligger, og kjør:

```uv sync -r requirements.txt```


Dette gjør automatisk:

- oppretter et virtuelt miljø for prosjektet

- installerer alle pakkene fra requirements.txt

- sørger for at alle bruker samme versjoner


De trenger ikke å kjøre pip install direkte, kun uv sync -r requirements.txt.

#### 1.3.3 Kjøre Python-skript via uv

Når uv sync er ferdig, kan du kjøre koden slik:

```uv run python isolated_metric.py```


eller hvilket skript tutorialen bruker.

### 1.4 Hvordan oppdaterer vi requirements.txt? 

Dette er relevant for deg som lager/opprettholder tutorialen.

Typisk arbeidsflyt:
**1. Installer eller oppdater en pakke (inne i prosjektet):**

```uv pip install ny-pakke```


eller (hvis du allerede er i venv):

```pip install ny-pakke```


**2. Oppdater requirements.txt basert på faktisk miljø:**

```uv pip freeze > requirements.txt```


(eller ```pip freeze``` > ```requirements.txt``` fra aktivert venv)

**3. Test at det funker på “blankt miljø” ved å kjøre:**

```uv sync -r requirements.txt```
```uv run python isolated_metric.py```


## 2. Den isolerte fasen – før vi bruker CHAP

I denne delen av tutorialen jobber vi **helt uten CHAP-Core**.

Målet med denne fasen er at du skal:

- forstå hvordan en modell/metric fungerer i praksis
- se nøyaktig hva som er:
  - input
  - beregning
  - output
- kunne teste og feilsøke logikken uten kompleks infrastruktur

> Viktig:
> I denne delen bruker vi **kun vanlig Python og pandas**.
> Ingen CHAP-kode brukes før senere i tutorialen.



### 2.1 Hva betyr “isolert” i praksis?

“Isolert” betyr at:

- vi jobber kun med:
  - CSV-filer
  - pandas DataFrames
  - vanlige Python-funksjoner
- vi bruker **ikke**:
  - `chap-core`
  - `MetricBase`
  - `MetricSpec`
  - CHAP CLI

Dette gjør at:

- du kan verifisere matematikken selv
- du kan teste små endringer raskt
- du forstår *hva CHAP senere faktisk automatiserer for deg*



### 2.2 Hva er det vi bygger isolert i denne tutorialen?

I denne tutorialen bygger vi isolert:

- én enkel evalueringsmetrikk
- basert på:
  - prediksjoner (`forecast`)
  - observasjoner (`disease_cases`)
- som returnerer:
  - absolutt feil per `location` og `time_period`




### 3.3 Hvorfor er denne isolerte fasen viktig før CHAP?

Uten denne fasen risikerer du at:

- CHAP oppleves som “magisk”
- feil blir vanskelige å debugge
- du ikke vet om feilen ligger i:
  - modellen
  - metrikken
  - eller CHAP-integrasjonen

Med en isolert fase kan du alltid si:

> “Jeg vet at matematikken fungerer – hvis noe feiler nå, er det integrasjonen.”






## 3. Bakgrunn: Hvordan evaluering fungerer i CHAP

Når en modell evalueres i CHAP, produseres prediksjoner for:

- ```location``` – f.eks. et distrikt

- ```time_period``` – f.eks. en uke

- ```horizon_distance``` – hvor langt frem i tid

- ```sample``` – indeks for stokastisk sample

Prediksjoner representeres som en flat DataFrame:
```
location  time_period  horizon_distance  sample  forecast
loc1      2023-W01     1                 0       10
loc1      2023-W02     2                 0       12
```
Observasjoner har samme struktur, men uten sample:
```
location  time_period  disease_cases
loc1      2023-W01     11.0
loc1      2023-W02     13.0

```

## 4. Isolert eksempel: Implementer en enkel metrikk

Før vi bruker CHAP, starter vi alltid med et isolert eksempel.


### 4.1 Eksempel på metrikk (absolutt feil)

```python
import pandas as pd
  
forecasts = pd.DataFrame(
    {
        "location": ["loc1", "loc1", "loc2", "loc2"],
        "time_period": ["2023-W01", "2023-W02", "2023-W01", "2023-W02"],
        "horizon_distance": [1, 2, 1, 2],
        "sample": [1, 1, 1, 1],
        "forecast": [10, 12, 21, 23],
    }
)

observations = pd.DataFrame(
    {
        "location": ["loc1", "loc1", "loc2", "loc2"],
        "time_period": ["2023-W01", "2023-W02", "2023-W01", "2023-W02"],
        "disease_cases": [11.0, 13.0, 19.0, 21.0],
    }
)

def my_metric(forecasts, observations):
    merged = forecasts.merge(observations, on=["location", "time_period"])
    merged["metric"] = (merged["forecast"] - merged["disease_cases"]).abs()
    return merged[["location", "time_period", "metric"]]

print(my_metric(forecasts, observations))
```

### 4.2 Forventet output
```
location time_period  metric
loc1     2023-W01     1.0
loc1     2023-W02     1.0
loc2     2023-W01     2.0
loc2     2023-W02     2.0
```

Dette betyr:

- Modellen traff med feil på ±1 og ±2

- Metrikken beregnes per location og time_period


## 5. Overgang til CHAP-kompatibel metrikk (konseptuelt)

Når metrikken fungerer isolert, pakkes den inn i CHAP ved å:

- Lage en klasse som arver fra MetricBase

- Implementere ```compute()```

- Definere en ```MetricSpec``` som forteller:

    - hvilke dimensjoner som returneres

    - navn og id på metrikken


**Viktig poeng:**

Vi endrer ikke matematikken, bare hvordan den pakkes inn.

## 6. Startpunkt: Klon `minimalist_example` og jobb isolert der først

I denne tutorialen starter vi **ikke fra et tomt prosjekt**.  
Vi starter med å **klone `minimalist_example`**, og bruker dette repoet som:

- grunnstruktur
- arbeidsmappe
- og senere som inngang til CHAP

Repoet ligger her:

https://github.com/dhis2-chap/minimalist_example



### 6.1 Klon repoet

```bash
git clone https://github.com/dhis2-chap/minimalist_example
cd minimalist_example
```


Dette repoet inneholder allerede:

- `train.py` – treningskode

- `predict.py` – prediksjonskode

- `MLproject` – koblingen til CHAP

- eksempeldata

Men:
**I starten skal du ignorere CHAP-delen helt``

### 6.2 Første fase: Bruk repoet kun som isolert prosjekt (uten CHAP)

I denne fasen skal du:

- bruke train.py og predict.py som vanlig Python-kode

- lese og skrive CSV-filer manuelt

- teste logikken med python train.py og python predict.py

- implementere og teste metrikken din isolert

- **ikke bruke CHAP**

- **ikke tenke på MLproject**

Poenget er at:

 Før modellen og metrikken fungerer helt isolert, skal du ikke bruke CHAP

### 6.3 Hvor passer metrikken du laget isolert inn?

Når du har laget denne isolerte metrikken:

```python
def my_metric(forecasts, observations):
    ...
```

bruker du den nå:

- direkte på CSV-output fra predict.py

- for å verifisere at:

  - prediksjoner gir mening

  - feilen oppfører seg som forventet

Dette skjer fortsatt uten CHAP.


## 7. Viktige regler

- Du skal alltid klone minimalist_example først

- Du skal alltid jobbe isolert i repoet først

- Du skal først bruke CHAP når alt fungerer isolert

- Du skal aldri starte direkte med CHAP

- All installasjon skal skje via:

```
uv sync -r requirements.txt
```

Ikke manuelt med pip install.


## 8. Neste steg etter denne tutorialen

Når denne tutorialen er gjennomført, er du klar for å:

- implementere en egen metrikk

- teste den isolert i repoet

- føre den inn i CHAP som MetricBase

- kjøre den via ```MLproject``` og CHAP

bruke den videre i DHIS2 Modeling App

Dette er en komplett utviklingssyklus:
idé → isolert test → CHAP-core.


