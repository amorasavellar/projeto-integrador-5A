# 🏥 Projeto Integrador 5A - Análise de Dados Oftalmológicos (SUS)

## 📋 Visão Geral

**Projeto Integrador 5A** é uma solução completa de **análise e visualização de dados de saúde oftalmológica** do Sistema Único de Saúde (SUS) do Brasil. O projeto implementa um pipeline de dados moderno utilizando arquitetura **Medallion** (Bronze → Silver → Gold) na plataforma **Databricks**, com visualizações interativas em **Power BI**.

### 🎯 Objetivo Principal
Integrar, transformar e visualizar dados de procedimentos oftalmológicos provenientes do SIA (Sistema de Informações Ambulatoriais) e SIH (Sistema de Informações Hospitalares), combinados com dados demográficos do IBGE e tabelas de procedimentos (SIGTAP).

---

## 🏗️ Arquitetura do Projeto

```
┌─────────────────────────────────────────────────────────────────────┐
│                         FONTES DE DADOS                             │
│   [SIA] [SIH] [SIGTAP] [IBGE - Municípios]                          │
└────────────────────┬────────────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────────────┐
│              01_LANDING (Ingestion / Raw)                           │
│  • ingest_sia_pa.ipynb        - Dados SIA Procedimentos             │
│  • ingest_sigtap.ipynb        - Tabela de Procedimentos             │
│  • ingest_ibge.ipynb          - Dados Demográficos (Municípios)     │
│  • ingest_sih.ipynb           - Dados SIH (Internações)             │
└────────────────────┬────────────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────────────┐
│              02_BRONZE (Raw Data Store)                             │
│  • load_sia.ipynb             - Validação e armazenamento SIA       │
│  • load_sigtap.ipynb          - Validação e armazenamento SIGTAP    │
│  • load_ibge_municipios.ipynb - Validação dados geográficos         │
│  • load_sih.ipynb             - Validação e armazenamento SIH       │
└────────────────────┬────────────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────────────┐
│              03_SILVER (Transformed Data)                           │
│  • transform_fato_sia.ipynb        - Fatos SIA processados          │
│  • transform_dim_sigtap.ipynb      - Dimensão de Procedimentos      │
│  • transform_dim_municipios.ipynb  - Dimensão de Localidades        │
│  • transform_fato_sih.ipynb        - Fatos SIH processados          │
└────────────────────┬────────────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────────────┐
│              04_GOLD (Aggregated Analytics)                         │
│  • setup_gold_oftalmo.ipynb   - Criação de tabelas analíticas       │
│  Tabelas prontas para BI                                            │
└────────────────────┬────────────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    POWER BI DASHBOARDS                              │
│  • pysus_oftalmo_go.pbip      - Projeto Power BI                    │
│  • Relatórios Interativos e KPIs                                    │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 📊 Estrutura de Dados - Medalha (Medallion Architecture)

### 🥉 BRONZE - Dados Brutos
**Localização**: `dbfs:/mnt/lake/bronze/`

Tabelas de dados originais, sem transformações, apenas validação básica:

| Tabela | Descrição | Fonte |
|--------|-----------|-------|
| `sia` | Procedimentos ambulatoriais oftalmológicos | SUS (SIA) |
| `sigtap` | Tabela de procedimentos com código e descrição | SIGTAP |
| `ibge_municipios` | Dados demográficos de municípios brasileiros | IBGE |
| `sih` | Internações hospitalares | SUS (SIH) |

### 🥈 SILVER - Dados Transformados
**Localização**: `dbfs:/mnt/lake/silver/`

Dados normalizados, com limpeza e estrutura dimensional:

| Tabela | Descrição |
|--------|-----------|
| `fato_sia` | Fatos de procedimentos ambulatoriais com chaves estrangeiras |
| `dim_sigtap` | Dimensão de procedimentos (normalizada) |
| `dim_municipios` | Dimensão de localidades com dados geográficos |
| `fato_sih` | Fatos de internações hospitalares |

### 🥇 GOLD - Dados Analíticos
**Localização**: `dbfs:/mnt/lake/gold/`

Agregações e tabelas otimizadas para análise e BI:

| Tabela | Descrição |
|--------|-----------|
| `analise_oftalmo_por_municipio` | Agregações por município e período |
| `procedimentos_mais_frequentes` | Top procedimentos oftalmológicos |
| `serie_temporal_procedimentos` | Evolução temporal de procedimentos |
| `indicadores_assistenciais` | KPIs de saúde oftalmológica |

---

## 🛠️ Fluxo de Processamento - Passo a Passo

### **Fase 00: Setup Inicial**

#### `00_create_catalogs_schemas.ipynb`
- Cria catálogos Unity no Databricks
- Define schemas (bronze, silver, gold)
- Configura permissões e acessos

#### `01_lake_backfill.ipynb`
- Inicializa volumes (mount points)
- Cria diretórios nas camadas do Data Lake
- Testa conectividade com camadas de armazenamento

---

### **Fase 01: LANDING - Ingestão de Dados**

Notebooks que fazem o download e armazenamento inicial dos dados.

#### `01_ingest_sia_pa.ipynb`
```
[API SUS/DataSUS] → Download procedimentos ambulatoriais → /landing/sia/
```
- Extrai dados de procedimentos oftalmológicos do SIA
- Filtra procedimentos oftalmológicos (códigos específicos)
- Armazena em formato bruto (Parquet ou Delta)

#### `02_ingest_sigtap.ipynb`
```
[SIGTAP Dataset] → Download tabela de procedimentos → /landing/sigtap/
```
- Obtém tabela de códigos e descrições de procedimentos
- Essencial para enriquecimento dos dados SIA

#### `03_ingest_ibge.ipynb`
```
[IBGE API] → Download dados municipais 2022 → /landing/ibge/
```
- Traz dados de municípios, população, região
- Dados geográficos para análise espacial

#### `04_ingest_sih.ipynb`
```
[SIH/DATASUS] → Download internações hospitalares → /landing/sih/
```
- Dados de internações oftalmológicas
- Complementa análise com dados hospitalares

---

### **Fase 02: BRONZE - Validação e Armazenamento**

Notebooks que fazem validação básica e armazenamento estruturado.

#### `01_load_sia.ipynb`
- Lê dados brutos do /landing/sia/
- Validações: duplicatas, nulos, tipos de dados
- Cria tabela `bronze.sia` com metadados
- **Saída**: Tabela Bronze de procedimentos

#### `02_load_sigtap.ipynb`
- Processa tabela de procedimentos
- Remove duplicatas e normaliza códigos
- Cria tabela `bronze.sigtap`
- **Saída**: Dimensão bruta de procedimentos

#### `03_load_ibge_municipios.ipynb`
- Valida dados geográficos
- Normaliza nomes de municípios
- Cria tabela `bronze.ibge_municipios`
- **Saída**: Tabela de referência geográfica

#### `04_load_sih.ipynb`
- Processa dados de internações
- Validação de campos clínicos
- Cria tabela `bronze.sih`
- **Saída**: Fatos de internações

---

### **Fase 03: SILVER - Transformação e Enriquecimento**

Notebooks que transformam dados em formato dimensional.

#### `01_transform_fato_sia.ipynb`
```
bronze.sia + bronze.sigtap + bronze.ibge_municipios 
    ↓
