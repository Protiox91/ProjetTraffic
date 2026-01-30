# ProjetTrafficDrone

## Objectif
Ce depot regroupe des scripts et ressources pour analyser des scenes de trafic vues par drone :
- detection et tracking de vehicules a partir de videos (YOLO + tracking)
- estimation de trajectoires, vitesses, accelerations et ecarts entre vehicules
- visualisation interactive des datasets highD / inD / rounD / uniD / exiD
- extraction de donnees highD pour MATLAB/SUMO

L'objectif est d'avoir un pipeline complet, de la video brute a la visualisation et aux exports de donnees.

---

## Initialisation (installation des dependances)

### 1) Creer un environnement Python
```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -U pip
```

### 2) Installer les dependances principales
```bash
python -m pip install numpy pandas matplotlib pillow opencv-python ultralytics scipy
```

Notes :
- `ultralytics` installe PyTorch (CPU par defaut). GPU optionnel si disponible.
- Pour les sous-projets dans `VisualisationGIT/`, vous pouvez aussi installer leurs requirements :
  ```bash
  python -m pip install -r VisualisationGIT/highD-dataset-master/Python/requirements.txt
  python -m pip install -r VisualisationGIT/drone-dataset-tools-master/requirements.txt
  ```

---

## Donnees et datasets

- Tous les liens vers les **datasets video** sont centralises dans `DatasetsVideo/liens.txt`.
- Des exemples de videos MP4 sont deja presents dans `DatasetsVideo/`.
- Les datasets de trajectoires (highD, inD, rounD, uniD, exiD) doivent etre places dans
  `Datasets/<nom-du-dataset>/data/` pour fonctionner avec les scripts de visualisation.
- Le dossier `ZIPs/` contient des archives (datasets ou outils) a decompresser si besoin.

---

## Utilisation (how to use) — scripts Python

### 1) `Codes/visualisation.py`
**But :** visualisation unifiee multi-datasets (highD, inD, rounD, uniD, exiD), avec trajectoires,
vitesse/acceleration et lignes de distance colorees.

**Prerequis :** placer les datasets dans un dossier qui contient des sous-dossiers `*/data`.
Le script cherche automatiquement dans la racine du depot ou dans `Datasets/`.
Vous pouvez aussi forcer un chemin avec la variable `TRAFFIC_DATASETS_ROOT`.

**Commande :**
```bash
python Codes/visualisation.py
```
Le script liste les datasets trouves, demande de choisir un dataset et un recording.

---

### 2) `Codes/visV2.py`
**But :** visualisation avancee du dataset highD (rectangles, trajectoires, distances colorees).

**Prerequis :** dataset highD dans `Datasets/highD-dataset-v1.0/data/` ou chemin specifie via
`HIGH_D_BASE_DIR`.

**Commande :**
```bash
HIGH_D_BASE_DIR="/chemin/vers/highD-dataset-v1.0/data" \
python Codes/visV2.py
```
Le recording est configure dans le script (variable `RECORDING_ID`).

---

### 3) `Codes/yolo26_car_kinematics.py`
**But :** detection + tracking (Ultralytics YOLO) pour produire une video annotee et un CSV
avec cinematique (trajectoires, vitesses, accelerations).

**Commande (exemple) :**
```bash
python Codes/yolo26_car_kinematics.py \
  --video DatasetsVideo/DJI_0025_cut1.Mp4 \
  --model yolo26s.pt \
  --out_dir Outputs
```
**Sorties :** video annotee `*_car_traj.mp4` + CSV `*_car_kinematics.csv` dans `Outputs/`.

---

### 4) `Codes/yolo26distances.py`
**But :** detection robuste par tuiles + tracker IoU/Hungarian + calcul de gaps/THW.
Genere video annotee + CSV de cinematique + CSV de gaps.

**Commande (exemple) :**
```bash
python Codes/yolo26distances.py \
  --video DatasetsVideo/DJI_0025_cut1.Mp4 \
  --model yolo26s.pt \
  --out_dir Outputs
```
**Sorties :**
- `*_robust_traj_gaps.mp4`
- `*_car_kinematics.csv`
- `*_gaps.csv`

