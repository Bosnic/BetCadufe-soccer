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

## Executando o Código
- Python: Execute o script para coletar dados e armazená-los no Google BigQuery. (python champions.ipynb)
- Power BI: Conecte-se ao Google BigQuery e carregue as views configuradas para visualizar os dados no dashboard.

## Visualização no Power BI
O dashboard desenvolvido no Power BI exibe dados interativos de jogos e estatísticas, facilitando a análise de métricas como desempenho de equipes e jogadores.

## Licença
Este projeto é licenciado sob a Licença Apache 2.0.