Joins, agregações, criação de chaves surrogate
    ↓
silver.fato_sia (Fato dimensional)
```
- Faz join com dimensões
- Cria SK (Surrogate Keys)
- Trata nulos e outliers
- Cria coluna de data para série temporal

#### `02_transform_dim_sigtap.ipynb`
```
bronze.sigtap → Limpeza e normalização → silver.dim_sigtap
```
- Normaliza nomes e descrições de procedimentos
- Classifica procedimentos por tipo/especialidade
- Cria hierarquias de classificação

#### `03_transform_dim_municipios.ipynb`
```
bronze.ibge_municipios → Enriquecimento geográfico → silver.dim_municipios
```
- Adiciona regiões, estados, microrregiões
- Normaliza nomes de localidades
- Prepara dados para análise espacial

#### `04_transform_fato_sih.ipynb`
```
bronze.sih + silver.dim_municipios + silver.dim_sigtap
    ↓
silver.fato_sih (Fato de internações)
```
- Transforma internações em fatos
- Enriquece com dimensões
- Calcula métricas de permanência

---

### **Fase 04: GOLD - Agregações e Analytics**

#### `00_setup_gold_oftalmo.ipynb`
```
silver.fato_sia + silver.fato_sih + dimensões
    ↓
Agregações e consolidações
    ↓
