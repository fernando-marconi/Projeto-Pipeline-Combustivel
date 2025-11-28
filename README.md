# Projeto | Pipeline e Modelagem de Dados de Preços de Combustíveis

## Visão Geral do Projeto

Este projeto demonstra a construção de um pipeline de **Engenharia de Dados** completo para processar, limpar e modelar dados históricos de preços de combustíveis no Brasil, oriundos da ANP (Agência Nacional do Petróleo, Gás Natural e Biocombustíveis).

A solução segue o padrão de **Arquitetura Medallion (Bronze, Prata e Ouro)**, implementada em um ambiente totalmente gratuito e reprodutível usando **Google Colab** e bibliotecas Python, simulando a estrutura de um Data Lake no Google Drive.

## Stack Tecnológico

| Categoria | Tecnologia | Uso no Projeto |
| :--- | :--- | :--- |
| **Ambiente** | `Google Colab` | Ambiente de desenvolvimento (Notebook) totalmente gratuito e baseado em nuvem. |
| **Data Lake** | `Google Drive` | Utilizado para simular o armazenamento em um Data Lake (S3, GCS ou Azure Blob Storage), separando as camadas Bronze, Prata e Ouro. |
| **Processamento** | `Python`, `Pandas`, `NumPy` | Linguagem e bibliotecas centrais para todo o ETL e manipulação de dados em memória. |
| **Data Warehouse** | `DuckDB` | Utilizado como um Data Warehouse/banco de dados analítico *in-process* (local) para consolidar e estruturar a Camada Prata via SQL. |
| **Saída Analítica** | `Parquet` | Formato colunar para armazenamento final dos dados modelados (Camada Ouro), otimizado para consumo em BI e Machine Learning. |

---

## Detalhamento da Arquitetura Medallion

O pipeline executa transformações incrementais em cada camada, elevando a qualidade dos dados:

### 1. Camada Bronze (Raw)
* **Origem dos Dados:** Múltiplos arquivos CSV (dados brutos) são lidos do diretório `Bronze/` no Google Drive.
* **Ação:** Ingestão e limpeza inicial dos nomes das colunas, removendo caracteres especiais e ajustando para o padrão `snake_case`.

### 2. Camada Prata (Silver)
* **Objetivo:** Garantir a qualidade e a estrutura dos dados.
* **Transformação:**
    * **Tipagem:** Colunas de preço (`valor_de_venda` e `valor_de_compra`) são convertidas para `float32`, tratando vírgulas como decimais. A coluna de data (`data_da_coleta`) é convertida para o tipo `datetime`.
    * **Consolidação (DuckDB):** Os dados transformados de todos os arquivos CSV são anexados a uma única tabela (`tabela_combustiveis`) em um banco de dados DuckDB, utilizando comandos SQL.
    * **Log de Qualidade:** Um log de processamento (`log_dados_combustivel.csv`) é criado e salvo na pasta `Prata/`, registrando tipos de dados e contagens de valores nulos por arquivo processado.

### 3. Camada Ouro (Gold)
* **Objetivo:** Fornecer dados prontos para análise de negócio (Camada de Serviço).
* **Modelagem Analítica:**
    * **Agregação:** O dataset é lido do DuckDB e agrupado para calcular o **`valor_medio_venda`** por diferentes dimensões (Região, Estado, Município, Produto, Ano e Mês).
    * **Feature Engineering:** Separação das colunas `ano`, `mes` e `dia`, e criação do campo `regiao` com o nome por extenso, a partir da sigla.
* **Saída:** O resultado final, totalmente modelado e sumarizado, é exportado para o formato **Parquet** (`analitico_dados_combustivel.parquet`) e salvo no diretório `Ouro/`.

---
