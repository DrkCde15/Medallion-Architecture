# Medallion Architecture - Pipeline de E-commerce

Projeto de data engineering utilizando a arquitetura **Medallion** (Bronze → Silver → Gold) para processar dados de um e-commerce, desde a ingestão de CSVs brutos até agregações prontas para análise/BI.

## 📁 Estrutura do Projeto

```
Medallion-Architecture/
├── dataset/                  # Arquivos CSV de origem (dados brutos)
│   ├── customers.csv
│   ├── orders.csv
│   ├── order_items.csv
│   ├── payments.csv
│   ├── products.csv
│   └── reviews.csv
├── Bronse/                   # Notebook - Camada Bronze (ingestão)
├── Silver/                   # Notebook - Camada Silver (limpeza e transformação)
├── Gold/                     # Notebook - Camada Gold (agregações e análise)
└── README.md
```

## 🏗️ Arquitetura Medallion

| Camada | Notebook | Função | Schema (Unity Catalog) |
| --- | --- | --- | --- |
| **Bronze** | [Bronse](#notebook-3306293912755106) | Ingestão de CSVs brutos e persistência em Delta Lake | `bronze` |
| **Silver** | [Silver](#notebook-3306293912755110) | Limpeza, tipagem, formatação e normalização de dados | `silver` |
| **Gold** | [Gold](#notebook-3306293912755111) | Agregações e tabelas analíticas prontas para BI | `gold` |

## 📊 Dados de Origem (Dataset)

O dataset contém 6 tabelas de um e-commerce fictício:

| Arquivo | Descrição | Registros |
| --- | --- | --- |
| `customers.csv` | Dados cadastrais de clientes | 508 |
| `orders.csv` | Pedidos realizados | 1.810 |
| `order_items.csv` | Itens de cada pedido | 4.482 |
| `payments.csv` | Pagamentos associados aos pedidos | 1.800 |
| `products.csv` | Catálogo de produtos (15 categorias) | 15 |
| `reviews.csv` | Avaliações de produtos pelos clientes | 1.300 |

## 🔄 Fluxo de Processamento

### Camada Bronze — Ingestão

Lê os 6 arquivos CSV com Spark e os persiste como **tabelas Delta** no schema `bronze`, adicionando uma coluna `_ingested_at` com timestamp de ingestão para rastreabilidade.

- Formato: Delta Lake (transações ACID)
- Modo: `overwrite`
- Tabelas criadas: `bronze.payments`, `bronze.customers`, `bronze.orders`, `bronze.order_items`, `bronze.products`, `bronze.reviews`

### Camada Silver — Transformação e Limpeza

Lê as tabelas da Bronze e aplica as seguintes transformações:

- **Tipagem correta**: conversão de strings para `decimal`, `int`, `date`
- **Formatação de moeda**: valores no padrão brasileiro (`R$ 1.981,30` / `-R$ 50,00`)
- **Normalização de timestamps**: conversão de UTC para `America/Sao_Paulo`
- **Limpeza de IDs**: remoção de prefixos redundantes (`PAY0*`, `O0*`, `C0*`, `P0*`, `R0*`)
- **Tratamento de nulos**: preenchimento com valores padrão (`"não informado"`, `"desconhecido"`, `"Produto sem nome"`, etc.)
- **Padronização de texto**: `lower()` e `trim()` em colunas categóricas

Tabelas criadas: `silver.payments`, `silver.customers`, `silver.orders`, `silver.order_items`, `silver.products`, `silver.reviews`

### Camada Gold — Agregações e Análise

Lê as tabelas da Silver e cria tabelas agregadas prontas para consumo em BI/análise:

| Tabela Gold | Descrição |
| --- | --- |
| `gold.vendas_por_categoria` | Receita total, itens vendidos e pedidos por categoria de produto |
| `gold.pedidos_por_status` | Total de pedidos, receita e ticket médio por status (delivered, shipped, etc.) |
| `gold.avaliacao_produto` | Avaliação média, total de reviews e notas mín/máx por produto |
| `gold.resumo_clientes` | Total gasto, ticket médio, primeiro e último pedido por cliente |

## 🛠️ Tecnologias

- **Databricks** (notebooks PySpark)
- **Delta Lake** (formato de armazenamento com ACID)
- **Unity Catalog** (schemas `bronze`, `silver`, `gold`)
- **Apache Spark** (processamento distribuído)

## ▶️ Como Executar

Execute os notebooks em ordem, respeitando a dependência entre as camadas:

1. **Bronse** → ingere os CSVs e cria as tabelas Delta na Bronze
2. **Silver** → lê a Bronze, transforma e persiste na Silver
3. **Gold** → lê a Silver, agrega e persiste na Gold

> ⚠️ Cada camada depende da anterior. Execute sempre na sequência Bronze → Silver → Gold.