Tabelas otimizadas para BI
```

**Tabelas GOLD criadas**:

1. **analise_oftalmo_por_municipio**
   - Agregação: município, período, procedimento
   - Métricas: quantidade procedimentos, custo, volume pacientes

2. **procedimentos_mais_frequentes**
   - Ranking de procedimentos por região
   - Tendências temporais

3. **serie_temporal_procedimentos**
   - Dados agrupados por mês/trimestre/ano
   - Permite análise de tendências

4. **indicadores_assistenciais**
   - KPIs consolidados
   - Taxa de procedimentos per capita
   - Cobertura por região

---

## 📈 Power BI - Visualizações

### 📁 Arquivos Power BI
```
/pbi/
├── pysus_oftalmo_go.pbip          # Projeto Power BI principal
├── pysus_oftalmo_go.Report        # Relatórios exportados
├── pysus_oftalmo_go.SemanticModel # Modelo semântico
└── assets/
    ├── dash_sia.png               # Referência visual
    └── Municipios_2022.json       # Dados para mapas
```

### 📊 Dashboards Esperados

1. **Dashboard Executivo**
   - KPIs principais (Total procedimentos, Pacientes, Cobertura)
   - Evolução temporal em gráficos de linha

2. **Análise por Município**
   - Mapa interativo com volume procedimentos
   - Tabela de municípios ordenados por volume

3. **Análise de Procedimentos**
   - Procedimentos mais realizados
   - Análise comparativa períodos

4. **Indicadores de Saúde**
   - Taxa de cobertura
   - Custo médio procedimento
   - Evolução de procedimentos per capita

---

## 🚀 Como Executar o Projeto

### Pré-requisitos
- ✅ Acesso ao Databricks (workspace com Unity Catalog)
- ✅ Clusters configurados (Databricks Runtime 13.3+)
- ✅ Acesso a APIs: DataSUS, IBGE, SIGTAP
- ✅ Power BI Desktop (para visualizações locais)
- ✅ Conexão com Databricks (no Power BI)

### Passo 1: Setup Inicial
```
1. Execute: src/00_setup/00_create_catalogs_schemas.ipynb
2. Execute: src/00_setup/01_lake_backfill.ipynb
```

### Passo 2: Ingestão (LANDING)
Execute todos os notebooks em `src/01_landing/`:
```
1. 01_ingest_sia_pa.ipynb
2. 02_ingest_sigtap.ipynb
3. 03_ingest_ibge.ipynb
4. 04_ingest_sih.ipynb
```

### Passo 3: Bronze Layer
Execute todos os notebooks em `src/02_bronze/`:
```
1. 01_load_sia.ipynb
2. 02_load_sigtap.ipynb
3. 03_load_ibge_municipios.ipynb
4. 04_load_sih.ipynb
```

### Passo 4: Silver Layer
Execute todos os notebooks em `src/03_silver/`:
```
1. 01_transform_fato_sia.ipynb
2. 02_transform_dim_sigtap.ipynb
3. 03_transform_dim_municipios.ipynb
4. 04_transform_fato_sih.ipynb
```

### Passo 5: Gold Layer
Execute o notebook em `src/04_gold/`:
```
1. 00_setup_gold_oftalmo.ipynb
```

### Passo 6: Power BI
1. Abra `pbi/pysus_oftalmo_go.pbip` no Power BI Desktop
2. Configure conexão com Databricks
3. Atualize dados
4. Publique no Power BI Service

---

## 📊 Estrutura de Diretórios

```
projeto-integrador-5A/
│
├── 📂 src/                                # Código Databricks (Notebooks)
│   ├── 00_setup/                         # Setup inicial
│   │   ├── 00_create_catalogs_schemas.ipynb
│   │   └── 01_lake_backfill.ipynb
│   ├── 01_landing/                       # Ingestão de dados
│   │   ├── 01_ingest_sia_pa.ipynb
│   │   ├── 02_ingest_sigtap.ipynb
│   │   ├── 03_ingest_ibge.ipynb
│   │   └── 04_ingest_sih.ipynb
│   ├── 02_bronze/                        # Validação dados brutos
│   │   ├── 01_load_sia.ipynb
│   │   ├── 02_load_sigtap.ipynb
│   │   ├── 03_load_ibge_municipios.ipynb
│   │   └── 04_load_sih.ipynb
│   ├── 03_silver/                        # Transformação dados
│   │   ├── 01_transform_fato_sia.ipynb
│   │   ├── 02_transform_dim_sigtap.ipynb
│   │   ├── 03_transform_dim_municipios.ipynb
│   │   └── 04_transform_fato_sih.ipynb
│   └── 04_gold/                          # Agregações analíticas
│       └── 00_setup_gold_oftalmo.ipynb
│
├── 📂 pbi/                               # Power BI
│   ├── pysus_oftalmo_go.pbip            # Projeto Power BI
│   ├── pysus_oftalmo_go.Report
│   ├── pysus_oftalmo_go.SemanticModel
│   └── assets/
│       ├── Municipios_2022.json
│       └── dash_sia.png
│
├── 📄 README.md                          # Este arquivo
├── 📄 LICENSE                            # MIT License
└── 📄 .gitignore
```

---

## 🔗 Integração Databricks → Power BI

### Fluxo de Dados
```
Databricks Workspace
    ↓
