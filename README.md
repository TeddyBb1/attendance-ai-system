<div align="center" style="padding: 20px 20px 10px; border-radius: 16px; background: linear-gradient(135deg, #111827, #020617); color: #fff;">

  <!-- Banner proiect (înlocuiește docs/banner.gif cu ce vrei tu) -->
  <img src="docs/banner.png" alt="Face Attendance AI System Banner" style="max-width: 100%; border-radius: 14px; margin-bottom: 18px; box-shadow: 0 18px 40px rgba(0,0,0,0.45);">

  <h1 style="font-size: 40px; margin-bottom: 0.2em;">📘 Etapa 3 – Analiza & Preprocesarea Datelor</h1>
  <h3 style="margin-top: 0;">Proiect: <span style="color:#38bdf8;">Face Attendance AI System</span></h3>

  <!-- Badges -->
  <p>
    <img src="https://img.shields.io/badge/University-POLITEHNICA%20Bucure%C8%99ti-blue" alt="UPB Badge">
    <img src="https://img.shields.io/badge/Disciplin%C4%83-Re%C8%9Bele%20Neuronale-7c3aed" alt="RN Badge">
    <img src="https://img.shields.io/badge/Stage-Etapa%203-success" alt="Stage 3 Badge">
    <img src="https://img.shields.io/badge/Project-Face%20Attendance%20AI-orange" alt="Project Badge">
  </p>

</div>

<br>

<div style="padding: 18px; margin-bottom: 25px; background: #f4f7fe; border-left: 5px solid #4f46e5; border-radius: 8px;">
  <strong>Student:</strong> Baba Cristian-Teodor<br>
  <strong>Disciplina:</strong> Rețele Neuronale – FIIR<br>
  <strong>Scopul etapei:</strong> Pregătirea unui set de date curat, standardizat și bine documentat pentru antrenarea rețelelor neuronale de recunoaștere facială.
</div>

---

## 🧭 1. Structura Repository-ului (Etapa 3)

<div style="padding: 20px; border-radius: 12px; background:#020617; color:#e5e7eb; font-family: Consolas, 'Fira Code', monospace; font-size: 14px;">
<pre>
face-attendance-ai/
├── README.md
├── docs/
│   └── datasets/
│        ├── dataset_description.md
│        ├── face_pipeline_diagram.png
│        └── sources.md
├── data/
│   ├── raw/               # imagini brute
│   ├── cleaned/           # imagini curățate (crop față, normalizare)
│   ├── embeddings/        # vectori 128D generați de RN
│   ├── train/             # set de antrenare
│   ├── validation/        # set de validare
│   └── test/              # set de testare final
├── src/
│   ├── preprocessing/     # pipeline de preprocesare
│   ├── face_extraction/   # YOLO + decupare față
│   ├── embedding_model/   # generare embeddings
│   └── utils/             # funcții auxiliare
├── config/
│   └── preprocessing.yaml # parametri & setări de preprocesare
└── requirements.txt
</pre>
</div>

---

## 🖼️ 2. Descrierea Setului de Date

<div style="display:flex; gap:20px; flex-wrap:wrap; margin: 25px 0;">

  <div style="flex:1; min-width:260px; padding:20px; background:#f8fafc; border-radius:12px; border:1px solid #d1d5db; transition: transform .25s ease, box-shadow .25s ease;">
    <h3>📍 Sursa Datelor</h3>
    <ul>
      <li>Imagini generate cu AI (control asupra condițiilor)</li>
      <li>Upload manual prin interfața web</li>
      <li>Capturi YOLO din fluxul camerei (live)</li>
    </ul>
    <p><strong>Scop:</strong> Antrenarea și testarea unui sistem de prezență bazat pe recunoaștere facială.</p>
  </div>

  <div style="flex:1; min-width:260px; padding:20px; background:#f8fafc; border-radius:12px; border:1px solid #d1d5db; transition: transform .25s ease, box-shadow .25s ease;">
    <h3>📊 Dimensiunea Dataset-ului</h3>
    <ul>
      <li>~200–300 imagini brute</li>
      <li>10–15 persoane distincte</li>
      <li>Format: JPG / PNG</li>
      <li>Rezoluție finală: 224×224 px</li>
    </ul>
    <p>Imaginile sunt organizate pe persoane, cu un număr relativ echilibrat de exemple per identitate.</p>
  </div>

</div>

---

### 2.1 Caracteristici Numerice & Metadate

Embeddings și metadate generate după preprocesare:

| Caracteristică | Tip      | Dimensiune | Descriere                                            |
|----------------|----------|------------|------------------------------------------------------|
| `embedding`    | numeric  | 128        | Vector RN pentru fiecare față (spațiul de featuri)  |
| `confidence`   | numeric  | scalar     | Scor YOLO de încredere că este față validă          |
| `bbox`         | numeric  | 4 valori   | Coordonatele dreptunghiului de detecție (x1,y1,x2,y2) |
| `person_id`    | categ.   | scalar     | ID unic al persoanei (label de clasă)               |

> Documentația detaliată se află în `docs/datasets/dataset_description.md`.

---

## 🔍 3. Analiza Exploratorie a Datelor (EDA)

<details>
  <summary><strong>🔎 Deschide analiza EDA (click aici)</strong></summary>