---

### 5) `Codes/extartcdata.py`
**But :** extraction highD vers CSV propres pour MATLAB/SUMO.

**Commande (exemple) :**
```bash
python Codes/extartcdata.py \
  --data_dir Datasets/highD-dataset-v1.0/data \
  --out_dir Datasets/exports \
  --recordings 01 02 03
```
**Sorties :** `trajectories_long.csv`, `car_following_pairs.csv`,
`lane_change_events.csv`, `summary.json` (par recording).

---

### 6) `VisualisationGIT/highD-dataset-master/Python/src/main.py`
**But :** outil officiel highD pour lire les CSV et visualiser les tracks.

**Commande (exemple) :**
```bash
cd VisualisationGIT/highD-dataset-master/Python/src
python main.py \
  --input_path ../data/01_tracks.csv \
  --input_static_path ../data/01_tracksMeta.csv \
  --input_meta_path ../data/01_recordingMeta.csv \
  --background_image ../data/01_highway.png
```
**Note :** les chemins par defaut pointent vers `../data/`. Adaptez selon votre structure.

---

### 7) `VisualisationGIT/highD-dataset-master/Python/src/data_management/read_csv.py`
**But :** module d'import CSV highD. Pas de CLI.

**Usage (import) :**
```python
from data_management.read_csv import read_track_csv, read_static_info, read_meta_info
```

---

### 8) `VisualisationGIT/highD-dataset-master/Python/src/visualization/visualize_frame.py`
**But :** classe de visualisation interactive. Utilisee par `main.py`.

