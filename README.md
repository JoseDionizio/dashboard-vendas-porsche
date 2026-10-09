# Dashboard de Vendas Porsche

Dashboard interativa em HTML, em arquivo único, construída a partir de uma planilha com 100 vendas da Porsche nos Estados Unidos. Projeto do desafio da DIO "transformar uma planilha de vendas em uma dashboard publicada na web".

**Dashboard publicada:** https://SEU-USUARIO.github.io/dashboard-vendas-porsche/

![Visão geral da dashboard](docs/print-geral.png)

---

## Perguntas de negócio

A dashboard foi organizada em torno de quatro perguntas. Cada uma tem um gráfico e uma frase de resposta, que é recalculada sempre que um filtro muda.

| # | Pergunta | Gráfico | Por que escolhi |
|---|----------|---------|-----------------|
| 1 | **Qual família de modelos traz mais receita, e isso bate com o volume?** | Barras de receita e de unidades lado a lado, por família | Volume e receita contam histórias diferentes. O Macan vende quase tanto quanto o Cayenne (17 contra 18 unidades), mas rende bem menos (US$ 1,3 mi contra US$ 2,1 mi), porque o ticket médio dele é de US$ 78,7 mil. Isso ajuda a decidir onde investir em estoque. |
| 2 | **Onde está concentrada a receita?** | Mapa em grade dos EUA, que alterna entre receita, vendas e ticket médio, com o top 5 de estados | Texas e Califórnia somam 31% da receita. Nevada tem só 4 vendas, mas o maior ticket médio (US$ 160,9 mil). Isso mostra onde focar ações comerciais. |
| 3 | **A forma de pagamento muda com o valor do carro?** | Barras de ticket médio por método de pagamento, com o número de vendas | Criptomoeda e transferência internacional (wire) têm ticket médio entre US$ 150 e 180 mil. Débito e transferência bancária comum ficam abaixo de US$ 91 mil. Isso ajuda a decidir que condições de pagamento oferecer em cada faixa de preço. |
| 4 | **Os elétricos já pesam no faturamento?** | Barras 100% empilhadas comparando a participação nas vendas e na receita, por combustível | Os elétricos (Taycan e Macan Electric) são 20% das vendas e 21% da receita, com ticket médio acima dos carros a combustão. É um indicador de tendência do portfólio. |

### O que ficou de fora, e por quê

Antes de escolher as perguntas, levantei uma lista ampla de KPIs comuns em dashboards automotivas e conferi cada um contra as colunas da base. Ficaram de fora os que **não têm dado na planilha**: meta de vendas, margem, conversão de leads, ciclo de venda, cor, motorização, loja ou concessionária, vendedor e market share.

A comparação com o período anterior (YoY/MoM) também saiu, por dois motivos: 24 vendas não têm data válida, e a base tem vendas datadas até 2027. Pedir esses indicadores à IA sem os dados faz ela inventar números.

---

## Funcionalidades

- **4 KPIs no topo:** vendas, receita, ticket médio (com a faixa de preço) e participação dos elétricos na receita.
- **7 filtros:** ano da venda (com a opção "Sem data"), região, estado, família, modelo, método de pagamento e combustível.
- **Filtros encadeados:** escolher uma região limita a lista de estados, e escolher uma família limita a lista de modelos.
- **Filtro por clique:** clicar numa barra, num estado do mapa ou num segmento de combustível filtra a dashboard inteira. Clicar de novo remove o filtro.
- **Respostas dinâmicas:** a frase de cada pergunta é reescrita conforme o recorte.
- **Tabela de detalhe:** mostra receita, vendas e ticket médio por família, e abre os modelos de cada família ao clicar.
- **Tema claro e escuro:** segue a configuração do sistema.
- **Layout responsivo:** funciona no celular.

### Exemplo de filtro aplicado

Filtro **Combustível = Elétrico**: o painel passa a mostrar 20 vendas e US$ 2,7 mi. O Taycan concentra 87% dessa receita, e Califórnia e Texas continuam liderando.

![Dashboard com filtro de combustível elétrico](docs/print-filtro-eletrico.png)

---

## Tratamento da base

