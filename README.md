# Teste Técnico - Data Scientist

## Objetivo 
O objetivo é fazer uma análise exploratória a respeito dos dados de licitações de órgãos públicos do Estado do Rio Grande do Sul.

## Compreensão dos dados

### Importando das bibliotecas
Antes de qualquer preparação, é necessário importar as bibliotecas Python contendo a funcionalidade necessária que será utilizada.

```
import pandas as pd
import numpy as np
import re
import string
import seaborn as sns
import matplotlib.pyplot as plt
```

### Obtendo o conjunto de dados

Os arquivos .csv referentes as licitacões e itens de cada ano foram unidos em dois conjuntos de dados.

```
def concatenate(files):
  aux = pd.DataFrame()
  for file in files:
    df = pd.read_csv(file, encoding='utf8')
    aux = pd.concat([aux, df], ignore_index=True)
  return aux
 
from glob import glob

files = sorted(glob('item*.csv'))
item = concatenate(files)
files = sorted(glob('licitacao*.csv'))
licitacao = concatenate(files)
```

Além disso, vamos ver o tipo de dados e seus tipos relacionados:

```
print(licitacao.info())
print(item.info())
```
```
<class 'pandas.core.frame.DataFrame'>
RangeIndex: 237011 entries, 0 to 237010
Data columns (total 61 columns):
 #   Column                       Non-Null Count   Dtype  
---  ------                       --------------   -----  
 0   CD_ORGAO                     237011 non-null  int64  
 1   NM_ORGAO                     237011 non-null  object 
 2   NR_LICITACAO                 237011 non-null  float64
 3   ANO_LICITACAO                237011 non-null  int64  
 4   CD_TIPO_MODALIDADE           237011 non-null  object 
 5   NR_COMISSAO                  153245 non-null  float64
 6   ANO_COMISSAO                 153245 non-null  float64
 7   TP_COMISSAO                  153245 non-null  object 
 8   NR_PROCESSO                  236858 non-null  object 
 9   ANO_PROCESSO                 236857 non-null  float64
 10  TP_OBJETO                    237011 non-null  object 
 11  CD_TIPO_FASE_ATUAL           237011 non-null  object 
 12  TP_LICITACAO                 237011 non-null  object 
 13  TP_NIVEL_JULGAMENTO          237011 non-null  object 
 14  DT_AUTORIZACAO_ADESAO        1747 non-null    object 
 15  TP_CARACTERISTICA_OBJETO     237011 non-null  object 
 16  TP_NATUREZA                  237011 non-null  object 
 17  TP_REGIME_EXECUCAO           50483 non-null   object 
 18  BL_PERMITE_SUBCONTRATACAO    162652 non-null  object 
 19  TP_BENEFICIO_MICRO_EPP       237011 non-null  object 
 20  TP_FORNECIMENTO              123411 non-null  object 
 21  TP_ATUACAO_REGISTRO          24035 non-null   object 
 22  NR_LICITACAO_ORIGINAL        1959 non-null    object 
 23  ANO_LICITACAO_ORIGINAL       41740 non-null   float64
 24  NR_ATA_REGISTRO_PRECO        1952 non-null    object 
 25  DT_ATA_REGISTRO_PRECO        1933 non-null    object 
 26  PC_TAXA_RISCO                50217 non-null   float64
 27  TP_EXECUCAO                  60574 non-null   object 
 28  TP_DISPUTA                   586 non-null     object 
 29  TP_PREQUALIFICACAO           566 non-null     object 
 30  BL_INVERSAO_FASES            237011 non-null  object 
 31  TP_RESULTADO_GLOBAL          35827 non-null   object 
 32  CNPJ_ORGAO_GERENCIADOR       13084 non-null   float64
 33  NM_ORGAO_GERENCIADOR         13081 non-null   object 
 34  DS_OBJETO                    237011 non-null  object 
 35  CD_TIPO_FUNDAMENTACAO        97756 non-null   object 
 36  NR_ARTIGO                    29757 non-null   float64
 37  DS_INCISO                    9323 non-null    object 
 38  DS_LEI                       9440 non-null    object 
 39  DT_INICIO_INSCR_CRED         2681 non-null    object 
 40  DT_FIM_INSCR_CRED            2707 non-null    object 
 41  DT_INICIO_VIGEN_CRED         2468 non-null    object 
 42  DT_FIM_VIGEN_CRED            2464 non-null    object 
 43  VL_LICITACAO                 227822 non-null  float64
 44  BL_ORCAMENTO_SIGILOSO        99785 non-null   object 
 45  BL_RECEBE_INSCRICAO_PER_VIG  22967 non-null   object 
 46  BL_PERMITE_CONSORCIO         237011 non-null  object 
 47  DT_ABERTURA                  236973 non-null  object 
 48  DT_HOMOLOGACAO               138645 non-null  object 
 49  DT_ADJUDICACAO               131955 non-null  object 
 50  BL_LICIT_PROPRIA_ORGAO       237011 non-null  object 
 51  TP_DOCUMENTO_FORNECEDOR      59942 non-null   object 
 52  NR_DOCUMENTO_FORNECEDOR      59942 non-null   object 
 53  TP_DOCUMENTO_VENCEDOR        18501 non-null   object 
 54  NR_DOCUMENTO_VENCEDOR        18501 non-null   object 
 55  VL_HOMOLOGADO                89496 non-null   object 
 56  BL_GERA_DESPESA              237011 non-null  object 
 57  DS_OBSERVACAO                8878 non-null    object 
 58  PC_TX_ESTIMADA               98 non-null      float64
 59  PC_TX_HOMOLOGADA             75 non-null      float64
 60  BL_COMPARTILHADA             237011 non-null  object 
dtypes: float64(11), int64(2), object(48)
memory usage: 110.3+ MB
None
<class 'pandas.core.frame.DataFrame'>
RangeIndex: 3200756 entries, 0 to 3200755
Data columns (total 32 columns):
 #   Column                          Dtype  
---  ------                          -----  
 0   CD_ORGAO                        int64  
 1   NR_LICITACAO                    float64
 2   ANO_LICITACAO                   int64  
 3   CD_TIPO_MODALIDADE              object 
 4   NR_LOTE                         int64  
 5   NR_ITEM                         int64  
 6   NR_ITEM_ORIGINAL                object 
 7   DS_ITEM                         object 
 8   QT_ITENS                        object 
 9   SG_UNIDADE_MEDIDA               object 
 10  VL_UNITARIO_ESTIMADO            float64
 11  VL_TOTAL_ESTIMADO               float64
 12  DT_REF_VALOR_ESTIMADO           object 
 13  PC_BDI_ESTIMADO                 float64
 14  PC_ENCARGOS_SOCIAIS_ESTIMADO    float64
 15  CD_FONTE_REFERENCIA             object 
 16  DS_FONTE_REFERENCIA             object 
 17  TP_RESULTADO_ITEM               object 
 18  VL_UNITARIO_HOMOLOGADO          float64
 19  VL_TOTAL_HOMOLOGADO             object 
 20  PC_BDI_HOMOLOGADO               float64
 21  PC_ENCARGOS_SOCIAIS_HOMOLOGADO  float64
 22  TP_ORCAMENTO                    object 
 23  CD_TIPO_FAMILIA                 float64
 24  CD_TIPO_SUBFAMILIA              float64
 25  TP_DOCUMENTO                    object 
 26  NR_DOCUMENTO                    object 
 27  TP_DOCUMENTO.1                  object 
 28  NR_DOCUMENTO.1                  object 
 29  TP_BENEFICIO_MICRO_EPP          object 
 30  PC_TX_ESTIMADA                  float64
 31  PC_TX_HOMOLOGADA                float64
dtypes: float64(12), int64(4), object(16)
memory usage: 781.4+ MB
```

