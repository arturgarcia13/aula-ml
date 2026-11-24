# aula-ml

Repositório da Atividade Prática 07 de Engenharia de Sistemas Inteligentes (CK0444, UFC). Aqui versiono o dataset de chamados, a divisão de treino e teste e os modelos gerados usando Git junto com DVC.

## Pré-requisitos

Você precisa de Git, Python 3.10 ou mais novo (rodei tudo no 3.14.4; `src/train.py` só usa a biblioteca padrão) e DVC 3 (`dvc 3.67.1` no meu ambiente).

## Armazenamento DVC

O arquivo `.dvc/config` já aponta o remoto padrão `storage` para `../../dvc-storage`, que resolve para a pasta irmã `../dvc-storage` a partir da raiz do projeto. Se o diretório `dvc-storage` estiver ao lado do clone com os objetos enviados via `dvc push`, o `dvc pull` puxa os dados e o modelo sem configuração extra.

Caso o storage fique em outro caminho na sua máquina, atualize o remoto apenas localmente para não alterar os arquivos versionados no Git:

```bash
dvc remote list
dvc remote modify --local storage url /caminho/para/dvc-storage
```

## Como restaurar uma versão

As rodadas de dados e treino ficam marcadas pelas tags `dados-v1`, `modelo-r1`, `dados-v2` e `modelo-r2`. Para recuperar `modelo-r1` em um diretório limpo:

```bash
git clone https://github.com/arturgarcia13/aula-ml.git aula-ml-restauracao
cd aula-ml-restauracao
git switch --detach modelo-r1
dvc pull
```

Não precisa treinar nada de novo: o `dvc pull` traz o `models/model.json` salvo naquela rodada. Para conferir se os arquivos recuperados batem com os hashes registrados em `reports/modelo.json`:

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

Se quiser rodar o treinamento de novo depois do `dvc pull`, execute `python3 src/train.py <model_id>`.
