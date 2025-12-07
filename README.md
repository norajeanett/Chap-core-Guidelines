# Guidelines for Tutorials for Master Students (CHAP-Core)

Denne guiden beskriver hvordan alle tutorials for masterstudenter som jobber med CHAP-Core skal utformes.

Målet er at alle tutorials skal være:

- Reproduserbare  
- Trygge å kjøre på en helt blank PC  
- Pedagogiske  
- Konsistente på tvers av prosjekter og semestre  
- Bygget på en tydelig progresjon:  
  **isolert eksempel → CHAP-integrasjon → (eventuelt) DHIS2**

---

## 1. Målgruppe

Alle tutorials er skrevet for:

- Masterstudenter i informatikk, data science, maskinlæring eller helseinformatikk  
- Studenter som:
  - Kan Python på grunnleggende nivå  
  - Har lite eller ingen erfaring med CHAP-Core fra før  

Tutorials skal **ikke** anta:

- At studenten kjenner DHIS2 fra før  
- At studenten har jobbet med CHAP tidligere  
- At studenten har spesifikke biblioteker installert  

---

## 2. Obligatorisk struktur for alle tutorials

Alle tutorials **MÅ** inneholde:

1. **Tittel**
2. **Kort introduksjon**
3. **Læringsmål**
4. **Forutsetninger**
5. **Oppsett på blank PC**
6. **Kjørbar demo**
7. **Forklaring av output**
8. **Feilsøking**
9. **Neste steg**

---

## Bli kjent med Chap Core with DHIS2 Modeling App

https://dhis2-chap.github.io/chap-core/modeling-app/index.html

## 3. Pedagogisk hovedprinsipp: Isolert eksempel før CHAP

Alle tutorials som involverer modeller **SKAL** følge denne progresjonen:

1. **Først: Et isolert eksempel**
   - Modellen kjøres uten CHAP
   - Studentene skal forstå:
     - hva input er
     - hva modellen gjør
     - hva output betyr
   - Dette gir studenten noe *konkret og fungerende* før kompleksiteten økes

2. **Deretter: Integrasjon med CHAP-Core**
   - Samme modell kobles til CHAP
   - Studentene lærer:
     - hvordan CHAP kaller modellen
     - hvordan train/predict styres via CHAP
     - hvordan output standardiseres

3. **Til slutt (valgfritt): Integrasjon med DHIS2**
   - Kun når studenten forstår både modellen og CHAP-pipelinen

 **Ingen student skal møte CHAP før de har sett et isolert, fungerende eksempel.**

---

## 4. Virtuelt miljø er OBLIGATORISK

Alle tutorials **SKAL** bruke:


python -m venv venv
source venv/bin/activate   # Windows: venv\Scripts\activate

## 5. requirements.txt er OBLIGATORISK

Alle tutorials SKAL ha en requirements.txt.

Det er ikke lov å skrive:

pip install chap-core
pip install pandas
pip install numpy


**Riktig måte:**
pip install -r requirements.txt


**Minimum requirements.txt for CHAP-tutorials:**
chap-core
pandas
numpy
pyyaml
scikit-learn

Dette sikrer:

- Samme versjoner for alle studenter

- Reproduserbare resultater

- Færre installasjonsfeil

---
## Bruk av minimalist_example (CHAP-integrasjon)

minimalist_example skal brukes som:

- Referanse for hvordan en modell integreres i CHAP-Core

- Eksempel på MLproject + train / predict-struktur

- Bro mellom ren ML-kode og CHAP-pipeline

Men:

- minimalist_example skal ikke være eneste oppsett

- Tutorials kan ha egne datasett og egne modeller i tillegg


Typisk progresjon for studentene:

1. Først: Kjøre modell uten CHAP

2. Så: Integrere modellen via minimalist_example-struktur

3. Til slutt: Koble til DHIS2 Modeling App

**Link til repo: https://github.com/dhis2-chap/minimalist_example**

---

## Tutorials skal:

- Forklare hva som skjer, ikke bare hva man skriver

- Ha korte, oversiktlige avsnitt

- Bruke kodeblokker for all kode

- Forklare nye begreper første gang de brukes

- Aldri hoppe over “selvsagte” steg
