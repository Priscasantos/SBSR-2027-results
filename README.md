# Dados processados — MapBiomas Col. 11 (2020–2025) e SIDRA/PAM (2000–2024, em preparação)

Repositório público de **dados tabulares processados** usados no artigo do
SBSR 2027 sobre lavouras e biomas brasileiros. Reúne duas famílias de dados:

- **MapBiomas Coleção 11** — áreas de lavouras temporárias (classe sintética 19)
  e perenes (classe sintética 36) no Brasil e por bioma, 2020–2025.
- **SIDRA/PAM (IBGE)** — estatísticas anuais da produção agrícola municipal
  agregadas por bioma, série 2000–2024, **em preparação**: os CSVs serão
  publicados quando disponíveis no pipeline de origem.

O site estático em `docs/` apresenta cada CSV como tabela filtrável e permite
baixar o arquivo bruto. Os dados foram copiados do repositório de código do
projeto (privado), sem recálculo; o código do pipeline está disponível sob
solicitação.

Este repositório é um **recorte curado** dos dados processados: contém apenas
os arquivos publicados e o site estático. Atualizações são feitas manualmente
pelo mantenedor, copiando CSVs/JSONs e atualizando as tabelas estáticas.

## O que não está aqui

Este repositório publica **somente dados tabulares derivados**. Não fazem
parte dele:

- rasters GeoTIFF anuais, reclassificações e pirâmides de overview;
- shapefiles, máscaras vetoriais e estilos de mapa (QML);
- o artigo, seus arquivos-fonte, as figuras e o PDF;
- o código do pipeline, os scripts de download e os ambientes virtuais;
- arquivos de validação interna que contenham caminhos absolutos da máquina
  de origem.

Esses itens permanecem no repositório de código do projeto (privado,
disponível sob solicitação) e podem ser reproduzidos a partir dos scripts do
projeto. O site estático reflete apenas os arquivos Col. 11 e os CSVs do
SIDRA aqui publicados.

## Proveniência

### MapBiomas Coleção 11

- Mosaicos anuais de uso e cobertura da terra do Brasil, 2020–2025, resolução
  de 30 m, `EPSG:4326`, classe `uint8`, NoData 0.
- As classes de lavoura da legenda original foram agrupadas em duas classes
  sintéticas:
  - **19 — Lavoura Temporária**: códigos 20 (Cana), 39 (Soja), 40 (Arroz),
    41 (Outras Lavouras Temporárias) e 62 (Algodão).
  - **36 — Lavoura Perene**: códigos 35 (Dendê), 46 (Café), 47 (Citrus) e
    48 (Outras Lavouras Perenes).
- As contagens de área usam a grade equal-area **Brazil Albers
  `EPSG:10857`**, com pixel de 30 m (900 m²). A reprojeção é feita com
  interpolação *nearest*.
- A máscara de biomas foi rasterizada na mesma grade: 0 = fora dos biomas;
  1 = Amazônia; 2 = Cerrado; 3 = Caatinga; 4 = Mata Atlântica; 5 = Pantanal;
  6 = Pampa.
- Os limites de bioma utilizados no pipeline seguem os shapefiles de origem;
  consulte a documentação do projeto para a fonte original.
- Os arquivos publicados são **agregações**; os rasters de origem não são
  distribuídos aqui.

### SIDRA/PAM (IBGE)

- Produção Agrícola Municipal, tabela **5457** do SIDRA/IBGE, nível municipal,
  série 2000–2024 no pipeline de origem.
- O método primário de agregação por bioma é a **interpolação areal** dos
  municípios, com os pesos calculados por `peso_municipio_bioma.py` e
  aplicados por `agrega_bioma_ponderado.py` (ADR-001).
- A atribuição pelo centroide municipal (`predicate='within'`) é a análise de
  **sensibilidade S-A**, não o método principal.
- **Nesta versão não há CSVs do SIDRA publicados.** A seção será acrescentada
  manualmente pelo mantenedor quando os arquivos estiverem disponíveis no
  repositório de código.

## Estrutura

```
.
├── README.md
├── CITATION.cff
├── LICENSE                      # CC BY-SA 4.0 (dados)
├── .gitignore
└── docs/                        # raiz do GitHub Pages
    ├── .nojekyll
    ├── index.html
    ├── licenca.html
    ├── dados/
    │   ├── col11/
    │   │   ├── areas_lavouras_2020_2025.csv
    │   │   ├── areas_por_bioma_2020_2025.csv
    │   │   ├── classes_col11.csv
    │   │   ├── areas_biomas_mascara.json
    │   │   ├── contagens_fase2_4326.json
    │   │   ├── contagens_fase4_albers.json
    │   │   ├── contagens_por_bioma.json
    │   │   ├── overviews_info.json
    │   │   └── residuos_cobertura_biomas.json
    │   └── sidra/               # criado quando houver CSVs do SIDRA
    └── tabelas/
        ├── areas-lavouras-2020-2025-csv.html
        ├── areas-por-bioma-2020-2025-csv.html
        └── classes-col11-csv.html
```

