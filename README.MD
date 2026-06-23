# Sanitation Data Analysis by Census Sectors in Espírito Santo — 2022 Census

## Overview

This project organizes, processes, integrates, and spatializes sanitation data from the 2022 Brazilian Demographic Census for the census sectors of the State of Espírito Santo.

The workflow was developed in Python and executed in Google Colab, mainly using `pandas`, `geopandas`, `matplotlib`, and `mapclassify`.

The final product is a geospatial dataset containing:

* the geometry of the census sectors of Espírito Santo;
* the unique identification of each sector through the `CD_SETOR` field;
* sanitation variables related to households;
* sanitation variables related to the resident population;
* derived indicators for territorial analysis;
* identification of sectors with and without sanitation data.

---

## Objectives

The project aims to:

1. consolidate sanitation data distributed across different files;
2. select the census sectors of Espírito Santo;
3. standardize the territorial key `CD_SETOR`;
4. replace values protected by statistical confidentiality, represented by `X`, with the value `4`, according to the methodological decision adopted in the project;
5. update column titles using the official variable dictionary;
6. integrate tabular data with census-sector geometries;
7. generate territorial indicators and thematic maps;
8. export the results in tabular and geospatial formats.

---

## Data sources

### Tabular data

The following files were used:

* `DADOS SANEAMENTO CENSO 2022_B.xlsx`;
* `DADOS SANEAMENTO CENSO 2022_C.xlsx`;
* `dicionario_de_dados_agregados_por_setores_censitarios_20260520 (1).xlsx`;
* `SETORES CENSITARIOS RURAIS ES 2022 (1).xlsx`.

Files `B` and `C` were filtered before upload so that they contained only sanitation-related variables.

### Geospatial data

The census-sector shapefile for Espírito Santo was used:

```text
ES_setores_CD2022.zip
```

The file contains the sector geometries and the attributes required for territorial identification.

---

## Variables used

### Households

| Code     | Short description                                                 |
| -------- | ----------------------------------------------------------------- |
| `V00309` | General sewer network, stormwater network, or equivalent solution |
| `V00310` | Septic tank or filter connected to the network                    |
| `V00311` | Septic tank or filter not connected to the network                |
| `V00312` | Rudimentary pit or hole                                           |
| `V00313` | Ditch                                                             |
| `V00314` | River, lake, stream, or sea                                       |
| `V00315` | Other disposal method                                             |
| `V00316` | No bathroom, toilet, or defined disposal method                   |

### Population

| Code     | Short description                                                               |
| -------- | ------------------------------------------------------------------------------- |
| `V00580` | Population in households connected to a general or stormwater network           |
| `V00581` | Population in households with a septic tank connected to the network            |
| `V00582` | Population in households with a septic tank not connected to the network        |
| `V00583` | Population in households using a rudimentary pit or hole                        |
| `V00584` | Population in households disposing into a ditch                                 |
| `V00585` | Population in households discharging into a river, lake, stream, or sea         |
| `V00586` | Population in households using another disposal method                          |
| `V00587` | Population in households without a bathroom, toilet, or defined disposal method |

---

## Treatment of `X` values

In IBGE datasets, the value `X` represents information that was not disclosed because of statistical confidentiality rules.

For the specific purpose of this project, all `X` values were replaced with `4`.

This replacement is a project-specific methodological imputation and does not necessarily represent the original value withheld by IBGE.

All analytical and cartographic products derived from the dataset should explicitly document this methodological decision.

---

## Processing workflow

### 1. File upload

The files are uploaded to Google Colab and stored in:

```text
/content/dados/
/content/dados_geoespaciais/
```

### 2. Validation

The notebook checks:

* file existence;
* worksheet names;
* header rows;
* presence of the territorial key;
* number of records;
* duplicate records;
* data types;
* presence and validity of geometries.

### 3. Territorial standardization

The integration key is standardized as:

```text
CD_SETOR
```

Codes are handled as 15-digit numeric strings.

### 4. Data consolidation

Sanitation files `B` and `C` are combined into a single dataset while preserving:

* one row per census sector;
* all sanitation variables;
* sectors present in only one source file;
* traceability of the source of each record.

### 5. Espírito Santo selection

Census sectors whose codes begin with the following prefix are selected:

```text
32
```

The prefix `32` identifies the State of Espírito Santo.

### 6. Replacement of `X` with `4`

Values equal to `X` or `x` are replaced with `4`.

The thematic variables are then converted to numeric format.

### 7. Column-title update

Variable codes are matched with the descriptions available in the worksheet:

```text
Dicionário não PCT
```

Column titles follow the pattern:

```text
V00309 - full variable description
```

### 8. Geometry integration

Census-sector geometries are joined to the final dataset through:

```text
CD_SETOR
```

The join preserves all sectors available in the shapefile.

Sectors without matching sanitation records remain in the final geospatial dataset with null attributes and the status:

```text
SEM_DADOS
```

### 9. Coordinate reference systems

The original shapefile uses:

```text
EPSG:4674 — SIRGAS 2000
```

