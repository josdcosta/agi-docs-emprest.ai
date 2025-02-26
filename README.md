# Documentações Emprest.AI 📄

## EMPRÉSTIMO CONSIGNADO E PESSOAL

### Índice
1. [Autores](#autores)
2. [Referências](#referências)
3. [1. Objetivo](#1-objetivo)
4. [2. Variáveis](#2-variáveis)
5. [3. Visão Geral do Funcionamento](#3-visão-geral-do-funcionamento)
6. [4. Dados Armazenados do Cliente](#4-dados-armazenados-do-cliente)
7. [5. Simulação de Empréstimo](#5-simulação-de-empréstimo)
8. [6. Concessão de Empréstimo](#6-concessão-de-empréstimo)
9. [7. Consulta de Dados de Empréstimo](#7-consulta-de-dados-de-empréstimo)
10. [8. Pagamento de Empréstimo](#8-pagamento-de-empréstimo)
11. [9. Refinanciamento de Empréstimo](#9-refinanciamento-de-empréstimo)
12. [11. Elegibilidade](#11-elegibilidade)
13. [12. Cálculos](#12-cálculos)

### Autores
- @Dalleth Martins
- @Josué Davi da Costa
- @Carollina Guedes
- @Victor Augusto Ferreira
- @Joao Formigari

### Referências
- Planilha de cálculo
- Leis e Regulamentações: Lei 10.820/2003 (base para consignados), Lei 14.509/2022 (margem consignável de 35%), Regulamentação INSS, Resoluções do Banco Central, Código de Defesa do Consumidor (art. 52, §2º para multa e juros mora).
- Leis e Regulamentações: Lei nº 8.078/1990: O Código de Defesa do Consumidor (CDC). Esta lei protege você quando contrata um empréstimo pessoal, garantindo informações claras, proibindo cobranças abusivas, protegendo contra problemas e ajudando a evitar o superendividamento.

### 1. Objetivo
O Emprest.AI é um backend projetado para gerenciar de forma eficiente e transparente o ciclo completo de empréstimos, abrangendo as modalidades Empréstimo Pessoal e Empréstimo Consignado. Suas funcionalidades incluem concessão de novos contratos, simulação de condições, consulta de dados, pagamento antecipado de parcela, refinanciamento (quando aplicável), portabilidade (para consignado) e cancelamento, com critérios adaptados a cada modalidade.

### 2. Variáveis
Os parâmetros abaixo do sistema Emprest.AI:


| Variável                   | Descrição                                                 | Valor Padrão            |
|----------------------------|-----------------------------------------------------------|-------------------------|
| jurosMinimoPessoal         | Taxa mínima de juros mensal (Empréstimo Pessoal)          | 8,49% ao mês            |
| jurosMaximoPessoal         | Taxa máxima de juros mensal (Empréstimo Pessoal)          | 9,99% ao mês            |
| jurosMinimoConsignado      | Taxa mínima de juros mensal (Empréstimo Consignado, 24 meses) | 1,80% ao mês            |
| jurosMaximoConsignado      | Taxa máxima de juros mensal (Empréstimo Consignado, 92 meses) | 2,14% ao mês            |
| valorMinimoPessoal         | Valor mínimo do Empréstimo Pessoal                        | R$ 100,00               |
| valorMaximoPessoal         | Valor máximo do Empréstimo Pessoal                        | R$ 20.000,00            |
| valorMinimoConsignado      | Valor mínimo do Empréstimo Consignado                     | R$ 1.000,00             |
| prazoMinimoPessoal         | Prazo mínimo em parcelas (Empréstimo Pessoal)             | 6 parcelas              |
| prazoMaximoPessoal         | Prazo máximo em parcelas (Empréstimo Pessoal)             | 30 parcelas             |
| prazoMinimoConsignado      | Prazo mínimo em parcelas (Empréstimo Consignado)          | 24 parcelas             |
| prazoMaximoConsignado      | Prazo máximo em parcelas (Empréstimo Consignado)          | 92 parcelas             |
| carenciaMaximaPessoal      | Período máximo de carência (Empréstimo Pessoal)           | 30 dias                 |
| carenciaMaximaConsignado   | Período máximo de carência (Empréstimo Consignado)        | 60 dias                 |
| idadeMaximaConsignado      | Idade máxima ao final (Empréstimo Consignado)             | 80 anos                 |
| idadeMaximaPessoal         | Idade máxima ao final (Empréstimo Pessoal)                | 75 anos                 |
| margemConsignavel          | Percentual da remuneração líquida para margem             | 35%                     |
| iof                        | Imposto sobre Operações Financeiras                       | Conforme legislação     |
| percentualRendaPessoal     | Percentual máximo da renda líquida para parcela (Empréstimo Pessoal)  | 30%            |
| percentualMinimoRefinanciamento | Percentual mínimo de parcelas pagas para refinanciamento | 20%                     |

### 3. Visão Geral do Funcionamento
O sistema é estruturado em áreas principais, aplicáveis a ambas as modalidades com ajustes específicos:

- Concessão de Empréstimos: Análise de crédito adaptada (Consignado: margem consignável; Pessoal: score e renda). Simulação e aprovação de contratos.
- Consulta de Empréstimos: Acompanhamento de status, parcelas e histórico de pagamentos.
- Pagamento de Empréstimos: Registro de pagamentos, incluindo antecipações.
- Refinanciamento: Renegociação de contratos existentes (ambas as modalidades).
- Portabilidade: Transferência de consignados e empréstimo pessoal para outro banco.
- Cancelamento de Contrato: Gestão de desistências ou finalizações.

### 4. Dados Armazenados do Cliente
```json
{
  "idCliente": [cpf],
  "nome": "[nome]",
  "remuneracaoLiquidaMensal": "[valor em reais]",
  "idade": "[número]",
  "tipoVinculo": "[aposentado, servidor, pensionista, empregado ou nulo]",
}
```

### 5. Simulação de Empréstimo
#### 5.1. Requisição - Sistema recebe informações do usúario identificando o tipo de modalidade.
##### Empréstimo Consignado - Aposentado, pensionistas, funcionários públicos.
```json
{
  "idCliente": [cpf],
  "valorEmprestimo": "[valor em reais]",
  "tipoEmprestimo": "[consignado ou pessoal]",
  "quantidadeParcelas": "[número]",
  "contratarSeguro": "[true ou false]",
  "dataInicioPagamento": "[data no formato DD/MM/AAAA]"
}
```
##### Empréstimo Pessoal
```json
{
  "idCliente": [cpf]
  "valorEmprestimo": "[valor em reais]",
  "tipoEmprestimo": "[consignado ou pessoal]",
  "quantidadeParcelas": "[número]",
  "contratarSeguro": "[true ou false]",
  "dataInicioPagamento": "[data no formato DD/MM/AAAA]"
}
```
#### 5.2. Processo Passo a Passo
**Passo 1: Consulta de Dados do Cliente**  
O sistema busca o `idCliente` na base e retorna `remuneracaoLiquidaMensal`, `idade`, `tipoVinculo` (para consignado) e por meio
do `idCliente`consulta  o analisador de risco que retorna  o `scoreCredito` (para pessoal). Se não encontrado, "Erro: Cliente não encontrado".

**Passo 2: Verificação Inicial de Elegibilidade**  
Empréstimo Consignado:
- Aplica [11.1.5. Tipo de Vínculo](#1115-tipo-de-vínculo).
- Aplica [11.1.2. Idade Máxima](#1112-idade-máxima).
- Aplica [11.1.3. Quantidade de Parcelas](#1113-quantidade-de-parcelas).
- Calcula dias de carência e aplica [11.1.6. Carência](#1116-carência).  

Empréstimo Pessoal:
- Aplica [11.1.2. Idade Máxima](#1112-idade-máxima).
- Aplica [11.2.1. Valor do Empréstimo](#1121-valor-do-empréstimo).
- Aplica [11.2.2. Quantidade de Parcelas](#1122-quantidade-de-parcelas).
- Aplica [11.2.3. Score de Crédito](#1123-score-de-crédito).
- Calcula dias de carência e aplica [11.2.5. Carência](#1125-carência).

**Passo 3: Determinação da Capacidade de Pagamento**  
Consignado: Executa [12.1. Margem Consignável](#121-margem-consignável).  
Pessoal: Executa [12.2. Capacidade de Pagamento](#122-capacidade-de-pagamento).

**Passo 4: Definição da Taxa de Juros**  
Consignado: Aplica [12.3. Taxa de Juros Mensal](#123-taxa-de-juros-mensal) e verifica [11.1.4. Taxa de Juros](#1114-taxa-de-juros).  
Pessoal: Aplica [12.3. Taxa de Juros Mensal](#123-taxa-de-juros-mensal) e verifica [11.2.4. Taxa de Juros](#1124-capacidade-de-pagamento).

**Passo 5: Cálculo do Custo do Seguro**  
Se `contratarSeguro = true`, aplica [12.4. Custo do Seguro](#124-custo-do-seguro).

**Passo 6: Cálculo do IOF**  
Executa [12.5. IOF](#125-iof).

**Passo 7: Cálculo do Valor Total Financiado**  
Aplica [12.6. Valor Total Financiado](#126-valor-total-financiado).

**Passo 8: Cálculo da Parcela Mensal**  
Executa [12.7. Parcela Mensal](#127-parcela-mensal).

**Passo 9: Validação Final de Elegibilidade**  
Consignado: Aplica [11.1.1. Margem Consignável](#1111-margem-consignável).  
Pessoal: Aplica [11.2.4. Capacidade de Pagamento](#1124-capacidade-de-pagamento).

### Passo 10: Geração da Tabela de Parcelas

Com base no `valorTotalFinanciado`, `quantidadeParcelas`, `taxaJurosMensal` e `dataInicioPagamento`, o sistema calcula e gera uma tabela detalhando cada parcela, incluindo:

1. **Número da parcela**
2. **Data de vencimento** (mensal a partir da `dataInicioPagamento`)
3. **Valor da parcela**
4. **Juros da parcela**
5. **Amortização do principal**
6. **Saldo devedor restante**

**Passo 11: Retorno da Simulação**  
Retorna os valores calculados sem gravar o contrato.

#### 5.3. Saída
##### Empréstimo Consignado/Pessoal
```json
{
  "idCliente": [cpf]
  "valorEmprestimo": "[valor em reais]",
  "tipoEmprestimo": "[consignado ou pessoal]",
  "quantidadeParcelas": "[número]",
  "taxaJurosMensal": "[valor decimal]",
  "custoSeguro": "[valor em reais]",
  "iof": "[valor em reais]",
  "valorTotalFinanciado": "[valor em reais]",
  "parcelaMensal": "[valor em reais]",
  "cetMensal": "[valor decimal]",
  "tabelaParcelas": [
    {
      "numeroParcela": "[número]",
      "dataVencimento": "[data no formato DD/MM/AAAA]",
      "valorParcela": "[valor em reais]",
      "juros": "[valor em reais]",
      "amortizacao": "[valor em reais]",
      "saldoDevedor": "[valor em reais]"
    }
    ... Demais parcelas
  ],
  "mensagem": "Simulação realizada com sucesso."
}
```

### 6. Concessão de Empréstimo
#### 6.1. Requisição
Empréstimo Consignado
```json
{
  "idCliente": "[cpf]",
  "valorEmprestimo": "[valor em reais]",
  "tipoEmprestimo": "[consignado ou pessoal]",
  "quantidadeParcelas": "[número]",
  "contratarSeguro": "[true ou false]",
  "dataInicioPagamento": "[data no formato DD/MM/AAAA]"
}
```

Empréstimo Pessoal
```json
{
  "idCliente": "[cpf]",
  "valorEmprestimo": "[valor em reais]",
  "tipoEmprestimo": "[pessoal]",
  "quantidadeParcelas": "[número]",
  "contratarSeguro": "[false]",
  "dataInicioPagamento": "[data no formato DD/MM/AAAA]"
}
```

#### 6.2. Processo Passo a Passo
- Consulta de Dados do Cliente: Mesmo que Simulação.
- Verificação Inicial de Elegibilidade: Mesmo que Simulação (ver [11.1](#111-empréstimo-consignado) para consignado ou [11.2](#112-empréstimo-pessoal) para pessoal).
- Determinação da Capacidade de Pagamento: Mesmo que Simulação.
- Definição da Taxa de Juros: Mesmo que Simulação.
- Cálculo do Custo do Seguro: Mesmo que Simulação.
- Cálculo do IOF: Mesmo que Simulação.
- Cálculo do Valor Total Financiado: Mesmo que Simulação.
- Cálculo da Parcela Mensal: Mesmo que Simulação.
- Validação Final de Elegibilidade: Mesmo que Simulação (ver [11.1](#111-empréstimo-consignado) para consignado ou [11.2](#112-empréstimo-pessoal) para pessoal).
- Gera a tabela de parcelas para o contrato: Idêntico ao passo da simulação, gera a tabela de parcelas para o contrato.
- Registro do Contrato: Cria o contrato e associa o pagamento (folha para consignado, débito automático para pessoal).

#### 6.3. Saída
(Idêntica à Simulação, com "mensagem": "Empréstimo concedido com sucesso.")

### 7. Consulta de Dados de Empréstimo
#### 7.1. Requisição
```json
{
  "idCliente": [cpf],
  "idEmprestimo": "[identificador único]"
}
```
### 7.2. Processo Passo a Passo

1. **Consulta de Dados do Cliente:**
   - Valida `idCliente`. Se não encontrado, retorna "Erro: Cliente não encontrado".

2. **Verificação do Empréstimo:**
   - Busca `idEmprestimo`. Se inválido, retorna "Erro: Empréstimo não encontrado ou inválido".

3. **Recuperação dos Dados do Empréstimo:**
   - Obtém dados do contrato, incluindo a tabela de parcelas.

4. **Consulta do Histórico de Pagamentos:**
   - Verifica parcelas pagas e restantes, atualizando o status de cada parcela na tabela ("paga" ou "pendente").

5. **Cálculo do Saldo Devedor:**
   - Executa 12.8. Saldo Devedor.

6. **Retorno dos Dados:**
   - Compila e retorna as informações, incluindo a tabela de parcelas atualizada.

#### 7.3. Saída
```json
{
  "idCliente": "[cpf]",
  "idEmprestimo": "[identificador único]",
  "valorEmprestimo": "[valor em reais]",
  "quantidadeParcelas": "[número]",
  "taxaJurosMensal": "[valor decimal]",
  "custoSeguro": "[valor em reais]",
  "iof": "[valor em reais]",
  "valorTotalFinanciado": "[valor em reais]",
  "parcelaMensal": "[valor em reais]",
  "totalParcelasPagas": "[número]",
  "totalParcelasRestantes": "[número]",
  "saldoDevedorAtualizado": "[valor em reais]",
  "tabelaParcelas": [
    {
      "numeroParcela": "[número]",
      "dataVencimento": "[data no formato DD/MM/AAAA]",
      "valorPresenteParcela": "[valor em reais]",
      "juros": "[valor em reais]",
      "multa": "[valor em reais ou null]",
      "jurosMora": "[valor em reais ou null]",
      "amortizacao": "[valor em reais]",
      "status": "[paga ou pendente]",
      "dataPagamento": "[data no formato DD/MM/AAAA ou null]"
    }
  ],
  "mensagem": "Consulta realizada com sucesso."
}
```

### 8. Pagamento de Empréstimo
#### 8.1. Requisição
##### Pagamento Parcela
```json
{
  "idCliente": "[cpf]",
  "idEmprestimo": "[identificador único]",
  "valorPagamento": "[valor em reais]",
  "numeroParcela": "[parcelas1, parcela2, ...]",
  "dataPagamento": "[data no formato DD/MM/AAAA]"
}
```


### 8.2. Processo Passo a Passo

1. **Consulta de Dados do Cliente:**
   - Valida `idCliente`. Se não encontrado, retorna "Erro: Cliente não encontrado".

2. **Verificação do Empréstimo:**
   - Busca `idEmprestimo`. Se inválido, retorna "Erro: Empréstimo não encontrado ou inválido".

3. **Validação do Status do Empréstimo:**
   - Confirma se está ativo. Se todas as parcelas estiverem quitadas, retorna "Erro: Empréstimo já liquidado".

4. **Consulta do Saldo Devedor Atual:**
   - Executa 12.8. Saldo Devedor com base nas parcelas pendentes.

5. **Processamento do Pagamento:**
   - Valida o `numeroParcela` informado. Se inválido (fora do intervalo ou já pago), retorna "Erro: Parcela inválida ou já quitada".
   - **5.1. Verificação de Atraso:** Se `dataPagamento > dataVencimento`, calcula:
     - Multa: 2% sobre `valorParcela` (limite do Código de Defesa do Consumidor, art. 52, §2º).
     - Juros de mora: 0,033% ao dia sobre `valorParcela`, proporcional aos dias de atraso.
     - Soma ao `valorParcela` para obter `valorTotalAjustado`. Se `valorPagamento < valorTotalAjustado`, retorna "Erro: Valor insuficiente para a parcela atrasada".
   - Aplica o `valorPagamento` à parcela especificada, marcando-a como "paga" na `dataPagamento`. Se houver excedente, registra como crédito ou devolve (regra a definir).

6. **Atualização do Contrato:**
   - Atualiza a tabela de parcelas com `multa`, `jurosMora` e `valorTotalAjustado` (se aplicável), refletindo a parcela paga e recalculando o saldo devedor.

7. **Retorno da Confirmação:**
   - Retorna o status atualizado, incluindo a tabela de parcelas revisada com a parcela paga.

#### 8.3. Saída
##### Pagamento Parcela
```json
{
  "idCliente": "[cpf]",
  "idEmprestimo": "[identificador único]",
  "valorEmprestimo": "[valor em reais]",
  "quantidadeParcelas": "[número]",
  "taxaJurosMensal": "[valor decimal]",
  "custoSeguro": "[valor em reais]",
  "iof": "[valor em reais]",
  "valorTotalFinanciado": "[valor em reais]",
  "parcelaMensal": "[valor em reais]",
  "totalParcelasPagas": "[número]",
  "totalParcelasRestantes": "[número]",
  "saldoDevedorAtualizado": "[valor em reais]",
  "tabelaParcelas": [
    {
      "numeroParcela": "[número]",
      "dataVencimento": "[data no formato DD/MM/AAAA]",
      "valorPresenteParcela": "[valor em reais]",
      "juros": "[valor em reais]",
      "multa": "[valor em reais ou null]",
      "jurosMora": "[valor em reais ou null]",
      "amortizacao": "[valor em reais]",
      "status": "[paga ou pendente]",
      "dataPagamento": "[data no formato DD/MM/AAAA ou null]"
    }
  ],
  "mensagem": "Pagamento da parcela registrado com sucesso."
}
```

#### 8.4. Tratamento de Atrasos
- Se uma parcela não for paga até a `dataVencimento`, aplica-se:
  - **Multa**: 2% sobre o valor da parcela (limite do Código de Defesa do Consumidor, art. 52, §2º).
  - **Juros de Mora**: 1% ao mês (0,033% ao dia) sobre o valor da parcela, proporcional aos dias de atraso.
- Exemplo: Parcela de R$ 305.96 vencida em 01/09/2025, paga em 11/09/2025 (10 dias de atraso):
  - Multa: `305.96 * 0,02 = 6,12`.
  - Juros de mora: `305.96 * 0,00033 * 10 = 1,00`.
  - Total a pagar: `305.96 + 6,12 + 1,00 = 313,08`.

## 9. Refinanciamento de Empréstimo
## 9.1. Requisição

```json
{
  "idCliente": "[cpf]",
  "idEmprestimoOriginal": "[identificador único]",
  "novoValorEmprestimo": "[valor em reais]",
  "novaQuantidadeParcelas": "[número]",
  "contratarSeguro": "[true ou false]",
  "dataInicioPagamento": "[data no formato DD/MM/AAAA]"
}
```

## 9.2. Processo Passo a Passo

1. **Consulta de Dados do Cliente:**
   - Valida `idCliente`.

2. **Verificação do Empréstimo Original:**
   - Busca `idEmprestimoOriginal`.

3. **Validação do Status:**
   - Confirma se ativo.

4. **Verificação de Elegibilidade:**
   - Aplica [11.3.1. Percentual Mínimo Pago](#1131-percentual-mínimo-pago).
   - Consignado: Aplica [11.1.2. Idade Máxima](#1112-idade-máxima), [11.1.3. Quantidade de Parcelas](#1113-quantidade-de-parcelas), [11.1.6. Carência](#1116-carência).
   - Pessoal: Aplica [11.2.2. Quantidade de Parcelas](#1122-quantidade-de-parcelas), [11.2.5. Carência](#1125-carência).

5. **Cálculo do Saldo Devedor:**
   - Executa 12.8. Saldo Devedor do contrato original.

6. **Determinação da Capacidade:**
   - Consignado: 12.1. Margem Consignável.
   - Pessoal: 12.2. Capacidade de Pagamento.

7. **Definição da Taxa de Juros:**
   - Aplica 12.3. Taxa de Juros Mensal.

8. **Cálculo do Custo do Seguro:**
   - Aplica 12.4. Custo do Seguro se `contratarSeguro = true`.

9. **Cálculo do IOF:**
   - Executa 12.5. IOF.

10. **Cálculo do Valor Total Financiado:**
    - Aplica 12.6. Valor Total Financiado, somando `saldoDevedorOriginal` e `novoValorEmprestimo`.

11. **Cálculo da Nova Parcela:**
    - Executa 12.7. Parcela Mensal.

12. **Geração da Tabela de Parcelas:**
    - Com base no `valorTotalFinanciado`, `novaQuantidadeParcelas`, `taxaJurosMensal` e `dataInicioPagamento`, gera a tabela com:
      - Número da parcela
      - Data de vencimento (mensal a partir da `dataInicioPagamento`)
      - Valor da parcela
      - Juros da parcela
      - Amortização do principal
      - Saldo devedor restante
      - Status (inicialmente "pendente")
      - Data de pagamento (inicialmente null)

13. **Validação Final:**
    - Consignado: Aplica [11.1.1. Margem Consignável](#1111-margem-consignável).
    - Pessoal: Aplica [11.2.4. Capacidade de Pagamento](#1124-capacidade-de-pagamento).

14. **Registro do Refinanciamento:**
    - Cria novo contrato com a tabela de parcelas e marca o original como "refinanciado".

## 9.3. Saída

```json
{
  "idCliente": "[cpf]",
  "idEmprestimo": "[identificador único]",
  "valorEmprestimo": "[valor em reais]",
  "quantidadeParcelas": "[número]",
  "taxaJurosMensal": "[valor decimal]",
  "custoSeguro": "[valor em reais]",
  "iof": "[valor em reais]",
  "valorTotalFinanciado": "[valor em reais]",
  "parcelaMensal": "[valor em reais]",
  "totalParcelasPagas": "[número]",
  "totalParcelasRestantes": "[número]",
  "saldoDevedorAtualizado": "[valor em reais]",
  "tabelaParcelas": [
    {
      "numeroParcela": "[número]",
      "dataVencimento": "[data no formato DD/MM/AAAA]",
      "valorPresenteParcela": "[valor em reais]",
      "juros": "[valor em reais]",
      "multa": "[valor em reais ou null]",
      "jurosMora": "[valor em reais ou null]",
      "amortizacao": "[valor em reais]",
      "status": "[paga ou pendente]",
      "dataPagamento": "[data no formato DD/MM/AAAA ou null]"
    }
  ],
  "mensagem": "Refinanciamento realizado com sucesso."
}
```

---
---
---
# 11. Elegibilidade

## 11.1. Empréstimo CONSIGNADO

### 11.1.1. Margem Consignável
Parcela ≤ (remuneracaoLiquida * margemConsignavel) - soma de parcelas ativas.

### 11.1.2. Idade Máxima
`idade + quantidadeParcelas / 12 < idadeMaximaConsignado`: A idade aproximada do cliente ao final do contrato não deve atingir ou exceder idadeMaximaConsignado.

### 11.1.3. Quantidade de Parcelas
Entre 24 e 92 parcelas.

### 11.1.4. Taxa de Juros
Taxa mensal ≤ 2,14%.

### 11.1.5. Tipo de Vínculo
"Aposentado", "servidor público" ou outro válido.

### 11.1.6. Carência
Dias até o primeiro pagamento ≤ carenciaMaximaConsignado.

---
---
---
## 11.2. Empréstimo PESSOAL

### 11.2.1. Idade Máxima
`idade + quantidadeParcelas / 12 < idadeMaximaPessoal`: A idade aproximada do cliente ao final do contrato não deve atingir ou exceder idadeMaximaPessoal.

### 11.2.2. Valor do Empréstimo
valorMinimoPessoal ≤ valorEmprestimo ≤ valorMaximoPessoal, conforme score:

| Faixa de Score | Nível de Risco     | Limite Crédito     |  
|----------------|--------------------|--------------------|
| 0-200          | Altíssimo risco    | N/A                |
| 201-400        | Alto risco         | R$ 100 a R$ 1.000  |
| 401-600        | Risco moderado     | R$ 100 a R$ 5.000  | 
| 601-800        | Risco baixo        | R$ 100 a R$ 15.000 |
| 801-1000       | Risco muito baixo  | R$ 100 a R$ 20.000 | 
|----------------|--------------------|--------------------|


### 11.2.3. Quantidade de Parcelas
prazoMinimoPessoal ≤ quantidadeParcelas ≤ prazoMaximoPessoal, conforme score:

| Faixa de Score | Nível de Risco     |  Meses              |
|----------------|--------------------|---------------------| 
| 0-200          | Altíssimo risco    |  N/A                |
| 201-400        | Alto risco         |  6 a 12             |
| 401-600        | Risco moderado     |  6 a 18             |
| 601-800        | Risco baixo        |  6 a 24             |
| 801-1000       | Risco muito baixo  |  6 a 30             |
|----------------|--------------------|---------------------|

### 11.2.4. Score de Crédito
scoreCredito ≥ 201.

### 11.2.5. Capacidade de Pagamento
Parcela ≤ rendaTotalLiquida * remuneracaoLiquida.

### 11.2.6 Carência
Dias até o primeiro pagamento ≤ carenciaMaximaPessoal.

---
---
---

## 11.3. Refinanciamento (Comum)

### 11.3.1. Percentual Mínimo Pago
≥ 20% das parcelas pagas.

## 11.4. Portabilidade (Empréstimo Consignado, Empréstimo Pessoal).

### 11.4.1. Parcelas em Dia
Sem parcelas vencidas.

### 11.4.2. Aceitação do Banco Destino
bancoDestino deve aceitar a portabilidade.

---
---
---

# 12. Cálculos

### 12.1. Renda Total Liquida
rendaTotalLiquida = (Rendas Familiar + remuneracaoLiquida - Total de Despesas ou Dividas) / quantidadeMembrosFamilia.

## 12.2. Margem Consignável (Empréstimo Consignado)
margemMaxima = remuneracaoLiquida * margemConsignavel - Parcela de Emprestimos Ativos

## 12.3. Capacidade de Pagamento (Empréstimo Pessoal)
capacidadeMaxima = rendaTotalLiquida * percentualRendaPessoal

## 12.4. Taxa de Juros Mensal
**Consignado**:
TaxaJurosMensal = 0,018 + 0,00005 * (quantidadeParcelas - 24), limitada a 2,14%.

**Pessoal**:
Interpolação entre jurosMinimoPessoal (8,49%) e jurosMaximoPessoal (9,99%) com base em scoreCredito.
| Faixa de Score | Nível de Risco     | Taxa               | 
|----------------|--------------------|--------------------|
| 0-200          | Altíssimo risco    | N/A                | 
| 201-400        | Alto risco         | 9,99%              | 
| 401-600        | Risco moderado     | 9,49% a 9,99%      | 
| 601-800        | Risco baixo        | 8,99% a 9,49%      | 
| 801-1000       | Risco muito baixo  | 8,49% a 8,99%      |
|----------------|--------------------|--------------------|

Taxa = Taxa_mín + [(Taxa_máx - Taxa_mín) × (Score - Score_mín)] / (Score_máx - Score_mín)

## 12.5. Custo do Seguro
CustoSeguro = valorBase * (0,0025 + 0,00005 * idade) * (quantidadeParcelas / 12)

## 12.6. IOF
percentualFixo = 0,0038
percentualVariado = 0,000082
IOF = (percentualFixo * valorBase) + (percentualVariado * valorBase * min(diasFinanciamento, 365))

## 12.7. Valor Total Financiado
ValorInicial = valorBase + IOF + CustoSeguro
ValorTotalFinanciado = ValorInicial * (1 + TaxaJurosMensal / 30) ^ diasCarencia

## 12.8. Parcela Mensal
ParcelaMensal = [ValorTotalFinanciado * TaxaJurosMensal] / [1 - (1 + TaxaJurosMensal)^(-quantidadeParcelas)]

## 12.9. Saldo Devedor
SaldoDevedor = ValorTotalFinanciado * [(1 + TaxaJurosMensal)^quantidadeParcelasRestantes - 1] / [(1 + TaxaJurosMensal)^quantidadeParcelasTotais - 1]
