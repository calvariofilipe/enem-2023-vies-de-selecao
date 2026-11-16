# ENEM 2023 — Viés de Seleção Socioeconômico

Este repositório reúne dados provenientes dos **Microdados do ENEM 2023**, disponibilizados publicamente pelo Instituto Nacional de Estudos e Pesquisas Educacionais Anísio Teixeira (INEP), preparados para o MVP de Análise Exploratória e Pré-processamento de Dados da pós-graduação em Ciência de Dados e Analytics da PUC-Rio.

O projeto investiga diferenças socioeconômicas entre o conjunto de inscritos e os participantes com notas disponíveis, examinando como diferentes critérios de seleção podem alterar a composição da população analisada.

Devido ao tamanho do arquivo original, de aproximadamente **1,74 GB**, optou-se por realizar a extração e a redução inicial dos dados separadamente, evitando que a execução do notebook principal dependesse do download e processamento integral da base. Foram selecionadas 16 variáveis relevantes para a investigação, preservando os 3.933.955 registros originais.

Por essa razão, o procedimento de preparação é **documentado neste README, com explicações e trechos de código**, em vez de ser reproduzido em um notebook auxiliar que exigiria o processamento de um arquivo de grande porte. Os dados resultantes estão disponíveis diretamente no repositório, permitindo que a análise principal seja executada de maneira mais simples e eficiente no Google Colab.

## Preparação dos dados

### 1. Obtenção da base original

Os dados foram obtidos no https://www.gov.br/inep/pt-br/acesso-a-informacao/dados-abertos/microdados/enem, selecionando a edição de 2023.

Após o download e a extração do arquivo ZIP, utilizamos o arquivo:

`microdados_enem_2023/DADOS/MICRODADOS_ENEM_2023.csv`

O CSV original possui aproximadamente 1,74 GB e contém informações sobre 3.933.955 inscrições.

### 2. Seleção das variáveis

Para reduzir o tamanho dos dados sem comprometer os objetivos da investigação, selecionamos 16 variáveis relacionadas à identificação das inscrições, características socioeconômicas, presença nas provas e notas.

| Grupo | Variáveis selecionadas |
|---|---|
| Identificação | `NU_INSCRICAO` |
| Características dos inscritos | `TP_FAIXA_ETARIA`, `TP_ESCOLA`, `SG_UF_PROVA` |
| Questionário socioeconômico | `Q001`, `Q002`, `Q006` |
| Presença nas provas | `TP_PRESENCA_CN`, `TP_PRESENCA_CH`, `TP_PRESENCA_LC`, `TP_PRESENCA_MT` |
| Notas | `NU_NOTA_CN`, `NU_NOTA_CH`, `NU_NOTA_LC`, `NU_NOTA_MT`, `NU_NOTA_REDACAO` |

A seleção foi implementada a partir dos nomes das colunas do arquivo original:

```python
COLUNAS = ["NU_INSCRICAO", "TP_FAIXA_ETARIA", "TP_ESCOLA", "SG_UF_PROVA", "Q001", "Q002", "Q006", "TP_PRESENCA_CN",
"TP_PRESENCA_CH", "TP_PRESENCA_LC", "TP_PRESENCA_MT", "NU_NOTA_CN", "NU_NOTA_CH", "NU_NOTA_LC","NU_NOTA_MT", "NU_NOTA_REDACAO"]
```

Não selecionamos apenas participantes presentes, tampouco exigimos notas válidas. **Todas as inscrições foram preservadas**, inclusive as de participantes ausentes ou eliminados.

### 3. Leitura e extração das colunas

O processamento foi realizado em Python, utilizando a biblioteca nativa `csv`.

O arquivo foi aberto com codificação `latin-1` e separador `;`, conforme a estrutura dos microdados originais.

A extração foi realizada linha por linha, evitando carregar todo o CSV na memória.

O trecho abaixo reproduz a lógica central do processamento:

```python
import csv

with open("MICRODADOS_ENEM_2023.csv",encoding="latin-1", newline="") as entrada:

    leitor = csv.reader(entrada, delimiter=";")
    cabecalho = next(leitor)
    posicoes = [cabecalho.index(coluna) for coluna in COLUNAS]

    for linha in leitor:
        selecionados = [linha[i] for i in posicoes]
```

No script utilizado, verificamos também se todas as colunas solicitadas estavam presentes no cabeçalho e se cada linha possuía a quantidade esperada de campos.

A operação seleciona colunas, mas não filtra linhas nem modifica os valores originais.

