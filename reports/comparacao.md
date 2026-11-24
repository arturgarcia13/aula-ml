# Comparação entre `chamados-r1` e `chamados-r2`

## O que mudou

Na branch `correcao-rotulo`, corrigi o erro intencional de rotulagem no chamado `006` (`bloqueio de acesso`) em `data/chamados.csv`: ele estava como `comum`, mas pela regra do dataset qualquer bloqueio pertence à classe `urgente`. Atualizei também `data/manifesto.json` para `chamados-v2` descrevendo a correção e marquei o snapshot na tag `dados-v2`.

O arquivo `data/split.csv` continuou igual (`git diff dados-v1 dados-v2 -- data/split.csv.dvc` retornou vazio), então a comparação entre os dois modelos usa os mesmos 4 exemplos de teste. Em seguida rodei `python3 src/train.py chamados-r2` e registrei o novo artefato na tag `modelo-r2`.

## Métricas e hashes

| Versão | Dataset | `split_sha256` | Acurácia | Recall `comum` (suporte=2) | Recall `urgente` (suporte=2) |
|---|---|---|---|---|---|
| `chamados-r1` (`modelo-r1`) | `chamados-v1` | `2ef820171c1a4625fe216e6da67c7c8dcf9d97a08288c0239cd23f1a6961d981` | 0.75 (3/4) | 1.00 (2/2) | 0.50 (1/2) |
| `chamados-r2` (`modelo-r2`) | `chamados-v2` | `2ef820171c1a4625fe216e6da67c7c8dcf9d97a08288c0239cd23f1a6961d981` | 1.00 (4/4) | 1.00 (2/2) | 1.00 (2/2) |

Mudando só o rótulo do chamado `006`, o classificador passou a associar o token `bloqueio` a `urgente` em `models/model.json`; no teste, isso corrigiu a previsão do chamado `010` (`bloqueio de conta`), que `r1` errava.

| Artefato | `chamados-r1` (`modelo-r1`) | `chamados-r2` (`modelo-r2`) | Mudou? |
|---|---|---|---|
| `data/chamados.csv` (`dataset_sha256`) | `bc3fd86a3a5f598841a58839a9726d901fa7e8361ffe79c5b43873ad8a1328ac` | `4ca2e29cb930a9161c2a851b3c71f5529dbb812db607ee4a5ea95d2223baa14d` | Sim |
| `data/split.csv` (`split_sha256`) | `2ef820171c1a4625fe216e6da67c7c8dcf9d97a08288c0239cd23f1a6961d981` | `2ef820171c1a4625fe216e6da67c7c8dcf9d97a08288c0239cd23f1a6961d981` | Não |
| `models/model.json` (`artifact_sha256`) | `09d965544c9298c5586af0a1becbf07fb3f3c7b7c228a803ab91179f8b458a09` | `3bcddff15fe272fe16e092dfb76dab30fa3786cd8c2ce4849ccf5ace002c6ee6` | Sim |

## Restauração de `modelo-r1` em clone limpo

Clonei o repositório do GitHub em `../aula-ml-restauracao`, fiz checkout destacado na tag `modelo-r1` (`git switch --detach modelo-r1`) e executei `dvc pull`. O commit restaurado bateu em `4892c3762129da7d7a37197d6603e3add78dc955`, e a conferência de SHA-256 contra `reports/modelo.json` passou nos três arquivos:

```text
OK data/chamados.csv bc3fd86a3a5f598841a58839a9726d901fa7e8361ffe79c5b43873ad8a1328ac
OK data/split.csv 2ef820171c1a4625fe216e6da67c7c8dcf9d97a08288c0239cd23f1a6961d981
OK models/model.json 09d965544c9298c5586af0a1becbf07fb3f3c7b7c228a803ab91179f8b458a09
```

## Decisão

Aceito o `chamados-r2` (`modelo-r2`). O critério definido para a revisão era aumentar o recall de `urgente` sem derrubar o recall de `comum` na mesma partição de teste; o recall de `urgente` subiu de `0.50` para `1.00`, o de `comum` se manteve em `1.00` e a acurácia foi de `0.75` para `1.00`. O teste tem só quatro chamados e serve apenas para validar a correção neste roteiro, mas dentro do critério adotado o novo modelo cumpre o objetivo.
