# Asistent za pretragu uputstava za lekove na srpskom

[English](../../README.md) | **Srpski**

Seminarski projekat kombinuje RAG pretragu uputstava za lekove na srpskom jeziku i QLoRA fino podešavanje modela Llama 3.1 8B Instruct. Repo sadrži zajednički kod, podatke za obuku, indeks za pretragu i radove na srpskom i engleskom jeziku.

Aplikacija je namenjena srpskom jeziku. Englesko izdanje predstavlja dokumentaciju i rad za međunarodne čitaoce i recenzente; oba izdanja koriste isti kod i model. Komentari, upiti i izlazi eksperimenata zadržavaju izvorni jezik.

## Radovi i resursi

| Resurs | Opis |
| --- | --- |
| [Seminarski rad na srpskom — PDF](seminarski_rad.pdf) | Ranije predati rad, sačuvan bez izmena |
| [Rad na engleskom — PDF](../en/paper.pdf) | Revidirana verzija za pregled, ažurirana 23. oktobra 2026. |
| [Kratak pregled projekta na engleskom](../../summary.md) | Implementacioni doprinos, tehničke odluke, evaluacija i naredni koraci |
| [Model na Hugging Face-u](https://huggingface.co/vekiii/medical-assistent-serbian-model) | Lokacija modela |
| [Eksperimentalni Space](https://huggingface.co/spaces/vekiii/medical-assistant-serbian) | Dostupnost zavisi od okruženja i hardverske kvote |

Engleski rad sadrži ispravke referenci, tehničkih opisa i tumačenja rezultata, pa nije doslovan prevod arhiviranog srpskog rada. Numerički rezultati i citirani izlazi modela sačuvani su. Engleski rad ima 17 fusnota, a srpski 18. U repou se radovi objavljuju kao PDF fajlovi; Word dokumenti isključeni su iz Git-a.

## Status i ograničenja

Ovo je istraživački prototip. Primeri pokazuju činjenične greške, nepotpune odgovore, zamenu lekova, nedosledno korišćenje pisma i nepoštovanje instrukcija. Rezultati ne dokazuju kliničku pouzdanost niti spremnost za samostalnu medicinsku upotrebu.

Praktični doprinos je implementacija sistema uz ograničene GPU resurse i analiza neuspešnih primera. Numerička evaluacija obuhvata samo **četiri pitanja**:

- Relevantnost pretrage određuje se prema imenu izvornog PDF-a. Isečak iz odgovarajućeg uputstva može se računati kao relevantan i kada ne sadrži odgovor.
- Pitanje o Pressingu sadrži „sirup” u evaluaciji pretrage, ali ne i u evaluaciji generisanja. To nisu identični ulazi.
- Semantička sličnost nije dokaz činjenične tačnosti. Podrazumevana ROUGE tokenizacija u notebooku nije prilagođena srpskim pismima.
- Poređenje osnovnog modela sa celim sistemom ne razdvaja doprinos RAG-a, finog podešavanja i kvantizacije.
- **591 ulazno polje sadrži doslovni string `nan`**, koji formatter uključuje u obuku. Podaci i rezultati sačuvani su radi dokumentovanja originalnog eksperimenta; te vrednosti treba očistiti pre nove obuke.

## Struktura repoa

```text
.
├── README.md                         # Englesko uputstvo
├── docs/
│   ├── en/paper.pdf                  # Revidirani engleski rad
│   └── sr/
│       ├── README.md                 # Ovo uputstvo
│       └── seminarski_rad.pdf        # Originalni srpski rad
├── notebooks/                       # Zajednička implementacija
├── dataset_LoRA/                    # SQuAD i objedinjeni skup za obuku
├── vector_database/                 # index.faiss i index.pkl
└── .gitignore
```

Lokalni direktorijumi `dataset_RAG/`, `lora_adapters/` i `merged_model/` isključeni su iz Git-a. Izvorna 54 PDF uputstva i težine modela nisu deo ovog Git repoa. FAISS indeks i njegovi metapodaci jesu uključeni. `final_train_dataset.json` koristi JSON Lines format uprkos nastavku `.json`.

## Implementacija i podaci

1. **Dokumenti:** ALIMS uputstva pretvaraju se u Markdown pomoću `pymupdf4llm`, a `MarkdownHeaderTextSplitter` deli ih prema zaglavljima. Sačuvani eksperiment proizveo je 1.769 isečaka.
2. **Pretraga:** `sentence-transformers/paraphrase-multilingual-mpnet-base-v2` daje vektore dimenzije 768. Uprkos nazivu, ovaj višejezični checkpoint koristi XLM-RoBERTa osnovu. Kod koristi podrazumevanu FAISS pretragu po euklidskom rastojanju, bez eksplicitne normalizacije vektora, i pet isečaka po upitu u aplikaciji.
3. **Prilagođavanje:** 3.152 opšta primera instrukcija i odgovaranja na pitanja na srpskom koriste se za QLoRA obuku, a ne namenski medicinski skup. Parametri uključuju 4-bitni NF4, dvostruku kvantizaciju, Float16 računanje, rang 16, alpha 32 i jednu epohu od 394 koraka.
4. **Generisanje:** objedinjeni model učitava se u 8-bitnoj preciznosti. Sačuvana konfiguracija koristi `do_sample=False` i granicu od 512 novih tokena. Istorija razgovora koristi se za reformulisanje upita; prompt za odgovor dobija trenutno pitanje i pronađeni tekst.
5. **Instrukcije:** prompt traži oslanjanje na kontekst, praćenje pisma korisnika, jednostavniji jezik i upozorenje na kraju. To nisu nezavisno sprovedene garancije. Metapodaci se čuvaju uz isečke, ali se ne dodaju eksplicitno u prompt za generisanje.

Skup za obuku kombinuje po 500 ćiriličnih i latiničnih [SQuAD primera](https://www.kaggle.com/datasets/aleksacvetanovic/squad-sr), 1.000 ćiriličnih primera iz [saillab Alpaca skupa](https://huggingface.co/datasets/saillab/alpaca-serbian-cleaned), 1.000 latiničnih iz [datatab Alpaca skupa](https://huggingface.co/datasets/datatab/alpaca-cleaned-serbian-full) i 152 srpska primera iz [Aya skupa](https://huggingface.co/datasets/CohereLabs/aya_dataset). Prema pismu instrukcija, odnos je 52,3% ćirilice i 47,7% latinice.

## Pokretanje projekta

Notebookovi su ulazne tačke za izvršavanje. Razvijani su uz Google Colab i Google Drive; projekat nije upakovana aplikacija sa jednom komandom za pokretanje. Zavisnosti se instaliraju u ćelijama. Ne postoji fiksirano i nezavisno provereno okruženje, pa novo pokretanje može zahtevati usklađivanje biblioteka.

### Preduslovi i putanje

- Za obuku i inferenciju koristi GPU okruženje. Opisani trening koristi NVIDIA T4 sa približno 15–16 GB VRAM-a. Spajanje modela zahteva i dovoljno sistemskog RAM-a i prostora na disku.
- Obezbedi pristup osnovnom Llama modelu i autentifikuj se sopstvenim Hugging Face nalogom kada je potrebno. Tokene nemoj čuvati u verzionisanim ćelijama ili izlazima notebookova.
- Poveži Google Drive. Podrazumevana projektna putanja je `/content/drive/MyDrive/NLP_proyect`.
- U `dataset_LoRA.ipynb` postavi `base_path` na direktorijum sa dva SQuAD JSON fajla; originalna vrednost pokazuje na lokalni Windows keš.
- Trening notebook očekuje `datasets/final_train_dataset.json`, a kreiranje indeksa `datasets/dataset_RAG` unutar Drive projekta. Repo i originalna Drive struktura razlikuju se: kopiraj ulaze na očekivane lokacije ili promeni promenljive putanja.
- Uskladi `db_path`, `model_id`/`model_path`, putanje adaptera i izlazne direktorijume. Aplikacija zahteva kompatibilan objedinjeni model i indeks. Za Hugging Face model postavi odgovarajući identifikator umesto podrazumevane Drive putanje.

### Redosled notebookova

| Korak | Notebook | Namena |
| --- | --- | --- |
| 1 | [dataset_LoRA.ipynb](../../notebooks/dataset_LoRA.ipynb) | Uzorkovanje izvora i formiranje objedinjenog JSONL fajla |
| 2 | [final_train_dataset_EDA.ipynb](../../notebooks/final_train_dataset_EDA.ipynb) | Provera praznih vrednosti, dužina, pisma i učestalih pojmova |
| 3 | [LLM_finetuning.ipynb](../../notebooks/LLM_finetuning.ipynb) | Obuka adaptera i spajanje sa osnovnim težinama po delovima |
| 4 | [vector_database_creation.ipynb](../../notebooks/vector_database_creation.ipynb) | Kreiranje FAISS indeksa iz lokalnih PDF uputstava |
| 5 | [app.ipynb](../../notebooks/app.ipynb) | Učitavanje modela i indeksa i pokretanje razgovora |
| 6 | [evaluation.ipynb](../../notebooks/evaluation.ipynb) | Evaluacija pretrage i odgovora na izolovana pitanja |

Kreiranje indeksa nezavisno je od obuke. Uz postojeći model i indeks možeš početi od `app.ipynb` nakon podešavanja zavisnosti i putanja. Sačuvani indeks omogućava inferenciju bez ponovne obrade PDF-ova. Učitavaj pickle metapodatke samo iz pouzdanog izvora; notebook eksplicitno dozvoljava deserijalizaciju.

Izvršavaj ćelije redom i resetuj `chat_history` između nezavisnih razgovora. Dokumentovani interfejs je izlaz notebooka; Gradio importi i Space link ne znače da je potvrđen stabilan produkcioni interfejs. Novi rezultati, naročito posle čišćenja podataka ili promene biblioteka, mogu se razlikovati od objavljenih.

## Dalji razvoj

Prvi planirani eksperiment je **osnovni model + RAG naspram fino podešenog modela + identičan RAG**. Treba koristiti ista pitanja, promptove, pronađene isečke, preciznost modela i parametre generisanja, uz resetovanje istorije između nezavisnih slučajeva. Porediti činjeničnu tačnost, potpunost, kvalitet srpskog jezika i praćenje instrukcija. Ovo kontrolisano poređenje još nije sprovedeno; potrebno je da bi se izdvojio doprinos dodatnog finog podešavanja. Zasebno poređenje celih sistema zatim može obuhvatiti uticaj modela na reformulisanje upita iz istorije razgovora i samu pretragu.

Prioriteti su veći nezavisno anotiran test skup, stručna procena činjenične tačnosti, kontrolisano poređenje četiri kombinacije osnovnog/fino podešenog modela sa/bez RAG-a, metrike prilagođene pismima i čišćenje podataka. Dalji eksperimenti mogu obuhvatiti usklađivanje promptova za obuku i inferenciju, hibridnu pretragu, eksplicitno prosleđivanje metapodataka, dodatno rangiranje i promenljivu dubinu pretrage.

## Licenciranje

Repo trenutno nema deklarisanu zajedničku licencu. Model, skupovi podataka i izvorni dokumenti imaju sopstvene uslove korišćenja; njihovo prisustvo ili linkovanje ne zamenjuje te uslove.