### 4. Divisão em arquivos menores

Para permitir a publicação no GitHub, dividimos os registros em arquivos com no máximo 250 mil linhas de dados cada.

A gravação original foi realizada diretamente em arquivos CSV compactados com GZIP:

```python
import gzip

saida = gzip.open(
    "enem_2023_parte_001.csv.gz",
    "wt",
    encoding="utf-8",
    newline="",
    compresslevel=6
)

escritor = csv.writer(saida, delimiter=";")
escritor.writerow(COLUNAS)

# Para cada registro processado:
escritor.writerow(selecionados)
```

O script criava automaticamente um novo arquivo ao alcançar 250 mil registros, repetindo o cabeçalho em cada parte.

Ao final, foram produzidos **16 arquivos `.csv.gz`**, totalizando aproximadamente 62,8 MB.

Posteriormente, os arquivos foram descompactados individualmente, preservando a divisão em 16 partes e o conteúdo das tabelas, para disponibilização no repositório como arquivos `.csv`.

O conjunto descompactado ocupa aproximadamente 209 MB.

### 5. Verificação do processamento

Durante a extração, foram contabilizados os registros e as frequências das categorias de renda familiar (`Q006`), presença em Matemática (`TP_PRESENCA_MT`) e disponibilidade da nota (`NU_NOTA_MT`).

O diagnóstico da extração apresentou:

| Indicador | Resultado |
|---|---:|
| Registros processados | 3.933.955 |
| Variáveis selecionadas | 16 |
| Arquivos gerados | 16 |
| Inscritos presentes em Matemática | 2.692.427 |
| Inscritos ausentes em Matemática | 1.239.316 |
| Inscritos eliminados em Matemática | 2.212 |

A soma das categorias de presença corresponde ao total de inscrições, confirmando a preservação dos registros nessa verificação.

Adicionalmente, no notebook principal, verificamos que a base reconstruída contém 3.933.955 registros e que não existem inscrições duplicadas.

### 6. Utilização no notebook principal

Os 16 arquivos CSV estão disponíveis na pasta [`dados/`](dados/) deste repositório.

O notebook principal utiliza `pandas` para carregar cada arquivo diretamente do GitHub e reunir os registros em um único DataFrame:

```python
import pandas as pd

url_base = (
    "https://raw.githubusercontent.com/"
    "calvariofilipe/enem-2023-vies-de-selecao/main/dados"
)

arquivos = [
    pd.read_csv(
        f"{url_base}/enem_2023_parte_{i:03d}.csv",
        sep=";",
        low_memory=False
    )
    for i in range(1, 17)
]

df = pd.concat(arquivos, ignore_index=True)
```

Dessa forma, as análises podem ser reproduzidas no Google Colab sem que o avaliador precise baixar o arquivo original de aproximadamente 1,74 GB.

Os critérios de seleção de participantes, as transformações e as decisões de pré-processamento são documentados no notebook principal, separadamente da extração inicial.
## Dados disponibilizados

Os arquivos preparados estão disponíveis na pasta [`dados/`](dados/), com nomes de `enem_2023_parte_001.csv` a `enem_2023_parte_016.csv`.

O conjunto contém:

| Característica | Quantidade |
|---|---:|
| Registros | 3.933.955 |
| Variáveis | 16 |
| Arquivos CSV | 16 |
| Tamanho total aproximado | 209 MB |

Os arquivos podem ser carregados diretamente do GitHub pelo Google Colab, sem a necessidade de baixar novamente os microdados originais.

## Reprodutibilidade

A preparação dos dados utiliza Python e pandas, com leitura em blocos de 250 mil registros.

O procedimento poderá ser reproduzido a partir do arquivo original disponibilizado pelo INEP, selecionando as mesmas 16 variáveis e preservando a ordem dos registros. O notebook principal utilizará diretamente os arquivos CSV disponibilizados neste repositório.

## Finalidade da análise

O estudo investiga três questões:

- A disponibilidade da nota de Matemática varia conforme a renda familiar?
- A seleção dos participantes com nota altera a composição socioeconômica da população analisada?
- Exigir notas nas cinco avaliações produz diferenças adicionais relevantes?

O objetivo é explorar os dados e avaliar as consequências das decisões de pré-processamento, sem treinamento de modelos preditivos.

## Licença e atribuição

Os dados originais pertencem à base pública disponibilizada pelo INEP. A fonte deve ser citada em qualquer utilização ou reprodução dos dados preparados.
