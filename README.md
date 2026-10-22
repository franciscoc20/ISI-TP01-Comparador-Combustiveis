# TP01 — Integração e Comparação de Preços de Combustíveis em Portugal

**Unidade curricular:** Integração de Sistemas de Informação (ISI) — Trabalho Prático 01  
**Curso:** Licenciatura em Engenharia de Sistemas Informáticos  
**Instituição:** Universidade Politécnica do Cávado e do Ave (UPCA)  
**Ano letivo:** 2026/2027  
**Docente:** Óscar Ribeiro

## Autores e divisão do trabalho

- **Francisco Alberto Martins Campinho — n.º 31079:** implementação em **Pentaho Data Integration (Kettle)**, Job de execução e apresentação dos resultados em **Node-RED**.
- **Miguel Marques — n.º 33221:** implementação do workflow em **KNIME Analytics Platform**.

O grupo apresenta **duas implementações independentes sobre o mesmo tema**. O Pentaho e o KNIME têm fontes, operações e resultados próprios; não existe comunicação automática entre os dois workflows. A submissão conjunta é realizada por Francisco Alberto Martins Campinho.

## Estrutura da entrega

```text
tp01-31079-33221/
├── README.md
├── doc/
│   └── 31079-33221_doc.pdf
├── dataintegration/
│   ├── pentaho/
│   │   ├── 01_gasoleo_simples.ktr
│   │   ├── 03_gasolina_95_simples.ktr
│   │   ├── 04_integrar_combustiveis.ktr
│   │   ├── 05_validar_combustiveis.ktr
│   │   ├── 06_identificar_duplicados.ktr
│   │   ├── 07_geoapi_municipios.ktr
│   │   ├── 08_cruzar_combustiveis_municipios.ktr
│   │   ├── 09_estatisticas_combustiveis.ktr
│   │   └── 10_job_principal_etl.kjb
│   └── knime/
│       └── ETL_Combustiveis.knwf
├── data/
│   ├── input/
│   │   ├── pentaho/                  # Os dois CSV originais da DGEG
│   │   └── knime/
│   │       ├── Postos.csv
│   │       ├── postos_abastecimento.json
│   │       └── catalogo_combustiveis.xml
│   └── output/
│       ├── pentaho/                  # CSV intermédios, rejeições e estatísticas finais CSV/XML
│       └── knime/
│           └── resultadofinal.csv
└── src/
    └── node_red_combustiveis.json
```

As linhas com `#` no esquema são descrições das pastas, não nomes de ficheiros. **Manter os nomes originais dos dois CSV do Pentaho e dos resultados gerados.** Os ficheiros do KNIME acima têm os nomes confirmados no ZIP recebido do Miguel. O nome real da saída é `resultadofinal.csv` (sem sublinhado).

## Ferramentas necessárias

- **Pentaho Data Integration / Spoon** (desenvolvido com PDI CE 10.2) e Java compatível; no desenvolvimento foi utilizado Java 21.
- **KNIME Analytics Platform**, com os nós/extensões requeridos pelo workflow importado.
- **Node.js e Node-RED** (no desenvolvimento: Node.js 24 e Node-RED 5.0.8).
- Pacote **`@flowfuse/node-red-dashboard`** para os elementos `ui-table` e `ui-chart` do Node-RED.
- Ligação à Internet para a consulta à GEO API PT realizada pelo Pentaho.

## Executar a parte Pentaho Kettle — Francisco

1. Abrir o **Spoon**.
2. Confirmar que os dois CSV originais da DGEG estão em `data/input/pentaho/`.
3. Abrir os ficheiros `.ktr` em `dataintegration/pentaho/` e **atualizar os caminhos de leitura e escrita** para a localização atual dos ficheiros. O projeto foi criado com caminhos absolutos do computador de desenvolvimento.
4. Abrir `dataintegration/pentaho/10_job_principal_etl.kjb` e, nas entradas do Job, confirmar/atualizar a localização de cada uma das oito transformações `.ktr`.
5. Confirmar que os passos de exportação escrevem em `data/output/pentaho/` e que esta pasta existe.
6. Executar o Job através de **Run (F9)**. A transformação `07_geoapi_municipios` consulta a GEO API PT e necessita de acesso à rede.
7. Verificar no histórico que as oito transformações terminaram com sucesso e abrir os resultados em `data/output/pentaho/`.

