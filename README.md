# Análise dos Dados de Saneamento por Setores Censitários do Espírito Santo — Censo 2022

## Visão geral

Este projeto organiza, trata, integra e espacializa dados de saneamento do Censo Demográfico 2022 para os setores censitários do Estado do Espírito Santo.

O fluxo foi desenvolvido em Python, com execução em Google Colab, utilizando principalmente `pandas`, `geopandas`, `matplotlib` e `mapclassify`.

O produto final é uma base geoespacial contendo:

* a geometria dos setores censitários do Espírito Santo;
* a identificação única de cada setor pelo campo `CD_SETOR`;
* variáveis de saneamento relacionadas às moradias;
* variáveis de saneamento relacionadas à população residente;
* indicadores derivados para análise territorial;
* identificação dos setores com e sem dados de saneamento.

## Objetivos

O projeto tem como objetivos:

1. consolidar os dados de saneamento distribuídos em diferentes arquivos;
2. selecionar os setores censitários do Espírito Santo;
3. padronizar a chave territorial `CD_SETOR`;
4. substituir os valores protegidos por sigilo estatístico, representados por `X`, pelo valor `4`, conforme decisão metodológica do projeto;
5. atualizar os títulos das colunas a partir do dicionário de variáveis;
6. integrar os dados tabulares à geometria dos setores censitários;
7. gerar indicadores territoriais e mapas temáticos;
8. exportar os resultados em formatos tabulares e geoespaciais.

## Fontes de dados

### Dados tabulares

Foram utilizados os seguintes arquivos:

* `DADOS SANEAMENTO CENSO 2022_B.xlsx`;
* `DADOS SANEAMENTO CENSO 2022_C.xlsx`;
* `dicionario_de_dados_agregados_por_setores_censitarios_20260520 (1).xlsx`;
* `SETORES CENSITARIOS RURAIS ES 2022 (1).xlsx`.

Os arquivos `B` e `C` foram previamente filtrados para conter apenas variáveis relacionadas ao saneamento.

### Dados geoespaciais

Foi utilizado o shapefile dos setores censitários do Espírito Santo:

```text
ES_setores_CD2022.zip
```

O arquivo contém a geometria dos setores e os atributos necessários à identificação territorial.

## Variáveis utilizadas

### Moradias

| Código   | Descrição resumida                              |
| -------- | ----------------------------------------------- |
| `V00309` | Rede geral, rede pluvial ou solução equivalente |
| `V00310` | Fossa séptica ou filtro ligado à rede           |
| `V00311` | Fossa séptica ou filtro não ligado à rede       |
| `V00312` | Fossa rudimentar ou buraco                      |
| `V00313` | Vala                                            |
| `V00314` | Rio, lago, córrego ou mar                       |
| `V00315` | Outra forma de destinação                       |
| `V00316` | Sem banheiro, sanitário ou destinação definida  |

### População

| Código   | Descrição resumida                                                   |
| -------- | -------------------------------------------------------------------- |
| `V00580` | População em moradias ligadas à rede geral ou pluvial                |
| `V00581` | População em moradias com fossa ligada à rede                        |
| `V00582` | População em moradias com fossa não ligada à rede                    |
| `V00583` | População em moradias com fossa rudimentar ou buraco                 |
| `V00584` | População em moradias com destinação em vala                         |
| `V00585` | População em moradias com lançamento em rio, lago, córrego ou mar    |
| `V00586` | População em moradias com outra forma de destinação                  |
| `V00587` | População em moradias sem banheiro, sanitário ou destinação definida |

## Tratamento dos valores `X`

Nos arquivos do IBGE, o valor `X` representa informação não divulgada por regra de proteção do sigilo estatístico.

Para a finalidade específica deste projeto, todos os valores `X` foram substituídos por `4`.

Essa substituição constitui uma imputação metodológica adotada pelo projeto e não representa necessariamente o valor original omitido pelo IBGE.

