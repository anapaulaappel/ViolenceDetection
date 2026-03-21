# ViolenceDetection

Deteção de violência em vídeo com YOLOv8, dados do cliente e dataset público (ex.: RWF-2000).

## Ordem de execução (ficheiros `01` … `10`)

Os notebooks estão numerados **na ordem em que devem ser executados** na pasta `notebooks/`.

| Passo | Notebook | O que faz |
|------|----------|-----------|
| 1 | `01_client_eda.ipynb` | Catálogo, movimento, thumbnails → `data/client_analysis/` |
| 2 | `02_dataset_public.ipynb` | Dataset YOLO a partir de `data/public_raw/` → `data/datasets/public_only/` |
| 3 | `03_train_public_only.ipynb` | Treino só no público → `models/public_only_best.pt` |
| 4 | `04_client_classification.ipynb` | Segmentos + pré-rotulagem + revisão → `data/client_labeled/` |
| 5 | `05_dataset_client_only.ipynb` | Imagens/labels YOLO do cliente → `data/datasets/client_only/` |
| 6a | `06_train_client_only.ipynb` | Experimento A (só cliente); fine-tune a partir de `public_only_best.pt` se existir |
| 6b | *ou* `07_train_combined.ipynb` | Experimento B (cliente + público); mesma inicialização opcional |
| 7 | `08_comparison.ipynb` | Comparar A vs B |
| 8 | `09_lstm_pre_violence.ipynb` | Classificador temporal (opcional) |
| 9 | `10_inference_demo.ipynb` | Demo em vídeo / RTSP |

## Atalho (sem dataset público)

`01` → `04` → `05` → `06` ou `07` — a pré-rotulagem no `04` fica só com COCO + pose até existir `public_only_best.pt`.

## Pesos em `models/`

| Ficheiro | Origem |
|----------|--------|
| `public_only_best.pt` | `03_train_public_only` |
| `client_only_best.pt` | `06_train_client_only` |
| `combined_best.pt` | `07_train_combined` |

No `04_client_classification`, o YOLO usa o melhor disponível: `combined_best` → `client_only_best` → `public_only_best` → `best.pt`.

## Dados

- Vídeos do cliente: `data/client_videos/`
- RWF-2000 (ou similar): `data/public_raw/violence/` e `non_violence/`
