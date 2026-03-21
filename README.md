# ViolenceDetection

Deteção de violência em vídeo com YOLOv8, dados do cliente e dataset público (ex.: RWF-2000).

## Ordem recomendada de execução

Esta sequência **pré-treina no público antes** de pré-rotular os vídeos do cliente, o que reduz falsos positivos na fila de revisão manual.

| Passo | Notebook | O que faz |
|------|----------|-----------|
| 1 | `notebooks/01_client_eda.ipynb` | Catálogo, movimento, thumbnails (`data/client_analysis/`) |
| 2 | `notebooks/04_dataset_public.ipynb` | Dataset YOLO a partir de `data/public_raw/` → `data/datasets/public_only/` |
| 3 | `notebooks/05a_train_public_only.ipynb` | Treino só no público → `models/public_only_best.pt` |
| 4 | `notebooks/02_client_classification.ipynb` | Segmentos com movimento + YOLO/pose + revisão → `data/client_labeled/` |
| 5 | `notebooks/03_dataset_client_only.ipynb` | Imagens/labels YOLO do cliente → `data/datasets/client_only/` |
| 6a | `notebooks/05b_train_client_only.ipynb` | Experimento A (só cliente); usa `public_only_best.pt` como inicialização se existir |
| 6b | *ou* `notebooks/06_train_combined.ipynb` | Experimento B (cliente + público); mesma inicialização opcional |
| 7 | `notebooks/07_comparison.ipynb` | Comparar A vs B |
| 8 | `notebooks/08_lstm_pre_violence.ipynb` | Classificador temporal (opcional) |
| 9 | `notebooks/09_inference_demo.ipynb` | Demo em vídeo / RTSP |

Os ficheiros **05a** e **05b** estão nomeados assim para aparecerem nesta ordem na pasta `notebooks/`.

## Ordem mínima (sem público)

`01` → `02` → `03` → `05b` ou `06` — possível, mas a pré-classificação no `02` fica limitada ao YOLO COCO + pose até existir um modelo próprio.

## Pesos esperados em `models/`

| Ficheiro | Origem |
|----------|--------|
| `public_only_best.pt` | `05a_train_public_only` |
| `client_only_best.pt` | `05b_train_client_only` |
| `combined_best.pt` | `06_train_combined` |

O notebook `02` escolhe o melhor disponível nesta ordem: `combined_best` → `client_only_best` → `public_only_best` → `best.pt`.

## Dados

- Vídeos do cliente: `data/client_videos/`
- RWF-2000 (ou similar): `data/public_raw/violence/` e `non_violence/`
