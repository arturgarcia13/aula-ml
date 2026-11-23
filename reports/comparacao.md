# Relatório de Comparação: `chamados-r1` vs `chamados-r2`

## 1. Alteração realizada

Na ramificação `correcao-rotulo` (tag `dados-v2`), foi corrigido um erro deliberado de rotulagem no conjunto de treino em `data/chamados.csv`:

- **Chamado `006`**: `"bloqueio de acesso"` alterado de `comum` para `urgente`, conforme a regra de rotulagem (`bloqueio` é `urgente`).
- **`data/manifesto.json`**: atualizado `dataset_id` de `chamados-v1` para `chamados-v2` e registrada a alteração no campo `alteracao`.
- **`data/split.csv`**: mantido idêntico entre `v1` e `v2` (`git diff dados-v1 dados-v2 -- data/split.csv.dvc` retornou vazio), garantindo comparação na mesma partição fixa de teste (4 exemplos: 2 `comum` e 2 `urgente`).

Em seguida, o modelo `chamados-r2` foi treinado com `python3 src/train.py chamados-r2` e registrado na tag `modelo-r2`.

## 2. Comparação de métricas

| Versão | Dataset | `split_sha256` | Acurácia | Recall `comum` (suporte=2) | Recall `urgente` (suporte=2) |
|---|---|---|---|---|---|
| `chamados-r1` (`modelo-r1`) | `chamados-v1` | `2ef820171c1a4625fe216e6da67c7c8dcf9d97a08288c0239cd23f1a6961d981` | 0.75 (3/4) | 1.00 (2/2) | 0.50 (1/2) |
| `chamados-r2` (`modelo-r2`) | `chamados-v2` | `2ef820171c1a4625fe216e6da67c7c8dcf9d97a08288c0239cd23f1a6961d981` | 1.00 (4/4) | 1.00 (2/2) | 1.00 (2/2) |

Com a correção do rótulo do chamado `006`, o token `bloqueio` passou a ser associado à classe `urgente` em `models/model.json`, fazendo com que o chamado de teste `010` (`bloqueio de conta,urgente`) passasse a ser classificado corretamente.

### Comparação dos hashes registrados em `reports/modelo.json`

| Artefato | `chamados-r1` (`modelo-r1`) | `chamados-r2` (`modelo-r2`) | Alterado? |
|---|---|---|---|
| `data/chamados.csv` (`dataset_sha256`) | `bc3fd86a3a5f598841a58839a9726d901fa7e8361ffe79c5b43873ad8a1328ac` | `4ca2e29cb930a9161c2a851b3c71f5529dbb812db607ee4a5ea95d2223baa14d` | Sim |
| `data/split.csv` (`split_sha256`) | `2ef820171c1a4625fe216e6da67c7c8dcf9d97a08288c0239cd23f1a6961d981` | `2ef820171c1a4625fe216e6da67c7c8dcf9d97a08288c0239cd23f1a6961d981` | Não |
| `models/model.json` (`artifact_sha256`) | `09d965544c9298c5586af0a1becbf07fb3f3c7b7c228a803ab91179f8b458a09` | `3bcddff15fe272fe16e092dfb76dab30fa3786cd8c2ce4849ccf5ace002c6ee6` | Sim |

## 3. Evidência de restauração em clone limpo (`modelo-r1`)

O repositório foi clonado novamente em `../aula-ml-restauracao`, com checkout destacado na tag `modelo-r1` e restauração dos artefatos via `dvc pull`:

- **Commit restaurado (`git rev-parse HEAD`)**: `4892c3762129da7d7a37197d6603e3add78dc955`
- **Verificação SHA-256 contra `reports/modelo.json`**:
  - `OK data/chamados.csv bc3fd86a3a5f598841a58839a9726d901fa7e8361ffe79c5b43873ad8a1328ac`
  - `OK data/split.csv 2ef820171c1a4625fe216e6da67c7c8dcf9d97a08288c0239cd23f1a6961d981`
  - `OK models/model.json 09d965544c9298c5586af0a1becbf07fb3f3c7b7c228a803ab91179f8b458a09`

## 4. Decisão

**Decisão: Aceitar `chamados-r2` (`modelo-r2`).**

**Critério avaliado:** melhorar o recall da classe `urgente` sem reduzir o recall da classe `comum`, utilizando exatamente a mesma partição de teste (`split_sha256` idêntico).

- Recall de `urgente` subiu de `0.50` para `1.00` (+0.50).
- Recall de `comum` manteve-se em `1.00` (sem regressão).
- Acurácia global subiu de `0.75` para `1.00`, embora o conjunto de teste tenha apenas 4 exemplos e sirva apenas para demonstrar o efeito da correção neste exercício.