O processo lê Gasóleo simples e Gasolina simples 95, converte os preços, valida registos, identifica duplicados, consulta os municípios e códigos INE da GEO API PT, associa os dados através de `Stream Lookup` e calcula estatísticas por `Distrito`, `Municipio`, `Combustivel` e `Codigo_INE`.

**Resultados da execução documentada:** 6.009 observações validadas, sem linhas nas saídas de rejeição documentadas, e 555 grupos estatísticos. Os resultados finais são `estatisticas_combustiveis.csv` e `estatisticas_combustiveis.xml`, com preços médio, mínimo e máximo e a contagem de observações. `Numero_Registos` não equivale necessariamente ao número de postos distintos.

## Executar o Node-RED — Francisco

1. Abrir um terminal e iniciar o serviço:

   ```powershell
   node-red
   ```

2. Abrir `http://127.0.0.1:1880/` no navegador.
3. Instalar o módulo `@flowfuse/node-red-dashboard` pela gestão de paleta, se ainda não estiver instalado.
4. No menu do Node-RED, escolher **Import** e selecionar `src/node_red_combustiveis.json`.
5. Abrir **ambos** os nós chamados `Ler Estatisticas CSV` e alterar o caminho do ficheiro para a localização atual de `data/output/pentaho/estatisticas_combustiveis.csv`. O fluxo exportado inclui caminhos absolutos do computador original.
6. Clicar em **Deploy**. Se a tabela não carregar, ativar o nó `Atualizar Estatisticas`.
7. Consultar:
   - **Endpoint HTTP (JSON):** http://127.0.0.1:1880/combustiveis
   - **Dashboard:** http://127.0.0.1:1880/dashboard/page1

A tabela permite pesquisar os dados e o gráfico de barras mostra o **número de observações** associado aos municípios e combustíveis selecionados, não os preços médios. O Node-RED lê as estatísticas já exportadas pelo Pentaho; não atualiza automaticamente os preços da DGEG.

## Executar a parte KNIME — Miguel

1. Abrir o **KNIME Analytics Platform**.
2. Importar o workflow exportado `dataintegration/knime/ETL_Combustiveis.knwf` através da função de **importação de workflows** do KNIME.
3. Abrir os nós de leitura e confirmar os caminhos para:
   - `data/input/knime/Postos.csv` — dados tabulares dos postos e preços;
   - `data/input/knime/postos_abastecimento.json` — dados geográficos de postos;
   - `data/input/knime/catalogo_combustiveis.xml` — catálogo de códigos, nomes e unidades dos combustíveis.
4. Atualizar caminhos locais, caso estejam configurados com localizações absolutas do computador do Miguel.
5. Verificar o caminho de escrita do nó **CSV Writer** e apontá-lo para `data/output/knime/resultadofinal.csv`.
6. Executar os nós do workflow até à saída e consultar o ficheiro final num editor de CSV ou no Excel.

A implementação KNIME trata tipos e valores em falta, remove duplicados, filtra preços, normaliza marcas e coordenadas e associa as origens através de nós `Joiner`, acrescentando ainda dados do catálogo XML. O cruzamento CSV/JSON utiliza os campos preparados de latitude aproximada, marca normalizada e longitude aproximada. **Na execução documentada, o resultado final tinha 5.466 linhas.** Os resultados não devem ser comparados diretamente com os 555 grupos estatísticos do Pentaho, pois as saídas têm granularidade diferente.

## Vídeos de demonstração

- **Pentaho Kettle e Node-RED — Francisco:** https://youtu.be/Os4p-xmINcE
- **KNIME — Miguel:** https://qrto.org/lHya9e (ligação associada ao QR Code fornecido pelo Miguel; confirmar acesso ao vídeo)

Os QR Codes também se encontram no capítulo dedicado à demonstração do relatório.

## Repositório GitHub

**Repositório conjunto:** https://github.com/franciscoc20/ISI-TP01-Comparador-Combustiveis

O repositório deve conter **as duas implementações** e os respetivos ficheiros de entrada, saída e documentação. A estrutura das pastas permite identificar os ficheiros de cada elemento.

## Fontes

- Preços dos combustíveis: [Direção-Geral de Energia e Geologia — Preços dos Combustíveis Online](https://precoscombustiveis.dgeg.gov.pt/).
- Municípios e códigos INE utilizados no Pentaho: [GEO API PT](https://geoapi.pt/).
- As três fontes de entrada da implementação KNIME estão incluídas em `data/input/knime/`.
