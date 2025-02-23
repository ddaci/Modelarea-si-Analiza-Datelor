# Modelarea și Analiza Datelor pentru Decizii de Management

## I. Pregătirea datelor

### 1. Setul de date
Setul de date utilizat este [New York City Job Dataset](https://www.kaggle.com/datasets/anoopjohny/new-york-city-job-dataset), având 6743 de intrări și 30 de coloane.

### 2. Etapele de pregătire a datelor
- **Codificare:** Fișierul CSV a fost convertit în UTF-8, eliminând caracterele incorect codificate.
- **Curățarea valorilor lipsă:** 
  - Eliminarea coloanelor `Recruitment Contact`, `Process Date` și `Work Location 1` folosind operatorul *Select Attributes*.
  - Aplicarea operatorului *Trim* pentru eliminarea spațiilor albe.
- **Transformarea tipurilor de date:** Coloana `Post Until` a fost convertită din *category* în *date*.
- **Eliminarea duplicatelor:** Au fost eliminate 79 de înregistrări duplicate.
- **Fișiere intermediare salvate:**
  - `DataPrep_1_missing_values.csv`
  - `DataPrep_2_change_type_date.csv`
  - `DataPrep_3_remove_duplicates.csv`

---

## II. Clasificare - Decision Tree

### 1. Obiectiv
Predicția nivelului salarial (`Salary Level`) ca fiind „mic”, „mediu” sau „mare”, pe baza unor atribute relevante:
- `Agency`, `Business Title`, `Job Category`, `Work Location`, `Posting Type`, `Salary Range To`.

### 2. Praguri pentru clasificare
- **Mic:** 40.000 - 80.000 USD  
- **Mediu:** 80.000 - 120.000 USD  
- **Mare:** >120.000 USD  

### 3. Procesul de clasificare
1. **Select Attributes:** Alegerea coloanelor relevante.  
2. **Filter Examples:** Eliminarea salariilor sub 10.000 USD.  
3. **Generate Attributes:** Crearea coloanei `Salary Level` bazată pe `Salary Range To`:
```python
if([Salary Range To] <= 80000, "Low", if([Salary Range To] <= 120000, "Medium", "High"))
```
4. **Set Role:** Setarea variabilei țintă ca `Salary Level`.
5. **Cross Validation:** Validare 10-fold pentru evaluarea performanței modelului.

### 4. Performanță model inițial
- **Precizie:**
  - *High:* 31.68%
  - *Low:* 49.41%
  - *Medium:* 42.91%
- **Recall:**
  - *High:* 2.66%
  - *Low:* 46.04%
  - *Medium:* 65.61%
- **Acuratețe generală:** 45.17%

### 5. Îmbunătățiri
- **Test 1:** Creșterea `Minimal Gain` de la 0.01 la 0.5:
  - Precizie *High*: 40%
  - Recall *Low*: 70.06%
  - Acuratețe totală: 40.30%

---

## III. Clustering - Explainable k-Means Clustering (ExKMC)

### 1. Atribute utilizate
- `Salary Range To`, `Job Category`, `Posting Type`, `# Of Positions`, `Career Level`, `Education`.

### 2. Etapele procesului
- **Codificare:**  
  - *Ordinal Encoding:* pentru `Career Level` și `Salary Level`.  
  - *One-Hot Encoding:* pentru `Job Domain` și `Education`.  
- **Scalare:** Normalizarea coloanei `# Of Positions` cu *Z-Score Standardization* și eliminarea valorilor extreme (>150).

### 3. Alegerea numărului optim de clustere (k)
Testare pentru k = 6, 8, 10, 15:
- **k = 8** a oferit cel mai bun echilibru între compactitate și separare:
  - *Davies-Bouldin Index:* -1.466 (cel mai mic).
  - *Distribuția exemplelor:* Uniformă și bine echilibrată.

---

## IV. Vizualizarea datelor și concluzii

![maddmpbi](https://github.com/user-attachments/assets/0f1410f5-2da0-4f2d-810c-61b174ccd9a2)


### 1. Analiza nivelelor salariale
- Majoritatea salariilor: *40k-60k USD* (1648 joburi).  
- Salarii mari (*220k-240k USD*): doar 18 poziții.  
- Domeniul tehnic cu master: salarii de ~80.000 USD.  

### 2. Distribuția joburilor postate
- Creștere semnificativă după iulie 2023, cu un vârf de 200 postări în 28 septembrie 2023.

### 3. Distribuția joburilor pe domenii
- **Social & Health Services:** 44.81% din joburi.  
- **Policy & Research:** Doar 1.53%.  
- Cele mai multe poziții de management sunt în domeniul tehnic (29.71%).

### 4. Cerințele de rezidență
- **62.65%:** Rezidență necesară în termen de 90 zile.  
- **35.46%:** Fără cerințe de rezidență.  

---

## V. Concluzii generale

### Pentru viitorii angajați
- **Entry-level:** *Finance & Administration*, *Social & Health Services*.  
- **Tehnic:** Optați pentru educație STEM pentru salarii mai mari.  
- **Siguranță publică:** Joburi stabile cu salarii bune.  

### Pentru organizații guvernamentale
- **Recrutare:** Creșterea accesibilității pentru comunități defavorizate.  
- **Formare:** Programe de certificare pentru joburile tehnice.  
- **Educație:** Investiții în educația medicală și STEM.

---

Acest proiect demonstrează cum analiza avansată a datelor poate sprijini deciziile de management în domeniul ocupării forței de muncă din New York City, oferind perspective atât pentru candidați, cât și pentru angajatori.