A planilha original tem cada campo em duas versões, o dado cru e o sanitizado. Antes de enviar a base para a IA, fiz o tratamento abaixo. O resultado está em [`dados/base_porsche_tratada.xlsx`](dados/base_porsche_tratada.xlsx), com as abas `dicionario` e `tratamento`, e em [`dados/base_porsche_tratada.csv`](dados/base_porsche_tratada.csv).

1. **Só colunas sanitizadas.** Descartei as 9 colunas cruas e renomeei as restantes em português.
2. **Dados pessoais removidos.** Saíram `customer_name` e `salesperson`. A base vai embutida no HTML público, então nenhum nome de pessoa pode ficar nela.
3. **Tipos convertidos.** Preço, ano do modelo e quilometragem estavam como texto e viraram número. A data virou data.
4. **Datas INVALID.** 24 das 100 vendas têm data impossível na origem, como 30/02 ou 31/04. **Mantive essas vendas**, porque modelo, preço e local estão corretos e excluí-las apagaria um quarto da receita. A data ficou vazia e criei a coluna `data_valida` (Sim/Não), que aparece no filtro como "Sem data".
5. **Colunas derivadas:**
   - `familia`: agrupa as 40 versões em 6 linhas (911, Cayenne, Macan, Taycan, Panamera, 718).
   - `combustivel`: Elétrico (Taycan e Macan Electric), Híbrido (E-Hybrid) ou Combustão.
   - `regiao`: as 4 regiões do Censo dos EUA (Nordeste, Meio-Oeste, Sul e Oeste), a partir do estado.
   - `status_grupo`: reúne 10 status de entrega parecidos em 5 grupos.
6. **Colado como valores.** Nenhuma fórmula no arquivo final.

**Observações sobre a base:**
- As datas válidas vão de 14/03/2024 a 01/10/2027. Parte das vendas está no futuro, o que indica uma base fictícia.
- Três datas INVALID parecem ter dia e mês trocados (`2024/15/07`, `2025/18/11` e `2027/15/04`). Não corrigi essas datas. Fica como próxima evolução: escrever uma segunda regra para recuperá-las.

---

## Ferramenta utilizada

Usei o **Claude (Anthropic)** nas duas etapas: no tratamento da base e na geração da dashboard em HTML.

O desafio sugere o ChatGPT com Canvas e um agente com skill própria. Não consegui gerar a dashboard no Canvas porque não tenho a versão paga do ChatGPT, e os agentes também exigem assinatura. Por isso fiz o mesmo caminho com outra IA. O processo continua o mesmo do desafio: preparar a base, escrever o prompt com as perguntas de negócio, refinar em linguagem natural e publicar no GitHub Pages.

---

## O prompt

O trabalho foi feito em duas etapas, cada uma com o seu prompt. Primeiro, a normalização dos dados. Depois, a geração da dashboard a partir da base já tratada. Separar as duas coisas fez diferença: a IA que desenha a dashboard recebe um dado limpo e não precisa adivinhar formatos.

### Etapa 1: normalização dos dados

```text
Anexei uma planilha com 100 vendas da Porsche. Cada campo aparece em duas
versões: o dado cru e a coluna "Sanitized". Prepare a base para uma dashboard:

1. Mantenha só as colunas sanitizadas e renomeie-as em português (snake_case).
2. Remova dados pessoais (nome do cliente e vendedor), porque a base vai
   embutida num HTML público.
3. Converta preço, ano do modelo e quilometragem para número, e a data para data.
4. Não exclua as vendas com data "INVALID": deixe a data vazia e crie a coluna
   data_valida (Sim/Não).
5. Crie as colunas derivadas: familia (primeira palavra do modelo), combustivel
   (Elétrico para Taycan e Macan Electric, Híbrido para E-Hybrid, Combustão para
   o resto), regiao (as 4 regiões do Censo dos EUA a partir do estado) e
   status_grupo (agrupando os status de entrega parecidos).
6. Entregue tudo como valores, sem fórmulas, em .xlsx e .csv, com uma aba de
   dicionário de dados e outra com o registro do tratamento.
Antes de tratar, aponte problemas na base que eu deva conhecer.
```

### Etapa 2: geração da dashboard

