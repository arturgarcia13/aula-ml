# aula-ml — Versionamento de Dados e Modelos

Repositório da Atividade Prática 07 (CK0444 — Engenharia de Sistemas Inteligentes, UFC).
Versiona datasets, partições de treino/teste e modelos de classificação de chamados usando Git + DVC.

## Pré-requisitos

- **Git** (`git --version`)
- **Python 3.10+** (`python3 --version`, testado com Python 3.14.4; o script `src/train.py` usa apenas a biblioteca padrão)
- **DVC 3.x** (`dvc --version`, testado com DVC 3.67.1)

## Configuração de acesso ao armazenamento DVC

O repositório já possui o remoto padrão `storage` configurado em `.dvc/config` apontando para a pasta irmã `../dvc-storage` (armazenada em `.dvc/config` como `../../dvc-storage` em relação ao diretório `.dvc/`).

Para que o `dvc pull` funcione em um clone limpo:

1. Garanta que o diretório `dvc-storage` esteja ao lado do diretório clonado (mesma pasta pai) com os objetos enviados via `dvc push`.
2. Caso o armazenamento esteja em outro caminho, atualize o remoto localmente sem alterar o histórico versionado (ou ajuste conforme necessário):

```bash
dvc remote list
# Caso precise apontar para outro diretório:
# dvc remote modify --local storage url /caminho/para/dvc-storage
```

## Restauração de versões anteriores

Cada rodada de dados e treinamento está marcada com tags Git (`dados-v1`, `modelo-r1`, `dados-v2`, `modelo-r2`). Para recuperar uma versão específica (por exemplo, `modelo-r1`) em um clone limpo sem retreinar o modelo:

```bash
git clone https://github.com/arturgarcia13/aula-ml.git aula-ml-restauracao
cd aula-ml-restauracao
git switch --detach modelo-r1
dvc pull
```

Para conferir a integridade dos arquivos restaurados contra o manifesto registrado em `reports/modelo.json`:

```bash
python3 - <<'PY'
import hashlib
import json
from pathlib import Path

manifesto = json.loads(Path("reports/modelo.json").read_text(encoding="utf-8"))
for caminho, campo in [
    ("data/chamados.csv", "dataset_sha256"),
    ("data/split.csv", "split_sha256"),
    ("models/model.json", "artifact_sha256"),
]:
    obtido = hashlib.sha256(Path(caminho).read_bytes()).hexdigest()
    if obtido != manifesto[campo]:
        raise SystemExit(f"Hash divergente para {caminho}")
    print(f"OK {caminho} {obtido}")
PY
```

## Reprodução do treinamento

Com os dados restaurados pelo DVC (`dvc pull`), o treinamento e a avaliação podem ser executados novamente com:

```bash
python3 src/train.py <model_id>
```
