Kutatási Módszertan és Alkalmazott eszközök
Feladat: Képfelbontás, pontosság és számítási költség vizsgálata MedMNIST adatokon (28, 64, 128, 224 pixel).

1. Modell: ResNet-18
Miért ResNet-18?
Ez a hivatalos MedMNIST baseline-modell, így összehasonlíthatók a mért eredmények.
Könnyű és gyors: 36 kísérlet (3 adat × 4 méret × 3 seed) Colabban is lefut vele pár óra alatt.
A belső szerkezete rögzített marad minden felbontáson, így tisztán a képméret hatását mérjük.
Input réteg (stem) szabály:
28×28-nál a kezdő konvolúciót kisméretűre ( 3×3 , stride=1) állítjuk, max-pooling nélkül, hogy ne veszítsen azonnal információt.
A többi méretnél a standard beállítást használjuk.
2. Tervezett Adathalmazok
A hatalmas méretű adatok helyett 3 kisebb, de valamennyire mégis eltérő méretü és eltérő jellegű adathalmazt használunk, hogy a tanitás ne tartson 20 órán át:

Adathalmaz	Terület / Típus	Csatorna	Képek száma	Miért választottuk?
PneumoniaMNIST	Tüdőröntgen (bináris)	1 (szürke)	~5 800	Nagy anatómiai struktúrák, szürkeárnyalatos
DermaMNIST	Bőrelváltozás (7 osztály)	3 (RGB)	~10 000	Közepes méret, színes kép
BloodMNIST	Vérsejtek (8 osztály)	3 (RGB)	~17 000	Finom mikroszkopikus részletek
3D adatok: Kihagyva, mert a köbös számítási igény nem tudjuk biztositani.

3. Szoftverek és Eszközök
PyTorch (torch, torchvision): Fő keretrendszer. A hivatalos medmnist Python API natív torch Dataset objektumokat ad vissza.
medmnist: Automatikus letöltés a hivatalos forrásból, api-n keresztül (Zenodo) a 4 felbontásban, fix felosztással.
fvcore: Elméleti számítási igény (FLOPs / MACs) pontos mérése.
scikit-learn: Hivatalos AUC és ACC számítás.
pandas: Mérési eredmények mentése táblázatos formában (CSV).
4. Konfiguráció: JSON
Minta beállítás (config.json):

{
  "dataset": "pneumoniamnist",
  "resolutions": [28, 64, 128, 224],
  "seeds": [42, 43, 44],
  "epochs": 25,
  "batch_size": 128,
  "learning_rate": 0.001,
  "amp": true,
  "patience": 5
}
Kimeneti nyers eredmény minta:

{
  "config": {
    "dataset": "pneumoniamnist",
    "resolution": 64,
    "seed": 43,
    "epochs": 25,
    "batch_size": 128,
    "amp": true
  },
  "environment": {
    "device": "cuda",
    "gpu_name": "Tesla T4",
    "pytorch_version": "2.1.0+cu121",
    "cuda_version": "12.1"
  },
  "predictive_metrics": {
    "test_auc": 0.9415,
    "test_acc": 0.8842,
    "best_val_auc": 0.9480
  },
  "compute_metrics": {
    "flops": 450000000,
    "latency_ms_per_image": 1.82,
    "throughput_fps": 549.4,
    "peak_vram_mb": 412.5,
    "total_train_time_s": 84.3
  }
}
A használt erőforrásokat, majd valahova beillesztjük.

5. Mérések és Elemzés
Hardver és idő:
FLOPs: Kiszámolva fvcore-ral.
Latency (idő/kép): 50 futtatás bemelegítés (warm-up) után mérve, medián értékkel.
Csúcs GPU-memória: torch.cuda.max_memory_allocated().
Knee-point (könyökpont):
Az a legkisebb felbontás, ahol a felbontás növelése már nem hoz legalább +0,5% AUC javulást.