<br>

### 3.1 Statistici Descriptive

- Distribuție imagini / persoană
- Număr mediu de imagini per individ
- Verificare iluminare, blur, poziția feței
- Rata de detecție YOLO (confidence > prag)

Exemple de observații:

- Persoanele au între 10 și 25 imagini fiecare.
- ~8% din imagini au confidence < 0.7 (considerate problematice).
- Câteva imagini conțin fețe parțial acoperite sau orientări extreme.

---

### 3.2 Analiza Calității Datelor

- Identificarea imaginilor:
  - fără față detectată
  - cu fețe multiple
  - cu detecție slabă (sub un prag stabilit, ex. 0.7)
- Identificarea imaginilor neclare (blur vizibil)
- Compararea distribuției fețelor între persoane

---

### 3.3 Probleme Identificate

- Iluminare neuniformă → necesară normalizarea / augmentarea.
- Dezechilibru: unele persoane au mai puține imagini față de altele.
- ~12 imagini conțin mai multe fețe → ele au fost fie filtrate, fie tratate separat.

</details>

---

## 🔧 4. Preprocesarea Datelor

<details>
  <summary><strong>🧼 4.1 Curățarea datelor (click pentru detalii)</strong></summary>

<br>

<div style="background:#fff7e6; padding:20px; border-left:5px solid #f59e0b; border-radius:8px;">
✔ Eliminare imagini duplicate<br>
✔ Eliminare imagini fără față detectată de YOLO<br>
✔ Eliminare imagini cu mai multe fețe (dacă nu au putut fi separate corect)<br>
✔ Conversie la format uniform (JPG, aceeași rezoluție de bază)<br>
</div>

</details>

---

<details>
  <summary><strong>✨ 4.2 Transformarea caracteristicilor (click pentru detalii)</strong></summary>

<br>

<div style="display:flex; gap:20px; flex-wrap:wrap;">

  <div style="flex:1; min-width:270px; padding:20px; border-radius:10px; background:#020617; color:#e5e7eb; transition: transform .25s ease, box-shadow .25s ease;">
    <h3>🧠 Embedding RN</h3>
    <p>Fiecare față validă este trecută printr-un model de rețea neuronală pentru a genera un vector de featuri de dimensiune 128. Acest vector este baza comparației între persoane.</p>
  </div>

  <div style="flex:1; min-width:270px; padding:20px; border-radius:10px; background:#0f172a; color:#e5e7eb; transition: transform .25s ease, box-shadow .25s ease;">
    <h3>📏 Normalizare & Crop</h3>
    <ul>
      <li>Decupare față utilizând bounding box-ul YOLO</li>
      <li>Redimensionare la 224×224 px</li>
      <li>Normalizare valori pixel (ex: [0,1])</li>
    </ul>
    <p>Aceste transformări asigură consistența imaginilor înainte de antrenare.</p>
  </div>

</div>

</details>

---

<details>
  <summary><strong>📦 4.3 Împărțirea seturilor Train / Validation / Test</strong></summary>

<br>

<div style="padding:25px; background:#e3fae6; border-radius:12px; border:1px solid #bbf7d0;">
  <h3>Proporții utilizate</h3>
  <ul>
    <li>70% — <strong>Train</strong></li>
    <li>15% — <strong>Validation</strong></li>
    <li>15% — <strong>Test</strong></li>
  </ul>
  <h4>Principii respectate:</h4>
  <ul>
    <li>Stratificare pe persoană (fiecare persoană apare în toate seturile, dar cu imagini diferite)</li>
    <li>Fără scurgere de informație (no data leakage)</li>
    <li>Statisticile de normalizare se calculează DOAR pe train și se aplică pe val/test</li>
  </ul>
</div>

</details>

---

## 📁 5. Fișiere Generate în Etapa 3

<div style="padding:20px; background:#eff6ff; border-radius:12px; border:1px solid #bfdbfe;">
<ul>
  <li>📂 <code>data/raw/</code> – imagini brute (direct din sursă)</li>
  <li>📂 <code>data/cleaned/</code> – imagini cropate, normalizate, gata de embedding</li>
  <li>📂 <code>data/embeddings/</code> – vectori 128D pentru fiecare față</li>
  <li>📂 <code>data/train/</code>, <code>data/validation/</code>, <code>data/test/</code> – împărțirea finală a seturilor</li>
  <li>📂 <code>src/preprocessing/</code> – codul pipeline-ului de preprocesare</li>
  <li>📄 <code>config/preprocessing.yaml</code> – praguri, dimensiuni, setări YOLO / RN</li>
  <li>📄 <code>docs/datasets/dataset_description.md</code> – documentație detaliată a dataset-ului</li>
</ul>
</div>

---

## ✅ 6. Stare Etapă (to-do list GitHub)

- [x] Structură repository configurată pentru Etapa 3  
- [x] Dataset analizat (EDA de bază realizată)  
- [x] Date curățate și preprocesate (crop, resize, normalizare)  
- [x] Generare embeddings (vectori 128D)  
- [ ] Împărțire finală Train / Validation / Test salvată în foldere dedicate  
- [ ] Actualizare documentație în `docs/datasets/dataset_description.md`  
- [ ] Export PDF / DOC pentru predarea oficială (opțional)

---