Os produtos analíticos e cartográficos derivados devem registrar essa decisão metodológica.

## Fluxo de processamento

### 1. Upload dos arquivos

Os arquivos são enviados ao Google Colab e armazenados nas pastas:

```text
/content/dados/
/content/dados_geoespaciais/
```

### 2. Validação

O notebook verifica:

* existência dos arquivos;
* nomes das abas;
* linha de cabeçalho;
* presença da chave territorial;
* quantidade de registros;
* duplicidades;
* tipos de dados;
* presença e validade das geometrias.

### 3. Padronização territorial

A chave de integração é padronizada como:

```text
CD_SETOR
```

Os códigos são tratados como texto numérico com 15 dígitos.

### 4. Consolidação dos dados

Os arquivos de saneamento `B` e `C` são combinados em um único dataset, preservando:

* uma linha por setor censitário;
* todas as variáveis temáticas;
* setores existentes apenas em um dos arquivos;
* rastreabilidade da origem dos registros.

### 5. Recorte do Espírito Santo

São selecionados os setores cujo código começa por:

```text
32
```

O prefixo `32` identifica o Estado do Espírito Santo.

### 6. Substituição de `X` por `4`

Os valores iguais a `X` ou `x` são substituídos por `4`.

As variáveis temáticas são posteriormente convertidas para formato numérico.

### 7. Atualização dos títulos

Os códigos das variáveis são associados às descrições existentes na aba:

```text
Dicionário não PCT
```

Os títulos seguem o padrão:

```text
V00309 - descrição completa da variável
```

### 8. Integração com a geometria

A geometria dos setores censitários é integrada ao dataset por meio de uma junção baseada em:

```text
CD_SETOR
```

A junção preserva todos os setores existentes no shapefile.

Os setores sem correspondência nos dados de saneamento permanecem no produto final com atributos nulos e situação:

```text
SEM_DADOS
```

### 9. Sistemas de referência

O shapefile original utiliza:

```text
EPSG:4674 — SIRGAS 2000
```

Para cálculos e representação cartográfica em coordenadas métricas, foi criada uma cópia em:

```text
EPSG:31984 — SIRGAS 2000 / UTM zona 24S
```

### 10. Indicadores derivados

Foram calculados:

```text
TOTAL_MORADIAS
MORADIAS_INADEQUADAS
PCT_MORADIAS_INADEQUADAS

TOTAL_POPULACAO
POPULACAO_INADEQUADA
PCT_POPULACAO_INADEQUADA
```

Foram consideradas potencialmente inadequadas as seguintes categorias:

* fossa rudimentar ou buraco;
* vala;
* lançamento em rio, lago, córrego ou mar;
* outra forma de destinação;
* ausência de banheiro, sanitário ou destinação definida.

## Produtos gerados

### Dados tabulares

```text
DADOS SANEAMENTO ES 2022.csv
DADOS SANEAMENTO ES 2022.parquet
DADOS SANEAMENTO ES 2022_X_SUBSTITUIDO_POR_4.csv
DADOS SANEAMENTO ES 2022_X_SUBSTITUIDO_POR_4.parquet
DADOS SANEAMENTO ES 2022_TITULOS_ATUALIZADOS.xlsx
MAPEAMENTO TITULOS DADOS SANEAMENTO ES 2022.csv
```

### Dados geoespaciais

```text
SANEAMENTO_SETORES_CENSITARIOS_ES_2022.gpkg
SANEAMENTO_SETORES_ES_2022_SHP.zip
```

### Produtos cartográficos

```text
MAPA_PERCENTUAL_MORADIAS_INADEQUADAS_ES_2022.png
MAPA_PERCENTUAL_POPULACAO_INADEQUADA_ES_2022.png
MAPA_PERCENTUAL_MORADIAS_INADEQUADAS_ES_2022_APRIMORADO.png
MAPA_PERCENTUAL_MORADIAS_INADEQUADAS_ES_2022_APRIMORADO.pdf
```

## Diagnóstico da integração

