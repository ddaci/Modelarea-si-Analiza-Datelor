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

**Software utilizat:**  
- **Altair AI Studio:** pentru curățarea și transformarea datelor.  
- **Python (pandas, numpy):** pentru procesarea fișierelor CSV și codificarea atributelor.  

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
    
### 6. Concluzie - o clasificare binară ar fi adus o mai bună performanță a modelului.

**Software utilizat:**  
- **Altair AI Studio:** pentru crearea și optimizarea arborelui de decizie.  
- **Python (scikit-learn):** pentru analiza performanței modelului.  

---

## III. Clustering - Explainable k-Means Clustering (ExKMC)

### 1. Descrierea metodei
Pentru segmentarea joburilor, am utilizat metoda **Explainable k-Means Clustering (ExKMC)**, care combină avantajele algoritmului *k-Means* cu interpretabilitatea oferită de arborii de decizie. Această metodă rezolvă problema interpretabilității algoritmilor tradiționali de clustering, oferind reguli clare și intuitive pentru fiecare cluster generat.

Deoarece ExKMC nu este integrat nativ în Altair AI Studio, am simulat procesul printr-un **workflow structurat**, urmând acești pași:

1. **Selecția atributelor relevante:**  
   - `Salary Range To` – pentru diferențierea joburilor în funcție de salariu.  
   - `Job Category` – pentru gruparea pe domenii de activitate.  
   - `Posting Type` – indică dacă jobul este *Internal* sau *External*.  
   - `# Of Positions` – pentru a diferenția joburile individuale de cele multiple.  
   - `Career Level` – nivelul de experiență necesar pentru job.  
   - `Education` – nivelul educațional necesar pentru fiecare poziție.  

2. **Prelucrarea datelor:**  
   - Am creat coloana `Job_Domain` pentru a grupa joburile în domenii largi:  
     - *Social & Health Services*, *Technical Services*, *Finance & Administration*, *Public Safety & Legal*, *Policy & Research*.  
   - Am generat coloana `Salary Level` pe baza pragurilor definite:  
     - *Low* (40k-80k), *Medium* (80k-120k), *High* (>120k).  
   - Am creat coloana `Education` extrăgând nivelul de educație din cerințele postului.  

3. **Codificarea atributelor:**  
   - *Ordinal Encoding*: pentru date ordinale (ex.: `Career Level`, `Salary Level`).  
   - *One-Hot Encoding*: pentru date categorice fără ordine (`Job Domain`, `Education`).  
   - Am aplicat *Z-Score Standardization* pentru scalarea atributului `# Of Positions` și am eliminat valorile extreme (>150), pentru a evita distorsionarea clustering-ului.

---

### 2. Aplicarea algoritmului k-Means
Pentru clustering am utilizat algoritmul k-Means, testând diferite valori pentru *k*: **6**, **8**, **10**, și **15**, evaluând următorii indicatori:

- **1. Avg. Within Centroid Distance (Compactitatea clusterelor):**  
  Măsoară cât de apropiate sunt punctele din cluster față de centroid.  
  - *k=6* a avut cea mai mare valoare, indicând clustere mai largi și mai puțin compacte.  
  - *k=8* a oferit un echilibru optim între compactitate și dimensiunea clusterelor.  
  - *k=15* a dus la supraîmpărțirea datelor.  

- **2. Davies-Bouldin Index (Separarea și coeziunea clusterelor):**  
  Valorile mai mici indică clustere bine separate și compacte.  
  - *k=8* a avut cel mai mic index (-1.466), indicând separarea optimă a clusterelor.  
  - *k=10* și *k=15* au prezentat o deteriorare a separării.  

- **3. Example Distribution (Uniformitatea distribuției datelor între clustere):**  
  Valorile apropiate de 1 indică distribuție uniformă.  
  - *k=6* a avut cea mai uniformă distribuție, dar clusterele erau prea generale.  
  - *k=8* a oferit un echilibru între distribuție și granularitatea clusterelor.  