Unity Catalog (Volumes)
    ↓
Tabelas GOLD (analíticas)
    ↓
Connection String Databricks
    ↓
Power BI Desktop
    ↓
DAX Measures & Visualizações
    ↓
Power BI Service (Publicação)
```

### Configuração Power BI
1. **Fonte de Dados**: Databricks SQL Connector
2. **Autenticação**: Databricks Personal Token ou Azure AD
3. **Catalog**: pysus_oftalmo_5a
4. **Schemas**: bronze, silver, gold
5. **Tabelas**: Todas as tabelas das camadas (especialmente gold)

---

## 📋 Dados de Entrada

| Dataset | Fonte | Formato | Frequência |
|---------|-------|---------|-----------|
| **SIA** | DataSUS | CSV/JSON | Mensal |
| **SIH** | DataSUS | CSV/JSON | Mensal |
| **SIGTAP** | DATASUS | CSV | Trimestral |
| **IBGE Municípios** | IBGE API | JSON | Anual (2022) |

---

## 🔐 Segurança e Qualidade

- ✅ **Validação de dados**: Duplicatas, nulos, tipos
- ✅ **Auditoria**: Logs de processamento em cada camada
- ✅ **Versionamento**: Delta Lake com histórico
- ✅ **Rastreabilidade**: Chaves surrogate e timestamps
- ✅ **Conformidade**: Dados públicos do SUS

---

## 👥 Autoria

**Leandro Amoras** - Desenvolvedor  
Licença: MIT License  
Ano: 2026

---
