# 📊 Analise Financeira e Cobrança de Faturas em SQL

Este repositório contém um conjunto de consultas analíticas voltadas para o setor financeiro e de cobrança. O objetivo do projeto é demonstrar a aplicação de **regras de negócio aplicadas**, **tratamento defensivo de dados nulos** e **agregação condicional (*Conditional Aggregation*)** para transformar dados brutos em relatórios executivos consolidados.

---

## 🛠️ Tecnologias e Conceitos Utilizados

- **SGBD:** PostgreSQL / ANSI SQL
- **Relacionamento de Tabelas:** `INNER JOIN` com aliases estratégicos
- **Tratamento de Dados Ausentes:** `COALESCE` para *fallback* de contatos, datas e valores monetários
- **Lógica Condicional:** `CASE WHEN` para mapeamento de status e classificação de regras financeiras
- **Agregação Condicional:** Uso de `SUM()` e `COUNT()` combinados com `CASE WHEN` para pivotar métricas por cliente em uma única consulta

---

## 🗄️ Estrutura do Banco de Dados

O projeto utiliza duas tabelas relacionais com dados simulados para cobrir casos reais de inadimplência, pagamentos parciais, pagamentos em dia e dados cadastrais ausentes:

1. **`CLIENTES`**: Armazena dados cadastrais (ID, Nome, E-mail e Telefone).
2. **`FATURAS`**: Histórico de cobrança contendo valores emitidos, valores pagos, datas de vencimento e datas de pagamento.

---

## 🎯 Soluções Analíticas Desenvolvidas

### 1. Higienização e Definição do Canal de Contato
Tratamento encadeado de nulos com `COALESCE` para definir o canal prioritário de cobrança: **E-mail** $\rightarrow$ **Telefone** $\rightarrow$ `'SEM CONTATO CADASTRADO'`.

### 2. Acompanhamento Temporal de Quitação
Mapeamento de faturas em aberto e verificação da pontualidade de pagamento comparando `DATA_PAGAMENTO` e `DATA_VENCIMENTO`.

### 3. Análise de Saldo e Risco Financeiro
Classificação do status financeiro por fatura (`QUITADO`, `PAGAMENTO_PARCIAL` ou `PENDENTE_TOTAL`), tratando `VALOR_PAGO` nulo como `0.00`.

### 4. Volumetria de Faturas por Cliente
Contagem condicional utilizando `COUNT(CASE WHEN ... THEN 1 END)` para segmentar a quantidade de faturas pagas vs. em aberto de cada cliente.

### 5. Resumo Financeiro Consolidado
Consolidação do fluxo financeiro agrupado por cliente, exibindo:
- **Total Emitido:** `SUM(VALOR_FATURA)`
- **Total Arrecadado:** `SUM(COALESCE(VALOR_PAGO, 0.00))`
- **Total Pendente:** `COALESCE(SUM(CASE WHEN DATA_PAGAMENTO IS NULL THEN VALOR_FATURA END), 0.00)`

### 6. Arrecadação por Forma de Quitação (Em Dia vs. Com Atraso)
Pivotagem monetária que isola os valores efetivamente pagos entre recebimentos **no prazo** e recebimentos **com atraso**, garantindo retorno `0.00` para categorias sem movimentação.

---

## 📌 Principais Aprendizados Técnicos

- **Prevenção de Nulos em Relatórios (`COALESCE` Externo):** Envolver a função de agregação com `COALESCE(SUM(...), 0.00)` impede que clientes sem movimentações na condição filtrada retornem `NULL`, garantindo compatibilidade direta com dashboards de BI (Power BI / Tableau).
- **Contagem vs. Soma Condicional:** Uso de `THEN 1` para contagem de volume de registros (`COUNT`) e `THEN COLUNA_VALOR` para acúmulo financeiro (`SUM`).
- **Garantia de Integridade:** Uso de `ELSE 0` dentro das estruturas `CASE` em operações monetárias para evitar falhas de cálculo no resultado final.