**Recomandare finală:** *k=8*, datorită echilibrului între compactitate, separare și distribuție uniformă.

---

### 3. Interpretarea clusterelor cu arbori de decizie
Pentru a explica clusterele generate, am aplicat un **arbore de decizie**, obținând următoarele grupuri semnificative:

1. **Cluster 1:**  
   - *# Of Positions > 7.030* – Joburi cu cerere foarte mare, specifice rolurilor operaționale sau logistice.  
   - *Exemple*: 24 poziții.  

2. **Cluster 0:**  
   - *Career Level > 1.5 și # Of Positions ≤ 7.030* – Roluri avansate, senior-level, cu cerințe educaționale ridicate.  
   - *Exemple*: 893 poziții.  

3. **Cluster 7 (Finance & Administration):**  
   - *Career Level ≤ 1.5 și # Of Positions > 1.479* – Joburi financiare și administrative la nivel intermediar.  
   - *Exemple*: 26 poziții.  

4. **Cluster 2 (Entry-Level Finance & Admin):**  
   - *Education: No formal education specified* – Roluri de bază, accesibile fără studii superioare.  
   - *Exemple*: 1062 poziții.  

5. **Cluster 5 (Technical Services):**  
   - *Salary Level > 1.5* – Joburi tehnice bine plătite, în inginerie, IT, proiectare.  
   - *Exemple*: 690 poziții.  

6. **Cluster 3 (Entry-Level Technical Services):**  
   - *Education: High School Diploma* – Roluri tehnice accesibile, cu salarii mai mici.  
   - *Exemple*: 612 poziții.  

7. **Cluster 4 (Social & Health Services):**  
   - *Education: No formal education specified* – Joburi entry-level în domeniul sănătății.  
   - *Exemple*: 1495 poziții.  

8. **Cluster 6 (Public Safety & Legal):**  
   - *Salary Level > 1.5 și Career Level ≤ 1.5* – Roluri cu responsabilități ridicate și salarii peste medie.  
   - *Exemple*: 529 poziții.  

---

### 4. Concluzii clustering ExKMC
1. **Pentru viitorii angajați:**  
   - *Entry-Level:* Cluster 2 (*Finance & Admin*) și Cluster 6 (*Health Services*).  
   - *Mid-Level:* Cluster 7 (*Finance & Admin*, cu cerere moderată).  
   - *Senior-Level:* Cluster 0 (*Technical Services*, salarii mari).  

2. **Pentru organizații guvernamentale:**  
   - **Cluster 1:** Campanii de recrutare pentru joburile cu cerere mare (>7.000 poziții).  
   - **Cluster 0:** Promovarea angajaților și atragerea de talente senior.  
   - **Cluster 5:** Sprijinirea educației STEM pentru joburile tehnice bine plătite.  

---


Utilizarea ExKMC a permis nu doar gruparea joburilor în funcție de asemănările lor, ci și explicarea logică a acestor grupuri. Această interpretabilitate ajută atât candidații să identifice oportunitățile potrivite, cât și organizațiile să optimizeze procesele de recrutare și formare.


## IV. Vizualizarea datelor și concluzii
![maddmpbi](https://github.com/user-attachments/assets/8e274431-c9b1-4727-a4e2-eacdb6edae10)

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

**Software utilizat:**  
- **Power BI:** pentru vizualizarea tendințelor și analizarea distribuțiilor.  
- **Python (pandas, matplotlib):** pentru procesarea datelor 

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

## VI. Software utilizat în proiect
1. **Python:** pentru curățarea și prelucrarea datelor (`pandas`, `numpy`).  
2. **Altair AI Studio:** pentru pregătirea datelor, clasificare și clustering.  
3. **RapidMiner:** pentru procese automate de curățare și modelare.  
4. **Power BI:** pentru vizualizarea datelor și analiza interactivă.  


---

Acest proiect demonstrează cum analiza avansată a datelor poate sprijini deciziile de management în domeniul ocupării forței de muncă din New York City, oferind perspective atât pentru candidați, cât și pentru angajatori.