## Dicionário dos arquivos

### `areas_lavouras_2020_2025.csv`

**18 linhas** (6 anos × 3 registros: classes 19, 36 e total). Área nacional
das lavouras na grade Albers.

| Coluna | Tipo | Descrição |
|---|---|---|
| `ano` | inteiro | Ano de referência, 2020–2025. |
| `classe_id` | inteiro ou texto | `19` = Lavoura Temporária; `36` = Lavoura Perene; `total` = soma das duas. |
| `classe_nome` | texto | Nome legível da classe ou `Total lavouras`. |
| `pixels` | inteiro | Número de pixels de 30 m (900 m² cada). |
| `area_ha` | decimal | Área em hectares: `pixels × 900 / 10.000`. |
| `area_km2` | decimal | Área em km²: `pixels × 900 / 1.000.000`. |

### `areas_por_bioma_2020_2025.csv`

**126 linhas** (6 anos × 7 recortes × 3 registros). Mesmas colunas acima, mais
`bioma`:

| Coluna | Tipo | Descrição |
|---|---|---|
| `ano` | inteiro | Ano de referência, 2020–2025. |
| `bioma` | texto | `Amazônia`, `Cerrado`, `Caatinga`, `Mata Atlântica`, `Pantanal`, `Pampa` ou `Brasil (total)`. |
| `classe_id` | inteiro ou texto | `19`, `36` ou `total`. |
| `classe_nome` | texto | Nome legível da classe. |
| `pixels` | inteiro | Pixels de 30 m dentro da máscara do bioma. |
| `area_ha` | decimal | Área em hectares. |
| `area_km2` | decimal | Área em km². |

`Brasil (total)` é a **soma dos seis biomas**, não a contagem nacional. A
diferença para `areas_lavouras_2020_2025.csv` corresponde aos poucos pixels
de lavoura fora da máscara de biomas e está documentada em
`residuos_cobertura_biomas.json` (por exemplo, 311 pixels em 2025).

### `classes_col11.csv`

**33 linhas**. Dicionário dos **33 códigos de origem** da Coleção 11
observados nos rasters.

| Coluna | Tipo | Descrição |
|---|---|---|
| `codigo` | inteiro | Código da classe no MapBiomas. |
| `nome` | texto | Nome da classe (sem acentos, como na legenda de origem). |
| `hex` | texto | Cor da classe em hexadecimal. |
| `status` | texto | `oficial` ou `observado_pendente_mapeamento`. |
| `fora_legenda_classica` | texto | `sim` para códigos acima de 75, ausentes da legenda clássica. |
| `cor_oficial` | texto | `sim` se a cor vem da paleta oficial da Coleção 11. |
| `observado_nos_rasteres` | texto | `sim` se o código aparece nos mosaicos processados. |
| `observacao` | texto | Nota sobre códigos pendentes (ex.: classe 91). |

O arquivo lista os **33 códigos de origem** observados nos rasters. As
classes **19** (Lavoura Temporária) e **36** (Lavoura Perene) usadas nas áreas
**não estão neste dicionário**: são classes sintéticas, criadas pela
reclassificação que agrupa nove códigos de lavoura da legenda original
(20, 39, 40, 41 e 62 → 19; 35, 46, 47 e 48 → 36). As demais linhas servem de
contexto e auditoria da legenda.

### Arquivos auxiliares (JSON)

#### `contagens_fase2_4326.json`

Contagens de controle feitas nos rasters ainda em `EPSG:4326`, por ano:

| Campo | Descrição |
|---|---|
| `ano` | Ano de referência. |
| `total_px` | Total de pixels da grade. |
| `nodata_px` | Pixels NoData. |
| `px19`, `px36` | Pixels das classes 19 e 36 após a reclassificação. |
| `pct_nodata` | Percentual de NoData na grade. |
| `soma_9_codigos_fonte` | Soma dos nove códigos de lavoura da legenda original; equivale a `px19 + px36`. |
| `segundos` | Tempo de processamento da etapa, em segundos. |

Essas contagens são anteriores à reprojeção e podem diferir levemente das
contagens Albers usadas nas áreas.

#### `contagens_fase4_albers.json`

Contagens nacionais na grade Albers, por ano:

| Campo | Descrição |
|---|---|
| `"19"`, `"36"` | Pixels de cada classe. São a base de `areas_lavouras_2020_2025.csv`. |
| `mtime_ns` | Mtime do raster Albers de origem, em nanossegundos desde a época Unix. |
| `size` | Tamanho do raster Albers de origem, em bytes. |

`mtime_ns` e `size` são controles de build, não variáveis científicas. Os
rasters não são publicados neste repositório.

#### `contagens_por_bioma.json`

Objeto por ano com uma lista de **7 pares** `[pixels_classe_19,
pixels_classe_36]`. A posição na lista segue os códigos da máscara:

| Índice | Recorte |
|---|---|
| 0 | Fora dos biomas |
| 1 | Amazônia |
| 2 | Cerrado |
| 3 | Caatinga |
| 4 | Mata Atlântica |
| 5 | Pantanal |
| 6 | Pampa |

#### `areas_biomas_mascara.json`

Área da máscara de biomas na grade Albers:

| Campo | Descrição |
|---|---|
| `pixels_total_grade` | Total de pixels da grade. |
| `pixels_nao_bioma` | Pixels fora dos seis biomas. |
| `biomas` | Objeto com chaves `"1"` a `"6"`. Cada item traz `bioma`, `pixels`, `area_km2`, `oficial_km2` e `delta_pct`. |

`delta_pct` compara a área rasterizada com a área oficial aproximada do IBGE:
`100 × (area_km2 − oficial_km2) / oficial_km2`. A diferença esperada vem da
generalização das bordas dos shapefiles, não das lavouras.

No **Pampa**, a máscara segue o shapefile de origem (~193.947 km²) e difere
cerca de **9,9%** da constante oficial do IBGE (176.496 km²;
`delta_pct` = 9,888%). A diferença é da borda do shapefile de origem, **não**
de erro de rasterização ou de cópia.

#### `residuos_cobertura_biomas.json`

Checagem de cobertura, por ano e classe (`"19"` e `"36"`):

| Campo | Descrição |
|---|---|
| `nacional` | Contagem nacional na grade Albers. |
| `soma_biomas` | Soma das contagens dos seis biomas. |
| `fora_biomas_px` | Pixels de lavoura fora da máscara de biomas. |
| `residuo_px` | `nacional − soma_biomas` (igual a `fora_biomas_px` nos dados atuais). |
| `residuo_pct` | Resíduo em percentual: `100 × residuo_px / nacional`. |

Os resíduos são inferiores a 0,0001% e **não são ajustados** nos CSVs.

#### `overviews_info.json`

Metadados das pirâmides internas dos rasters reclassificados (que não são
publicados aqui):

| Campo | Descrição |
|---|---|
| `niveis_solicitados` | Níveis de overview solicitados: 2, 4, 8, 16, 32, 64 e 128. |
| `resampling` | Método de reamostragem (`nearest`). |
| `compressao` | Compressão usada (`DEFLATE`). |
| `arquivos` | Objeto por nome de arquivo com `niveis`, `menor_overview_px`, `shape_base`, `crs` e `unique_amostra_overview2`. |

Em `menor_overview_px`, a ordem é **[largura, altura]**; em `shape_base`, a
ordem é **[altura, largura]** da grade base. `unique_amostra_overview2` lista
as classes encontradas em uma amostra do overview de nível 2.

## Atualização

Este repositório é um snapshot curado dos dados publicados. Atualizações
são feitas manualmente pelo mantenedor: copiar os CSVs/JSONs para
`docs/dados/` e atualizar as tabelas HTML e os totais de `docs/index.html`.

Para publicar no GitHub Pages, aponte a fonte para o branch `main` e a pasta
`/docs` (o `.nojekyll` já está incluído). O site também funciona aberto
diretamente do disco, pois todos os caminhos são relativos.

## Licença e atribuição

- **Dados derivados** (`docs/dados/`): [CC BY-SA
  4.0](https://creativecommons.org/licenses/by-sa/4.0/deed.pt-br). Consulte
  `LICENSE`.
- **MapBiomas Coleção 11**: dados de origem conforme os termos da Coleção 11
  do MapBiomas (consultar <https://mapbiomas.org>); esta obra derivada é
  CC BY-SA 4.0. Atribuição: “MapBiomas — Coleção 11 do Mapeamento Anual de
  Cobertura e Uso da Terra do Brasil”.
- **Limites de bioma**: limites utilizados no pipeline; consulte a
  documentação do projeto para a fonte original.
- **SIDRA/PAM (IBGE)**: dados públicos. Atribuição: “IBGE — Produção Agrícola
  Municipal (PAM), SIDRA tabela 5457”. Site:
  <https://sidra.ibge.gov.br/tabela/5457>.

Ao reutilizar, cite também a fonte original de cada conjunto e indique as
transformações feitas (reclassificação das lavouras, agregação por bioma e
recorte temporal).

## Citação

O arquivo `CITATION.cff` está pronto para o GitHub. Versão BibTeX:

```bibtex
@dataset{santos2026dados,
  author       = {Santos, Priscilla Azevedo dos and Adami, Marcos and Silva, Igor Leite da},
  title        = {Dados processados — MapBiomas Coleção 11 (2020–2025) e SIDRA/PAM (2000–2024, em preparação)},
  year         = {2026},
  publisher    = {GitHub},
  url          = {https://github.com/Priscasantos/sbsr-2027-results},
  license      = {CC-BY-SA-4.0},
  note         = {Dados tabulares derivados do MapBiomas Coleção 11; SIDRA/PAM em preparação}
}
```

## Contato

Priscilla Azevedo dos Santos — GitHub: [@Priscasantos](https://github.com/Priscasantos)