```text
Contexto: anexei uma base com 100 vendas da Porsche nos EUA (CSV). Colunas:
id_venda, data_venda, data_valida (Sim/Não), ano_venda, modelo, familia,
combustivel (Combustão/Elétrico/Híbrido), ano_modelo, preco_usd, quilometragem_mi,
metodo_pagamento, cidade, estado (sigla), regiao, status_entrega, status_grupo.
24 vendas não têm data: não as exclua, trate como "Sem data".

Crie uma dashboard em HTML, em um único arquivo e com os dados embutidos, que
responda a quatro perguntas de negócio:
1. Qual família traz mais receita, e isso bate com o volume?
   → barras horizontais de receita e de unidades lado a lado, ordenadas por receita.
2. Onde está concentrada a receita?
   → mapa em grade dos estados dos EUA, com escala de uma só cor, alternando
     entre receita, vendas e ticket médio, mais o top 5 de estados.
3. A forma de pagamento muda com o valor do carro?
   → barras de ticket médio por método de pagamento, com o número de vendas.
4. Os elétricos já pesam no faturamento?
   → duas barras 100% empilhadas (vendas e receita) por combustível.
Cada pergunta deve ter uma frase de resposta que se atualiza com os filtros.

KPIs no topo: vendas, receita, ticket médio e % da receita vinda de elétricos.
Filtros: ano da venda (com "Sem data"), região, estado, família, modelo,
pagamento e combustível, com um botão "Limpar filtros". Estado depende da região
e modelo depende da família. Clicar numa barra ou num estado também filtra.
No fim, uma tabela por família com receita, vendas, ticket médio e % da receita,
que abre os modelos ao clicar.

Visual: estilo ficha técnica de showroom, com fundo prata claro, títulos em
caixa alta expandida (Archivo), texto em Public Sans e números em IBM Plex Mono.
Um único azul para os dados. Tema claro e escuro, e funcionando no celular.
Valores em US$, no formato "US$ 12,8 mi" e "US$ 128,3 mil".
Não invente métricas que a base não tem (meta, margem, leads, lojas).
```

### O que mudou até a versão final

Os ajustes abaixo aconteceram na etapa 2, ao refinar as perguntas e o visual da dashboard.

| # | Mudança | Motivo |
|---|---------|--------|
| 1 | Comecei com uma lista ampla de KPIs (meta, margem, conversão de leads, ciclo de venda, cor, lojas, market share) | Era a estrutura "ideal" de uma dashboard automotiva |
| 2 | Cortei tudo o que a base não tem e troquei o filtro "canal" por **método de pagamento** | Sem os dados, a IA preencheria esses indicadores com números inventados |
| 3 | Troquei o ranking dos 40 modelos por um ranking de **6 famílias** | Cada modelo tem de 1 a 4 vendas, então um "bottom 10" seria só empate |
| 4 | Troquei "vendas por cidade" por estado e região | São 79 cidades para 100 vendas, quase uma venda por cidade |
| 5 | Acrescentei a 4ª pergunta (elétricos) e criei a coluna `combustivel` | Mostra uma tendência de portfólio que as outras três perguntas não cobrem |
| 6 | Ajustes visuais: rótulos que vazavam da barra, contraste do mapa no tema escuro e rolagem lateral no celular | Encontrados ao testar a dashboard nos temas claro e escuro e na largura de celular |

---

## Estrutura do repositório

```
dashboard-vendas-porsche/
├── index.html                      # a dashboard (arquivo único, dados embutidos)
├── README.md
├── dados/
│   ├── base_porsche_tratada.xlsx   # base tratada + dicionário + registro do tratamento
│   └── base_porsche_tratada.csv    # mesma base, usada na dashboard
└── docs/
    ├── print-geral.png
    └── print-filtro-eletrico.png
```

## Como rodar localmente

Não precisa instalar nada. Baixe o repositório e abra o `index.html` no navegador.

## Próximos passos

- Escrever uma regra para recuperar as datas com dia e mês trocados.
- Quando tiver acesso ao ChatGPT pago, gerar a mesma dashboard no Canvas e com um agente, e comparar com esta versão.
- Montar um agente com duas skills: uma para tratar o dado e outra para desenhar a dashboard.