O diagnóstico realizado identificou:

* `8.568` setores com dados de saneamento e geometria;
* `138` setores existentes apenas no shapefile;
* `0` setores existentes apenas no dataset tabular;
* `0` geometrias inválidas;
* `16` variáveis temáticas disponíveis;
* cobertura integral da geometria para os setores presentes no dataset.

Os 138 setores adicionais permanecem no arquivo geoespacial com atributos nulos e indicação de ausência de dados.

## Estrutura sugerida do repositório

```text
projeto-saneamento-setores-es/
├── README.md
├── notebooks/
│   └── saneamento_setores_es_2022.ipynb
├── dados/
│   ├── entrada/
│   └── processados/
├── dados_geoespaciais/
│   ├── entrada/
│   └── processados/
├── mapas/
├── scripts/
└── docs/
```

Recomenda-se não versionar diretamente arquivos muito grandes no GitHub.

Para arquivos que ultrapassem o limite do repositório, considerar:

* Git Large File Storage — Git LFS;
* Google Drive;
* armazenamento institucional;
* repositório de dados separado.

## Dependências

O projeto utiliza Python 3 e as seguintes bibliotecas:

```text
pandas
numpy
openpyxl
pyarrow
geopandas
shapely
fiona
pyproj
matplotlib
mapclassify
```

Instalação no Google Colab:

```python
!pip install pandas numpy openpyxl pyarrow geopandas shapely fiona pyproj matplotlib mapclassify -q
```

## Execução

1. Abra o notebook no Google Colab.
2. Execute os blocos na ordem definida.
3. Faça upload dos arquivos tabulares.
4. Faça upload do ZIP contendo o shapefile.
5. Valide os arquivos e as chaves territoriais.
6. Consolide os arquivos de saneamento.
7. Selecione os setores do Espírito Santo.
8. Substitua os valores `X` por `4`.
9. Atualize os títulos das colunas.
10. Integre os dados à geometria.
11. Calcule os indicadores derivados.
12. Gere os mapas e exporte os arquivos finais.

## Limitações

* A substituição de `X` por `4` é uma decisão metodológica específica deste projeto.
* Os resultados não devem ser interpretados como reprodução exata dos valores sigilosos do IBGE.
* Setores sem dados de saneamento permanecem com valores nulos.
* O formato Shapefile limita os nomes dos campos a 10 caracteres.
* O GeoPackage deve ser priorizado quando for necessário preservar os nomes completos dos atributos.
* Os percentuais calculados dependem da disponibilidade das variáveis divulgadas.

## Recomendações de uso

O GeoPackage é o formato recomendado para análises no QGIS porque:

* preserva nomes completos de campos;
* suporta maior quantidade de atributos;
* evita limitações do Shapefile;
* apresenta melhor compatibilidade com valores nulos;
* permite armazenar múltiplas camadas em um único arquivo.

O Shapefile deve ser utilizado quando houver exigência de compatibilidade com sistemas legados.

## Aplicações

Os resultados podem apoiar:

* diagnóstico territorial do esgotamento sanitário;
* priorização de áreas para saneamento rural;
* identificação de setores com maior precariedade;
* planejamento de investimentos;
* formulação de políticas públicas;
* cruzamento com comunidades, imóveis rurais, bacias hidrográficas e redes de drenagem;
* produção de mapas temáticos;
* integração com QGIS e outros sistemas de informação geográfica.

## Responsável

**Robson Monteiro dos Santos**

Projeto desenvolvido no contexto de estudos e análises da Secretaria de Estado do Meio Ambeinte e dos Recursos Hídricos do Estado do Espírito Santo para fins de avaliaçaõ das condições do saneamento rural.

## Licença

A licença do repositório deverá ser definida antes da publicação.

Sugestões:

* `MIT` para os códigos;
* `CC BY 4.0` para a documentação e os produtos derivados;
* observância das condições de uso e atribuição aplicáveis às bases do IBGE.
  ::: 
