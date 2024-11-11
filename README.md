# BetCadufe-soccer ⚽📊

Projeto de integração de dados de futebol via API, armazenamento no Google Cloud e visualização no Power BI.

## Descrição
O **BetCadufe-soccer** é um projeto focado na coleta, transformação e visualização de dados de futebol. Através da API da [API Sports](https://api-sports.io), este código em Python realiza a coleta de dados de jogos e estatísticas de futebol, organiza-os em tabelas no Google Cloud e, em seguida, os disponibiliza via views para visualização no Power BI. O objetivo é criar um dashboard interativo com insights e métricas relevantes para análise de dados esportivos.

## Funcionalidades
- **Integração com API**: Coleta de dados de futebol usando a API Sports.
- **Armazenamento no Google Cloud**: Armazena os dados em tabelas no Google BigQuery para fácil manipulação e consulta.
- **Visualização no Power BI**: Views no Google Cloud são utilizadas para carregar dados no Power BI, onde um dashboard interativo exibe as principais métricas.

## Como Executar

### Pré-requisitos
- Conta ativa e chave de API no [API Sports](https://api-sports.io).
- Acesso ao Google Cloud e Google BigQuery.
- Instalação do Python e bibliotecas requeridas (veja abaixo).

### Configuração do Ambiente
1. Clone este repositório:
   ```bash
   git clone https://github.com/Bosnic/BetCadufe-soccer.git

2. Instale as dependências necessárias:
   pip install -r requirements.txt

3. Configure as credenciais do Google Cloud e da API Sports no código Python.
   pip install -r requirements.txt

## Criação de Views no Google Cloud\
As seguintes views foram criadas no Google Cloud para facilitar a consulta e carregar os dados no Power BI:

### 1. **vw_estadios**

Esta view é responsável por trazer informações sobre os estádios onde as partidas ocorreram.

```sql
SELECT DISTINCT
    estadio_id,
    estadio_nome,
    estadio_nomeCidade
FROM (
    SELECT 
        SAFE_CAST(fixture__venue__id AS STRING) as estadio_id, 
        fixture__venue__name as estadio_nome, 
        fixture__venue__city as estadio_nomeCidade
    FROM
        `soccer.past_fixtures`
)
WHERE
    estadio_id IS NOT NULL
```

### 2. **vw_estastisticas**

Esta view agrega as estatísticas de cada jogo, incluindo chutes, faltas, escanteios, etc.
```sql
WITH stats_aggregated AS (
  SELECT
    SAFE_CAST(fixture AS STRING) as jogo_id,
    SAFE_CAST(team__id AS STRING) as time_id,
    team__logo as team_logo,
    team__name as team_name,
    ARRAY_AGG(STRUCT(statistics.value, statistics.type)) as stats_array
  FROM
    `soccer.fixturesStatistics`
  CROSS JOIN
    UNNEST(statistics) as statistics
  GROUP BY
    jogo_id,
    time_id,
    team_logo,
    team_name
)

SELECT
  jogo_id,
  time_id,
  team_logo,
  team_name,
  MAX(SAFE_CAST(CASE WHEN stat.type = 'Shots on Goal' THEN stat.value ELSE NULL END AS INT64)) AS chutes_aGol,
  MAX(SAFE_CAST(CASE WHEN stat.type = 'Shots off Goal' THEN stat.value ELSE NULL END AS INT64)) AS chutes_foraDoGol,
  MAX(SAFE_CAST(CASE WHEN stat.type = 'Total Shots' THEN stat.value ELSE NULL END AS INT64)) AS chutes_total,
  MAX(SAFE_CAST(CASE WHEN stat.type = 'Blocked Shots' THEN stat.value ELSE NULL END AS INT64)) AS chutes_bloqueados,
  MAX(SAFE_CAST(CASE WHEN stat.type = 'Shots insidebox' THEN stat.value ELSE NULL END AS INT64)) AS chutes_dentroDaArea,
  MAX(SAFE_CAST(CASE WHEN stat.type = 'Shots outsidebox' THEN stat.value ELSE NULL END AS INT64)) AS chutes_foraDaArea,
  MAX(SAFE_CAST(CASE WHEN stat.type = 'Fouls' THEN stat.value ELSE NULL END AS INT64)) AS faltas,
  MAX(SAFE_CAST(CASE WHEN stat.type = 'Corner Kicks' THEN stat.value ELSE NULL END AS INT64)) AS escanteios,
  MAX(SAFE_CAST(CASE WHEN stat.type = 'Offsides' THEN stat.value ELSE NULL END AS INT64)) AS impedimentos,
  MAX(SAFE_CAST(CASE WHEN stat.type = 'Ball Possession' THEN REPLACE(stat.value, '%', '') ELSE NULL END AS FLOAT64)) AS posseDeBola,  -- Assuming percentage is stored as string with '%'
  MAX(SAFE_CAST(CASE WHEN stat.type = 'Yellow Cards' THEN stat.value ELSE NULL END AS INT64)) AS cartoes_amarelos,
  MAX(SAFE_CAST(CASE WHEN stat.type = 'Red Cards' THEN stat.value ELSE NULL END AS INT64)) AS cartoes_vermelhos,
  MAX(SAFE_CAST(CASE WHEN stat.type = 'Goalkeeper Saves' THEN stat.value ELSE NULL END AS INT64)) AS defesasDoGoleiro,
  MAX(SAFE_CAST(CASE WHEN stat.type = 'Total passes' THEN stat.value ELSE NULL END AS INT64)) AS passes_total,
  MAX(SAFE_CAST(CASE WHEN stat.type = 'Passes accurate' THEN stat.value ELSE NULL END AS INT64)) AS passes_finalizados,
  MAX(SAFE_CAST(CASE WHEN stat.type = 'Passes %' THEN REPLACE(stat.value, '%', '') ELSE NULL END AS FLOAT64)) AS passes_precisao,  -- Assuming percentage is stored as string with '%'
  MAX(SAFE_CAST(CASE WHEN stat.type = 'expected_goals' THEN stat.value ELSE NULL END AS FLOAT64)) AS gols_esperados
FROM
  stats_aggregated,
  UNNEST(stats_array) as stat
GROUP BY
  jogo_id,
  time_id,
  team_logo,
  team_name
```

### 3. **vw_estastisticas_partidas**

Esta view combina informações das partidas e as respectivas estatísticas dos times.

```sql
SELECT
    f.jogo_id,
    f.juiz_nome,
    DATE(f.data) AS data,
    TIME(f.data) AS hora,
    f.status_id,
    f.liga_id,
    SAFE_CAST( f.temporada AS STRING) AS temporada,
    f.rodada,
    f.time_id,
    f.time_nome,
    f.time_logoUrl,
    SAFE_CAST(f.time_vitoria AS STRING) as time_vitoria,
    f.gols,
    f.gols_primeiroTempo,
    f.gols_segundoTempo,
    f.gols_prorrogacao,
    f.gols_penalti,
    f.estadio_id,
    f.casaFora,
    s.chutes_aGol,
    s.chutes_foraDoGol,
    s.chutes_total,
    s.chutes_bloqueados,
    s.chutes_dentroDaArea,
    s.chutes_foraDaArea,
    s.faltas,
    s.escanteios,
    s.impedimentos,
    s.posseDeBola / 100 as posseDeBola,
    s.cartoes_amarelos,
    s.cartoes_vermelhos,
    s.defesasDoGoleiro,
    s.passes_total,
    s.passes_finalizados,
    s.passes_precisao,
    s.gols_esperados
FROM
    `soccer_view.vw_partidas_todas` f
LEFT JOIN
    `soccer_view.vw_estatisticas` s
ON
    f.jogo_id = s.jogo_id AND f.time_id = s.time_id;
```

### 4. **vw_partidas_casa**

Esta view traz informações sobre os jogos jogados em casa.

```sql
SELECT 
	SAFE_CAST( fixture__id AS STRING ) as jogo_id,
	fixture__referee as juiz_nome,
	fixture__date as data,
	fixture__status__short as status_id,
	SAFE_CAST( league__id AS STRING ) as liga_id,
	league__season as temporada,
	league__round as rodada,
	SAFE_CAST(teams__home__id AS STRING ) AS time_id,
	teams__home__name AS time_nome,
	teams__home__logo AS time_logoUrl,
	teams__home__winner AS time_vitoria,
	goals__home AS gols,
	score__halftime__home AS gols_primeiroTempo,
	score__fulltime__home AS gols_segundoTempo,
	score__extratime__home AS gols_prorrogacao,
	score__penalty__home AS gols_penalti,
	SAFE_CAST(fixture__venue__id AS STRING) AS estadio_id,
	'Casa' AS casaFora
FROM `soccer.past_fixtures`
```

### 5. **vw_partidas_fora**

Esta view traz informações sobre os jogos jogados fora de casa.

```sql
SELECT 
	SAFE_CAST( fixture__id AS STRING ) as jogo_id,
	fixture__referee as juiz_nome,
	fixture__date as data,
	fixture__status__short as status_id,
	SAFE_CAST( league__id AS STRING ) as liga_id,
	league__season as temporada,
	league__round as rodada,
	SAFE_CAST(teams__away__id AS STRING ) AS time_id,
	teams__away__name AS time_nome,
	teams__away__logo AS time_logoUrl,
	teams__away__winner AS time_vitoria,
	goals__away AS gols,
	score__halftime__away AS gols_primeiroTempo,
	score__fulltime__away AS gols_segundoTempo,
	score__extratime__away AS gols_prorrogacao,
	score__penalty__away AS gols_penalti,
	SAFE_CAST(fixture__venue__id AS STRING) AS estadio_id,
	'Fora' AS casaFora
FROM `soccer.past_fixtures`
```

### 6. **vw_partidas_todas**

Esta view traz as informações das partidas jogadas fora e dentro de casa.

```sql
SELECT 
    *
FROM
    `soccer_view.vw_partidas_casa`

UNION ALL

SELECT 
    *
FROM
    `soccer_view.vw_partidas_fora`
```

### 7. **vw_status**

Esta view traz os status das partidas, como "Em andamento", "Finalizado", etc.

```sql
SELECT DISTINCT
	fixture__status__short as status_id, 
	fixture__status__long as status_nome
FROM
	`soccer.past_fixtures`
```

### 8. **vw_times**

Esta view traz os dados dos times, incluindo nome e logo.

```sql
SELECT DISTINCT
    time_id,
    time_nome,
    time_logoUrl
FROM `soccer_view.vw_partidas_todas`
```

Essas views foram criadas para facilitar a consulta e carregar os dados no Power BI, permitindo uma análise eficiente das partidas de futebol.

## Executando o Código
- Python: Execute o script para coletar dados e armazená-los no Google BigQuery. (python champions.ipynb)
- Google Cloud: Crie as views no Google BigQuery para facilitar a consulta e o carregamento dos dados.
- Power BI: Conecte-se ao Google BigQuery e carregue as views configuradas para visualizar os dados no dashboard.

## Visualização no Power BI
O dashboard desenvolvido no Power BI exibe dados interativos de jogos e estatísticas, facilitando a análise de métricas como desempenho de equipes e jogadores.

## Licença
Este projeto é licenciado sob a Licença Apache 2.0.
