# Case Técnico | Analista de Dataviz Sênior | Dataside

Este repositório contém a documentação técnica e funcional do dashboard desenvolvido em Power BI como parte do processo seletivo para a oportunidade de **Analista de Dataviz Sênior**.

## Objetivo

Construir um relatório analítico em Power BI a partir da base fornecida no teste técnico, respeitando a restrição de **não tratar o arquivo fora do Power BI**, realizando toda a ingestão, normalização, modelagem e construção visual diretamente no ecossistema da ferramenta.

## Link de publicação

Acesse o dashboard publicado no Power BI Service pelo link abaixo:

[Visualizar dashboard publicado](https://app.powerbi.com/view?r=eyJrIjoiZGExMjg5NWYtZGY3NC00NmU5LTg0YzUtMzQxNTU2YzQ4MjVjIiwidCI6IjE1M2U3N2E0LWMyOWQtNGYyZS04ODU3LWU0MDU5M2YxNjkzMCJ9)

---

## Visão geral da solução

O projeto foi estruturado em três camadas principais:

- **Staging (`stg_Vendas`)**: ingestão e normalização da planilha bruta
- **Fato (`fVendas`)**: tabela transacional enxuta com métricas e chaves
- **Dimensões (`dClientes`, `dVendedores`, `dProdutos`, `dCalendario`)**: tabelas auxiliares para composição do modelo estrela

O modelo foi desenhado para suportar análise de:

- receita
- margem
- mix de clientes e produtos
- concentração de faturamento
- eficiência operacional
- atraso de pedidos
- lead time de entrega

---

## Principais transformações realizadas no Power Query

### 1. Parametrização do caminho do arquivo

Para evitar dependência de caminho fixo no ambiente local, foram criados dois parâmetros no Power Query:

- `pPastaArquivo`
- `pNomeArquivo`

O caminho final do arquivo é montado dinamicamente pela concatenação desses parâmetros.

### Benefício
Permite mover o arquivo de pasta sem necessidade de reescrever a query principal, facilitando manutenção e portabilidade do PBIX.

---

### 2. Leitura da planilha bruta

A aba `Export` foi utilizada como fonte de dados original.

A ingestão foi feita diretamente via `Excel.Workbook(File.Contents(...))`, conforme solicitado no desafio.

---

### 3. Normalização do cabeçalho

A planilha original apresentava cabeçalhos desalinhados em mais de uma linha, o que inviabilizava uma simples promoção de cabeçalho padrão.

Para resolver isso no Power Query, foi adotada a seguinte estratégia:

- manutenção das 23 colunas relevantes
- leitura separada das linhas que compunham o cabeçalho visual
- montagem do cabeçalho final por posição
- correção explícita do bloco desalinhado de cliente/cidade
- padronização dos nomes das colunas para uso analítico

### Exemplo de problema tratado
Algumas colunas de cliente e cidade não vinham corretamente nomeadas após a leitura inicial, exigindo ajuste manual controlado no próprio Power Query, sem tratamento externo ao arquivo.

---

### 4. Limpeza da base

Foram aplicados os seguintes tratamentos:

- remoção das linhas acima do início real dos dados
- remoção de linhas totalmente vazias
- remoção de linhas inválidas ou cabeçalhos repetidos dentro da massa
- validação numérica de chaves importantes como `num_venda` e `cliente_cod`

---

### 5. Tipagem das colunas

As colunas foram convertidas para os tipos corretos, incluindo:

- datas
- inteiros
- textos
- valores numéricos decimais

Essa etapa foi importante para garantir consistência nas medidas DAX e correta renderização dos visuais.

---

### 6. Enriquecimento da base

Na `stg_Vendas`, foram criadas colunas derivadas relevantes para análise operacional:

- `valor_custo`
- `valor_margem`
- `fl_pedido_atrasado`
- `lead_time_entrega`

#### Regras aplicadas
- **valor_custo** = custo unitário × quantidade
- **valor_margem** = valor bruto − valor custo
- **fl_pedido_atrasado** = 1 quando data de entrega > data prevista
- **lead_time_entrega** = diferença em dias entre data da venda e data da entrega

---

## Modelagem de dados

Foi adotado um **modelo estrela**, com separação entre tabela fato e dimensões.

### Tabela fato
`fVendas`

Contém:
- chaves de relacionamento
- datas
- métricas
- flags operacionais

Campos principais:
- `num_venda`
- `data_venda`
- `data_entrega_prevista`
- `data_entrega`
- `vendedor_cod`
- `cliente_cod`
- `produto_cod`
- `qtd_itens`
- `preco_unitario`
- `valor_bruto`
- `produto_custo_unitario`
- `fl_devolucao`
- `valor_custo`
- `valor_margem`
- `fl_pedido_atrasado`
- `lead_time_entrega`

### Dimensões
- `dClientes`
- `dVendedores`
- `dProdutos`
- `dCalendario`

---

## Grão da tabela fato

Um ponto importante do case foi a correta interpretação do grão da base.

O campo `num_venda` **não é uma chave única da fato**.

O nível de detalhe da tabela é:

- **pedido + cliente + produto**

Isso significa que um mesmo número de venda pode aparecer associado a mais de um cliente e mais de um produto, sem configurar erro de modelagem.

---

## Medidas principais construídas

Entre as medidas desenvolvidas, destacam-se:

- Receita Bruta
- Margem Bruta
- % Margem
- Receita YTD
- Qtd Pedidos
- % Pedidos Atrasados
- Lead Time Médio
- Share de Receita por Cliente
- Ranking de Clientes por Receita
- Insights narrativos dinâmicos

---

## Storytelling e abordagem visual

A página inicial do dashboard foi desenhada para responder rapidamente a quatro perguntas:

1. Quanto a operação vendeu?
2. Como a receita evoluiu ao longo do ano?
3. Quem concentra maior participação no faturamento?
4. Como está a eficiência operacional?

A proposta visual foi orientada por:

- leitura executiva
- hierarquia clara de informação
- alinhamento com a identidade visual da Dataside
- uso de KPIs, narrativa e gráficos simples de alta legibilidade

---

## Orientações para quem for abrir o PBIX

### 1. Verifique os parâmetros do arquivo
Ao abrir o projeto em outro ambiente, revise os parâmetros no Power Query:

- `pPastaArquivo`
- `pNomeArquivo`

Caso a base esteja em outra pasta, basta atualizar os parâmetros, sem necessidade de editar o código M.

### 2. Atualização da base
Após ajustar os parâmetros, execute:

- **Transformar Dados**
- **Fechar e Aplicar**
- **Atualizar**

### 3. Publicação
Caso deseje republicar o conteúdo no Power BI Service, será necessário:

- atualizar as credenciais da fonte local
- publicar o PBIX no workspace desejado
- revisar o link de compartilhamento

### 4. Dependência da base original
O projeto foi construído considerando a estrutura original do arquivo do case. Alterações na posição das colunas ou na disposição do cabeçalho podem exigir revisão da query de staging.

---

## Premissas e limitações

- A base contém apenas um ano de histórico
- Não há comparativo anual anterior para análises YoY
- Algumas estruturas do Excel original exigiram tratamento explícito de desalinhamento de cabeçalho
- O forecast foi considerado apenas como possibilidade analítica futura, não como componente final central do dashboard

---

## Diferenciais técnicos aplicados

- parametrização do caminho do arquivo
- staging robusta em Power Query
- tratamento de cabeçalhos desalinhados sem uso de ferramentas externas
- separação entre staging, fato e dimensões
- construção de insights narrativos dinâmicos em DAX
- preocupação com escalabilidade, clareza semântica e manutenção futura

---

## Tecnologias utilizadas

- Power BI Desktop
- Power Query (M)
- DAX
- Power BI Service

---

## Autor

Jefferson Diego da Silva

Projeto desenvolvido como case técnico para processo seletivo de Analista de Dataviz Sênior.