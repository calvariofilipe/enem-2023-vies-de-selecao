# ENEM 2023 — Viés de Seleção Socioeconômico

Este repositório reúne os dados e notebooks utilizados no MVP de Análise Exploratória e Pré-processamento de Dados da pós-graduação em Ciência de Dados e Analytics da PUC-Rio.

O projeto investiga diferenças socioeconômicas entre os inscritos no ENEM 2023 e os participantes com notas disponíveis, examinando como os critérios de seleção podem alterar a composição da população analisada.

## Origem e preparação dos dados

Os dados foram obtidos dos [Microdados do ENEM 2023](https://www.gov.br/inep/pt-br/acesso-a-informacao/dados-abertos/microdados/enem), disponibilizados publicamente pelo Instituto Nacional de Estudos e Pesquisas Educacionais Anísio Teixeira (INEP).

Como o arquivo original possui aproximadamente **1,74 GB**, foi realizada uma etapa prévia de preparação, na qual foram selecionadas 16 variáveis relevantes para a investigação, preservando todos os **3.933.955 registros**.

O resultado foi dividido em 16 arquivos CSV menores, disponibilizados na pasta [`dados/`](dados/).

## Organização do repositório

- **`dados/`** — contém os 16 arquivos CSV preparados, de `enem_2023_parte_001.csv` a `enem_2023_parte_016.csv`.
- **`MVP_3.ipynb`** — notebook principal, contendo a análise exploratória, o pré-processamento e a discussão dos resultados.
- **`Preparacao_Microdados_ENEM_2023.ipynb`** — notebook auxiliar que documenta o download dos microdados originais, a seleção das variáveis e a divisão dos registros em arquivos menores.

## Como executar

Para reproduzir a análise, basta abrir o notebook principal no Google Colab e executar suas células na sequência. Os dados são carregados diretamente dos arquivos CSV disponíveis neste repositório, sem necessidade de baixar a base original do INEP.

**Não é necessário executar o notebook auxiliar de preparação.** Ele foi incluído exclusivamente para documentar o procedimento utilizado e permitir sua reprodução, caso desejado. Sua execução exige o download e o processamento dos microdados originais, tornando essa etapa consideravelmente mais demorada e dispensável para a análise principal.

## Fonte dos dados

Instituto Nacional de Estudos e Pesquisas Educacionais Anísio Teixeira (INEP). *Microdados do ENEM 2023*. Disponível em: https://www.gov.br/inep/pt-br/acesso-a-informacao/dados-abertos/microdados/enem.
