# TP01 — Sistema de Integração e Comparação de Preços de Combustíveis em Portugal

**Unidade curricular:** Integração de Sistemas de Informação (ISI) — Trabalho Prático 01  
**Curso:** Licenciatura em Engenharia de Sistemas Informáticos  
**Instituição:** Universidade Politécnica do Cávado e do Ave (UPCA)  
**Docente:** Óscar Ribeiro  
**Ano letivo:** 2026/2027  
**Grupo:** Francisco Alberto Martins Campinho (n.º 31079) e Miguel Marques (n.º 33221)  
**Entrega:** conjunta, submetida por Francisco Alberto Martins Campinho

## 1. Descrição

O trabalho aborda a integração de dados relacionados com preços de combustíveis em Portugal através de **duas implementações ETL distintas**, desenvolvidas separadamente pelos elementos do grupo e reunidas nesta entrega:

- **Pentaho Data Integration (Kettle) e Node-RED — Francisco (31079):** processamento de CSV de Gasóleo simples e Gasolina simples 95 da DGEG, associação dos municípios à GEO API PT (JSON), validação, deduplicação e cálculo de estatísticas por município e combustível. O Job executa oito transformações e produz resultados em CSV e XML. O Node-RED permite consultar o CSV final numa tabela, num gráfico e através de um endpoint HTTP.
- **KNIME — Miguel (33221):** workflow de integração de ficheiros CSV de preços, dados geográficos JSON e catálogo XML de combustíveis; inclui preparação de valores e tipos, filtragem, remoção de duplicados, correspondência entre as fontes com Joiner e exportação de uma tabela integrada para CSV. Na execução documentada, o resultado KNIME contém 5.466 linhas.

São **duas soluções independentes para o mesmo tema**. Os seus resultados têm estruturas diferentes e não se deve assumir que são equivalentes ou que existe comunicação automática entre elas.

## 2. Organização dos ficheiros

```text
tp01-31079-33221/
├── README.md
├── doc/
│   └── 31079-33221_doc.pdf
├── dataintegration/
│   ├── 01_gasoleo_simples.ktr
│   ├── 03_gasolina_95_simples.ktr
│   ├── 04_integrar_combustiveis.ktr
│   ├── 05_validar_combustiveis.ktr
│   ├── 06_identificar_duplicados.ktr
│   ├── 07_geoapi_municipios.ktr
│   ├── 08_cruzar_combustiveis_municipios.ktr
│   ├── 09_estatisticas_combustiveis.ktr
│   ├── 10_job_principal_etl.kjb
│   └── knime/
│       └── [workflow KNIME e respetivos ficheiros do projeto]
├── data/
│   ├── input/
│   │   ├── [CSV originais do Pentaho]
│   │   └── knime/  [CSV, JSON e XML utilizados no KNIME]
│   └── output/
│       ├── [CSV e XML produzidos pelo Pentaho]
│       └── knime/  [resultado_final.csv e outras saídas do KNIME]
└── src/
    └── node_red_combustiveis.json
```

**Nota sobre a estrutura:** os textos entre parênteses retos são indicações de conteúdo, **não são nomes de ficheiros a criar**. Manter os nomes verdadeiros do projeto KNIME e dos dados originais. O workflow KNIME deve ser incluído na forma exportada pelo KNIME ou com os ficheiros necessários à sua abertura. Se contiver caminhos externos, esses caminhos terão de ser adaptados.

## 3. Requisitos

- Pentaho Data Integration (PDI/Kettle CE 10.2) e Java compatível (desenvolvimento em Java 21).
- KNIME Analytics Platform, com os nós/extensões usados no workflow entregue.
- Node.js e Node-RED (desenvolvimento com Node.js 24 e Node-RED 5.0.8).
- Pacote do dashboard `@flowfuse/node-red-dashboard` no Node-RED.
- Acesso à Internet para a consulta REST à GEO API PT na implementação Pentaho.
- Permissões para ler os ficheiros de entrada e escrever os ficheiros de saída.

## 4. Executar a implementação Pentaho

1. Abrir o **Spoon**.
2. Confirmar a existência dos dois CSV originais em `data/input/`.
3. Rever os **caminhos dos ficheiros** nos passos de leitura e exportação dos `.ktr` e os caminhos das transformações referidas no `.kjb`. O projeto foi desenvolvido com caminhos locais do computador original, que podem não coincidir com a nova pasta de entrega.
4. Ajustar as saídas para `data/output/`, se necessário.
5. Abrir `dataintegration/10_job_principal_etl.kjb` e executar o Job (**Run/F9**).
6. Confirmar no histórico que as **oito transformações** terminaram com sucesso.
7. Consultar `data/output/estatisticas_combustiveis.csv` e `data/output/estatisticas_combustiveis.xml`.