### Limpeza e preparação dos dados

Inicialmente foi selecionado algumas colunas de interesse e depois a filtragem do tipo de objeto da licitação. Foi utilizado apenas TP_OBJETO=”Compras”.

```
licitacao = licitacao.loc[licitacao['TP_OBJETO'] == 'COM'].reset_index(drop=True)
licitacao = licitacao[['CD_ORGAO', 'ANO_LICITACAO', 'NM_ORGAO', 'DS_OBJETO', 'TP_OBJETO']]
licitacao.head()
```

Em seguida,  foi identificado os dados faltantes:

```
licitacao.isnull().sum()
item.isnull().sum()
```
Como cada órgão cadastra os itens licitados em campos de texto livre no sistema (colunas DS_OBJETO e DS_ITEM das tabelas fornecidas) e não há uma padronização, então foi feita a remoção de linhas com dados faltantes ou inválidos. Além disso, os textos foram convertidos para minúsculos, houve a remoção das pontuações e remoção de partes dos textos para obter maior semelhança entre os itens e facilitar a análise.

![image](https://user-images.githubusercontent.com/32077255/128227027-063c4e49-cf64-40a0-aea0-1505d64841cd.png)

#### Transformação das variáveis categóricas em variáveis numéricas

Para realizar a contagem dos objetos em DS_OBJETO é necessário que o formato do dado seja numérico. Dessa forma, a coluna DS_OBJETOS foi convertida para valores inteiros e foi representada como 'category_id'

![image](https://user-images.githubusercontent.com/32077255/128227493-2e3210f1-533f-4066-b619-cccee3a48f0b.png)

#### Análise e exploração dos dados.

A fim de descobrir quais foram os 10 objetos mais comprados pelos órgãos públicos, então foi obtida a frequência de cada objeto no dataframe licitacao. A coluna index representa o valor numérico associado a um determinado objeto.

![image](https://user-images.githubusercontent.com/32077255/128228128-552100df-676f-4235-9b03-bd084f956431.png)

### Visualização dos dados 

O gráfico a seguir representa os 10 objetos mais comprados pelos órgãos públicos entre 2016 e 2019.

![image](https://user-images.githubusercontent.com/32077255/128231602-0a56b144-518c-47ff-8588-98a4e49d3cef.png)

Ademais, foram obtidos gráficos relacionados a cada ano para obter uma visão mais aprofundada e encontrar algum padrão nas compras.

![image](https://user-images.githubusercontent.com/32077255/128229297-7fdc95a3-1651-4863-b1f9-a2e7af169ac2.png)
![image](https://user-images.githubusercontent.com/32077255/128229376-2c5b26ba-cbdd-48ea-a927-3ba4a933bded.png)

## Conclusão

Tendo em vista os gráficos apresentados é possível observar que o objeto descrito como 'materiais' foi o mais comprado entre os anos de 2016-2019. Com relação aos gráficos por ano, os objetos 'materiais' e 'medicamentos' foram os mais adquiridos por cada um dos anos. Além disso, os objetos 'merenda escolar', 'materiais de expediente', 'materiais elétricos', 'tubos de concreto', 'gêneros alimentícios' e 'materiais de construção' estão presentes entre os 10 mais comprados de cada um dos anos.

Houve uma tentativa em realizar uma análise referente as compras por época do ano a partir da coluna DT_HOMOLOGACAO e por tipo de órgão comprador a fim de identificar algum padrão, mas o tempo acabou não sendo suficiente. 