For calculations and cartographic representation in metric coordinates, a projected copy was created using:

```text
EPSG:31984 — SIRGAS 2000 / UTM zone 24S
```

### 10. Derived indicators

The following indicators were calculated:

```text
TOTAL_MORADIAS
MORADIAS_INADEQUADAS
PCT_MORADIAS_INADEQUADAS

TOTAL_POPULACAO
POPULACAO_INADEQUADA
PCT_POPULACAO_INADEQUADA
```

The following categories were considered potentially inadequate:

* rudimentary pit or hole;
* ditch;
* discharge into a river, lake, stream, or sea;
* other disposal method;
* absence of a bathroom, toilet, or defined disposal method.

---

## Generated products

### Tabular data

```text
DADOS SANEAMENTO ES 2022.csv
DADOS SANEAMENTO ES 2022.parquet
DADOS SANEAMENTO ES 2022_X_SUBSTITUIDO_POR_4.csv
DADOS SANEAMENTO ES 2022_X_SUBSTITUIDO_POR_4.parquet
DADOS SANEAMENTO ES 2022_TITULOS_ATUALIZADOS.xlsx
MAPEAMENTO TITULOS DADOS SANEAMENTO ES 2022.csv
```

### Geospatial data

```text
SANEAMENTO_SETORES_CENSITARIOS_ES_2022.gpkg
SANEAMENTO_SETORES_ES_2022_SHP.zip
```

### Cartographic products

```text
MAPA_PERCENTUAL_MORADIAS_INADEQUADAS_ES_2022.png
MAPA_PERCENTUAL_POPULACAO_INADEQUADA_ES_2022.png
MAPA_PERCENTUAL_MORADIAS_INADEQUADAS_ES_2022_APRIMORADO.png
MAPA_PERCENTUAL_MORADIAS_INADEQUADAS_ES_2022_APRIMORADO.pdf
```

---

## Integration diagnostics

The completed diagnostic identified:

* `8,568` sectors with both sanitation data and geometry;
* `138` sectors present only in the shapefile;
* `0` sectors present only in the tabular dataset;
* `0` invalid geometries;
* `16` thematic variables available;
* complete geometry coverage for all sectors present in the sanitation dataset.

The 138 additional sectors remain in the geospatial output with null sanitation attributes and an indication that data are unavailable.

---

## Suggested repository structure

```text
sanitation-census-sectors-es/
├── README.md
├── notebooks/
│   └── sanitation_census_sectors_es_2022.ipynb
├── data/
│   ├── raw/
│   └── processed/
├── geospatial_data/
│   ├── raw/
│   └── processed/
├── maps/
├── scripts/
└── docs/
```

Large files should not be committed directly to a standard GitHub repository.

For files that exceed repository limits, consider using:

* Git Large File Storage — Git LFS;
* Google Drive;
* institutional storage;
* a separate data repository.

---

## Dependencies

The project uses Python 3 and the following libraries:

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

Installation in Google Colab:

```python
!pip install pandas numpy openpyxl pyarrow geopandas shapely fiona pyproj matplotlib mapclassify -q
```

---

## Running the project

1. Open the notebook in Google Colab.
2. Run the notebook cells in the defined order.
3. Upload the tabular files.
4. Upload the ZIP file containing the shapefile.
5. Validate the files and territorial keys.
6. Consolidate the sanitation files.
7. Select the census sectors of Espírito Santo.
8. Replace `X` values with `4`.
9. Update the column titles.
10. Join the sanitation data to the census-sector geometries.
11. Calculate the derived indicators.
12. Generate the maps and export the final files.

---

## Limitations

* Replacing `X` with `4` is a project-specific methodological decision.
* The results must not be interpreted as an exact reconstruction of confidential values withheld by IBGE.
* Sectors without sanitation data remain with null values.
* The Shapefile format limits field names to 10 characters.
* GeoPackage should be preferred when full attribute names need to be preserved.
* Calculated percentages depend on the availability and completeness of the variables published by IBGE.

---

## Recommended use

GeoPackage is the recommended format for analyses in QGIS because it:

* preserves complete field names;
* supports a larger number of attributes;
* avoids Shapefile field-name limitations;
* handles null values more consistently;
* allows multiple layers to be stored in a single file.

Shapefile should be used only when compatibility with legacy systems is required.

---

## Applications

The outputs may support:

* territorial diagnosis of wastewater disposal conditions;
* prioritization of rural sanitation interventions;
* identification of census sectors with higher sanitation vulnerability;
* investment planning;
* public-policy development;
* integration with communities, rural properties, watersheds, and drainage networks;
* thematic mapping;
* use in QGIS and other geographic information systems.

---

## Author

**Robson Monteiro dos Santos**

Project developed within the context of rural sanitation studies and analyses in Environmental and Water Resource Secretariat at State of Espírito Santo, Brazil.

---

## License

Opensource for public entitites and students.

Suggested options:

* `MIT` for source code;
* `CC BY 4.0` for documentation and derived products;
* compliance with the applicable terms of use and attribution requirements for IBGE datasets.
  ::: 