**Usage :** via `main.py` (pas d'execution directe).

---

### 9) `VisualisationGIT/drone-dataset-tools-master/src/run_track_visualization.py`
**But :** visualisation des datasets exiD / inD / rounD / uniD (outil officiel).

**Commande (exemple) :**
```bash
cd VisualisationGIT/drone-dataset-tools-master/src
python run_track_visualization.py --dataset exid --recording 26 \
  --dataset_dir /chemin/vers/exiD-dataset-v2.1/data
```

---

### 10) `VisualisationGIT/drone-dataset-tools-master/src/tracks_import.py`
**But :** module d'import des CSV (utilise par `run_track_visualization.py`).

**Usage (import) :**
```python
from tracks_import import read_from_csv, read_all_recordings_from_csv
```

---

### 11) `VisualisationGIT/drone-dataset-tools-master/src/track_visualizer.py`
**But :** moteur de visualisation (fenetre interactive). Utilise par `run_track_visualization.py`.

**Usage :** via `run_track_visualization.py` (pas d'execution directe).

---

## Inventaire detaille du depot (fichier par fichier)

### Racine
- `ARCHITECTURE.md` : description technique du pipeline et des formats de datasets.
- `yolo26s.pt` : poids YOLO (utilisable avec les scripts YOLO).

### `Codes/`
- `Codes/visualisation.py` : visualisation multi-datasets (highD, inD, rounD, uniD, exiD).
- `Codes/visV2.py` : visualisation highD avancee (trajectoires + distances colorees).
- `Codes/yolo26_car_kinematics.py` : YOLO + tracking -> video annotee + CSV cinematique.
- `Codes/yolo26distances.py` : detection robuste par tuiles + gaps/THW + CSV.
- `Codes/extartcdata.py` : extraction highD vers exports CSV MATLAB/SUMO.
- `Codes/readme.txt` : note rapide sur le chemin `BASE_DIR` et le `RECORDING_ID`.
- `Codes/Tutoriel dataset Matlab.pdf` : tuto PDF (dataset MATLAB).

### `DatasetsVideo/`
- `DatasetsVideo/liens.txt` : **tous les liens vers datasets video et ressources**.

### `Outputs/`
- `Outputs/DJI_0025_cut1_car_traj.mp4` : exemple de sortie (trajectoires YOLO).
- `Outputs/20220602_100001_Koh_Dor_4W_d_1_1_org_sample_car_traj.mp4` : autre exemple de sortie.
- `Outputs/.DS_Store` : fichier systeme macOS.

#### `VisualisationGIT/highD-dataset-master/`
- `VisualisationGIT/highD-dataset-master/README.md` : README officiel highD tools.
- `VisualisationGIT/highD-dataset-master/LICENSE` : licence.
- `VisualisationGIT/highD-dataset-master/.gitignore` : ignore git.
- `VisualisationGIT/highD-dataset-master/Python/README.md` : README Python tools.
- `VisualisationGIT/highD-dataset-master/Python/requirements.txt` : dependances Python highD tools.
- `VisualisationGIT/highD-dataset-master/Python/src/main.py` : entree principale (visualisation).
- `VisualisationGIT/highD-dataset-master/Python/src/data_management/read_csv.py` : import CSV.
- `VisualisationGIT/highD-dataset-master/Python/src/visualization/visualize_frame.py` : visualisation interactive.
- `VisualisationGIT/highD-dataset-master/Matlab/README.md` : README MATLAB tools.
- `VisualisationGIT/highD-dataset-master/Matlab/initialize.m` : initialisation MATLAB.
- `VisualisationGIT/highD-dataset-master/Matlab/bin/startVisualization.m` : script de lancement MATLAB.
- `VisualisationGIT/highD-dataset-master/Matlab/visualization/trackVisualization.fig` : UI MATLAB (figure).
- `VisualisationGIT/highD-dataset-master/Matlab/visualization/trackVisualization.m` : UI MATLAB (code).
- `VisualisationGIT/highD-dataset-master/Matlab/utils/plotHighway.m` : utilitaire MATLAB (fond).
- `VisualisationGIT/highD-dataset-master/Matlab/utils/plotTracks.m` : utilitaire MATLAB (tracks).
- `VisualisationGIT/highD-dataset-master/Matlab/utils/plotTracksOnImage.m` : utilitaire MATLAB (overlay image).
- `VisualisationGIT/highD-dataset-master/Matlab/utils/readInTracksCsv.m` : import CSV tracks.
- `VisualisationGIT/highD-dataset-master/Matlab/utils/readInVideoCsv.m` : import CSV meta.

#### `VisualisationGIT/drone-dataset-tools-master/`
- `VisualisationGIT/drone-dataset-tools-master/README.md` : README officiel drone-dataset tools.
- `VisualisationGIT/drone-dataset-tools-master/LICENSE` : licence.
- `VisualisationGIT/drone-dataset-tools-master/.gitignore` : ignore git.
- `VisualisationGIT/drone-dataset-tools-master/requirements.txt` : dependances Python.
- `VisualisationGIT/drone-dataset-tools-master/data/.gitkeep` : placeholder du dossier data.
- `VisualisationGIT/drone-dataset-tools-master/doc/screenshot_track_visualization.png` : capture d'ecran.
- `VisualisationGIT/drone-dataset-tools-master/src/tracks_import.py` : import des CSV.
- `VisualisationGIT/drone-dataset-tools-master/src/run_track_visualization.py` : CLI de visualisation.
- `VisualisationGIT/drone-dataset-tools-master/src/track_visualizer.py` : visualiseur interactif.

### `ZIPs/`
- `ZIPs/exiD-dataset-v2.1.zip` : archive dataset exiD.
- `ZIPs/inD-dataset-v1.1.zip` : archive dataset inD.
- `ZIPs/round-dataset-v1.1.zip` : archive dataset rounD.
- `ZIPs/uniD-dataset-v1.1.zip` : archive dataset uniD.
- `ZIPs/highD-dataset-v1.0.zip` : archive dataset highD.
- `ZIPs/highD-dataset-master.zip` : archive outils highD (repo).
- `ZIPs/drone-dataset-tools-master.zip` : archive outils drone-dataset.
- `ZIPs/drive-download-20260126T131006Z-1-001.zip` : archive diverse (telechargement groupel).

---

## Notes pratiques
- Les videos et datasets peuvent etre volumineux : prevoir de l'espace disque.
- Les scripts YOLO sont sensibles aux performances : GPU recommande mais non obligatoire.
- Les fichiers `.URL` / `.url` sont des raccourcis macOS, pas les datasets eux-memes.