**Resultados documentados do Pentaho:** 6.009 observações geograficamente validadas, zero registos rejeitados nas saídas documentadas e **555 grupos estatísticos**, exportados nos dois formatos. As estatísticas agrupam por `Distrito`, `Municipio`, `Combustivel` e `Codigo_INE`, calculando preço médio, mínimo, máximo e número de observações. `Numero_Registos` não representa necessariamente o número de postos distintos.

## 5. Executar o Node-RED

1. Iniciar o Node-RED num terminal do Windows:

   ```powershell
   node-red
   ```

2. Abrir `http://127.0.0.1:1880/`.
3. Instalar `@flowfuse/node-red-dashboard`, caso não esteja instalado.
4. Em **Menu → Import**, selecionar `src/node_red_combustiveis.json` e importar o fluxo.
5. Nos **dois** nós `Ler Estatisticas CSV`, atualizar o caminho para o ficheiro `data/output/estatisticas_combustiveis.csv` na localização atual. O JSON exportado pode conservar caminhos absolutos antigos.
6. Clicar em **Deploy**. Se necessário, clicar no nó `Atualizar Estatisticas`.
7. Abrir:
   - Dados em JSON: `http://127.0.0.1:1880/combustiveis`
   - Dashboard: `http://127.0.0.1:1880/dashboard/page1`

A tabela mostra as estatísticas e o gráfico de barras representa o **número de registos**, não o preço médio. Não há atualização automática das fontes DGEG no dashboard.

## 6. Executar a implementação KNIME

1. Abrir o **KNIME Analytics Platform**.
2. Importar/abrir o workflow KNIME entregue em `dataintegration/knime/`, utilizando a opção de importação adequada ao formato disponibilizado.
3. Conferir os caminhos para o **CSV de preços**, o **JSON de dados geográficos** e o **catálogo XML**, que devem estar incluídos em `data/input/knime/`.
4. Confirmar as configurações de leitura e a pasta de saída do **CSV Writer**; atualizar eventuais caminhos absolutos para a localização atual.
5. Executar o workflow e verificar os estados dos nós até ao CSV Writer.
6. Consultar o ficheiro `resultado_final.csv` em `data/output/knime/`, ou no caminho efetivamente configurado na entrega. O resultado documentado tem **5.466 linhas**.

O workflow KNIME trata a ligação CSV/JSON através de campos preparados de **latitude aproximada, marca normalizada e longitude aproximada** e cruza posteriormente os dados com um catálogo XML de combustíveis. Os detalhes devem ser confirmados no workflow e no relatório; não foi demonstrada execução conjunta com o Job Pentaho.

## 7. Fontes e saídas

**Pentaho:** ficheiros CSV de Gasóleo simples e Gasolina simples 95, obtidos no portal [Preços dos Combustíveis Online — DGEG](https://precoscombustiveis.dgeg.gov.pt/); referência municipal da [GEO API PT](https://geoapi.pt/). Saídas CSV e XML, incluindo ficheiros intermédios e ficheiros de rejeição (que podem estar vazios).

**KNIME:** CSV de preços, JSON de dados geográficos de postos e catálogo XML de combustíveis, conforme documentado no relatório e nos ficheiros de entrada incluídos. Saída tabular `resultado_final.csv`.

## 8. Vídeos de demonstração

- **Pentaho Kettle e Node-RED — Francisco:** https://youtu.be/Os4p-xmINcE
- **KNIME — Miguel:** QR Code e ligação de acesso incluídos no capítulo 9 do relatório integrado: https://qrto.org/lHya9e (confirmar que o endereço abre o vídeo correto).

## 9. Repositório GitHub

**Repositório conjunto:** https://github.com/franciscoc20/ISI-TP01-Comparador-Combustiveis

O repositório deverá conter os **ficheiros dos dois elementos**, e não apenas o Pentaho/Node-RED. Depois de acrescentar o workflow e os dados KNIME, atualizar o README no GitHub com esta versão.

## 10. Submissão

Entregar **um único ficheiro `31079-33221.ZIP`**, que contenha a pasta raiz `tp01-31079-33221/` com o README, relatório, fontes e dados das duas implementações. O PDF integrado deve estar em `doc/31079-33221_doc.pdf`. A submissão será feita por Francisco Alberto Martins Campinho, em nome do grupo, até **22 de outubro de 2026, às 23h59**.

Antes de submeter, verificar que os ficheiros KNIME estão presentes e que os links dos vídeos e do repositório funcionam.
