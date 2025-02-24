# Documentação do Emprest.AI - Backend de Empréstimo Pessoal

## Índice
1. [Autores](#autores)
2. [Referências](#referencias)
3. [Objetivo](#1-objetivo)
4. [Visão Geral do Funcionamento](#2-visao-geral-do-funcionamento)
    1. [Regras Principais](#regras-principais)
5. [Gerenciamento de Clientes](#3-gerenciamento-de-clientes)
    1. [Cadastro/Atualização de Cliente](#31-cadastroatualizacao-de-cliente)
6. [Concessão de Empréstimos](#4-concessao-de-emprestimos)
    1. [Entrada de Dados](#41-entrada-de-dados)
    2. [Processo de Cálculo](#42-processo-de-calculo)
    3. [Saídas](#43-saidas)
7. [Consulta e Atualização de Parcelas](#5-consulta-e-atualizacao-de-parcelas)
    1. [Consulta](#51-consulta)
    2. [Atualização](#52-atualizacao)
8. [Antecipação de Parcelas](#6-antecipacao-de-parcelas)
    1. [Entrada](#61-entrada)
9. [Observações](#7-observacoes)

## Autores
- @Dalleth Martins
- @Josué Davi da Costa
- @Carollina Guedes
- @Victor Augusto Ferreira

## Referências
- Planilha de Cálculos
- Leis e Regulamentações: Código de Defesa do Consumidor (art. 52, §2º para multa e juros mora), Resoluções do Banco Central (IOF e taxas), Lei 13.977/2020 (proteção ao consumidor em crédito).

## 1. Objetivo
O Emprest.AI - Empréstimo Pessoal é um sistema backend projetado para gerenciar de forma eficiente e transparente o ciclo completo de empréstimos pessoais. Isso inclui o cadastro e análise de crédito do cliente, a concessão de novos contratos, o acompanhamento de parcelas, a antecipação de pagamentos e a renovação de empréstimos. O sistema define limites de crédito com base na renda e no score de crédito, calcula taxas de juros ajustadas ao perfil de risco e incorpora encargos como TAC e IOF, alinhando-se às práticas de mercado para oferecer uma experiência otimizada e segura.

## 2. Visão Geral do Funcionamento
O sistema é estruturado em cinco áreas principais:
- **Gerenciamento de Clientes:** Cadastro e atualização de dados pessoais e financeiros.
- **Concessão de Empréstimos:** Análise de crédito, simulação e aprovação de novos contratos.
- **Consulta de Empréstimos:** Acompanhamento de status, parcelas e histórico.
- **Atualização de Dados:** Registro de pagamentos, incluindo antecipações e ajustes.
- **Renovação de Contrato:** Processamento de novos empréstimos com base no saldo existente.

### Regras Principais
- **Limite de Crédito:** Até 30% da renda líquida mensal, ajustado por score e idade.
- **Taxas de Juros:** De 1,0% a 4,0% ao mês, conforme risco, prazo e perfil do cliente.
- **Prazos:** De 6 a 60 meses, com limites reduzidos acima de 65 anos.
- **Encargos:** Incluem TAC, IOF, seguro opcional, taxa de cadastro e, se aplicável, taxa de liquidação antecipada.
- **CET:** Reflete o custo total, incluindo juros e encargos.

## 3. Gerenciamento de Clientes

### 3.1. Cadastro/Atualização de Cliente

#### Entrada
| Campo          | Tipo    | Descrição                  | Exemplo           | Obrigatório? |
|----------------|---------|----------------------------|-------------------|--------------|
| idCliente      | String  | CPF do cliente             | "987.654.321-09"  | Sim          |
| nome           | String  | Nome completo              | "Maria Souza"     | Sim          |
| rendaLiquida   | Decimal | Renda líquida mensal       | 5000.00           | Sim          |
| idade          | Inteiro | Idade na data atual        | 45                | Sim          |
| scoreCredito   | Inteiro | Pontuação de crédito (0-1000) | 750               | Sim          |

#### Processo
Registra ou atualiza os dados na tabela de clientes. O limite de crédito inicial é calculado como rendaLiquida * 0.30, ajustado por score e idade, com validação de CPF e criptografia de dados sensíveis.

#### Requisição (Exemplo)
```json
{
  "idCliente": "987.654.321-09",
  "nome": "Maria Souza",
  "rendaLiquida": 5000.00,
  "idade": 45,
  "scoreCredito": 750
}
```

#### Saída
```json
{
  "idCliente": "987.654.321-09",
  "nome": "Maria Souza",
  "rendaLiquida": 5000.00,
  "idade": 45,
  "scoreCredito": 750,
  "limiteCredito": 1500.00,
  "mensagem": "Cliente cadastrado/atualizado com sucesso."
}
```

## 4. Concessão de Empréstimos

### 4.1. Entrada de Dados

| Campo               | Tipo    | Descrição                            | Exemplo           | Obrigatório? |
|---------------------|---------|--------------------------------------|-------------------|--------------|
| idCliente           | String  | CPF do cliente                       | "987.654.321-09"  | Sim          |
| valorEmprestimo     | Decimal | Valor solicitado                     | 15000.00          | Sim          |
| quantidadeParcelas  | Inteiro | Número de parcelas (6 a 60)          | 36                | Não          |
| dataInicioPagamento | Data    | Data do primeiro pagamento (futura)  | "15/03/2025"      | Não          |
| contratarSeguro     | Booleano| Opção de contratar seguro            | true              | Sim          |

*Nota:* Se quantidadeParcelas não for informada, o sistema sugere opções. Se dataInicioPagamento não for fornecida, usa carência de 30 dias a partir da data da solicitação.

#### Dados Adicionais (Renovação)
| Campo                | Tipo    | Descrição                            | Exemplo           | Obrigatório? |
|----------------------|---------|--------------------------------------|-------------------|--------------|
| idEmprestimoOriginal | String  | Identificador do empréstimo existente| "EMP-00456"       | Sim (renovação) |
| novoValorEmprestimo  | Decimal | Valor adicional solicitado           | 5000.00           | Não          |

### 4.2. Processo de Cálculo

#### Consulta Inicial
Valida o idCliente na tabela de clientes, obtendo rendaLiquida, scoreCredito, idade e empréstimos ativos. Para renovações, consulta o saldo devedor do idEmprestimoOriginal.

#### Validação
- "Erro: Cliente não encontrado"
- "Erro: Valor solicitado excede limite de crédito (X.XX)"
- "Erro: Idade fora da faixa permitida (18-75)"

#### Limite de Crédito
Calculado como rendaLiquida * 0.30, ajustado por score e idade.

#### Taxa de Juros
Determinada por risco, score, idade e prazo:
- **Faixas:** Baixo risco (1,0%-1,5%), Médio (1,6%-2,5%), Alto (2,6%-4,0%).
- **Ajuste por Idade:** +0,5% (66-70 anos), +1,0% (71-75 anos).

*Exemplo:* Score 750, 45 anos, 36 meses → 1,3%. Score 600, 68 anos, 24 meses → 2,0% + 0,5% = 2,5%.

#### Encargos
- **TAC:** 2% do valorEmprestimo.
- **IOF:** (0,0038 * ValorEmprestimo) + (0,000082 * ValorEmprestimo * min(PrazoEmDias, 365)).
- **Seguro (opcional):** 0,001 * idade * ValorEmprestimo.

#### Valor Total Financiado
- **ValorInicial = ValorEmprestimo + TAC + IOF + CustoSeguro**
- **ValorTotalFinanciado = ValorInicial * (1 + TaxaJurosMensal / 30) ^ DiasCarencia**

#### Parcela Mensal (Price)
- **Parcela = [ValorTotalFinanciado * TaxaJurosMensal] / [1 - (1 + TaxaJurosMensal)^(-QuantidadeParcelas)]**

#### CET
- **ValorEmprestimo = Parcela * [(1 - (1 + CET)^(-QuantidadeParcelas)) / CET]**

#### Validação Final
- Parcela ≤ 30% da renda.
- quantidadeParcelas entre 6 e máximo por idade (60, 48 ou 36 meses).

### 4.3. Saídas

#### Com quantidadeParcelas Informada

##### Requisição (Exemplo)
```json
{
  "idCliente": "987.654.321-09",
  "valorEmprestimo": 15000.00,
  "quantidadeParcelas": 36,
  "dataInicioPagamento": "15/03/2025",
  "contratarSeguro": true
}
```

##### Saída
```json
{
  "idCliente": "987.654.321-09",
  "valorEmprestimo": 15000.00,
  "parcela": 525.50,
  "quantidadeParcelas": 36,
  "dataInicioPagamento": "15/03/2025",
  "dataFimContrato": "15/03/2028",
  "taxaJurosMensal": 0.0180,
  "taxaEfetivaMensal": 0.0215,
  "contratarSeguro": true,
  "custoSeguro": 675.00,
  "tac": 300.00,
  "iof": 228.90,
  "valorTotalFinanciado": 16203.90,
  "saldoDevedor": 16203.90,
  "totalPago": 0.00,
  "totalDevido": 18918.00,
  "limiteUtilizado": 525.50,
  "limiteRestante": 974.50,
  "prazoMaximoPermitido": 60
}
```

#### Sem quantidadeParcelas

##### Requisição (Exemplo)
```json
{
  "idCliente": "987.654.321-09",
  "valorEmprestimo": 15000.00,
  "contratarSeguro": true
}
```

##### Saída
```json
{
  "idCliente": "987.654.321-09",
  "valorEmprestimo": 15000.00,
  "contratarSeguro": true,
  "opcoesParcelamento": [
    {"quantidadeParcelas": 12, "parcela": 1392.75, "taxaJurosMensal": 0.0150, "valorTotalFinanciado": 15953.90, "totalDevido": 16713.00},
    {"quantidadeParcelas": 24, "parcela": 742.10, "taxaJurosMensal": 0.0160, "valorTotalFinanciado": 16003.90, "totalDevido": 17810.40},
    {"quantidadeParcelas": 36, "parcela": 525.50, "taxaJurosMensal": 0.0180, "valorTotalFinanciado": 16203.90, "totalDevido": 18918.00}
  ]
}
```

## 5. Consulta e Atualização de Parcelas

### 5.1. Consulta

#### Entrada
| Campo       | Tipo   | Descrição                  | Exemplo           | Obrigatório? |
|-------------|--------|----------------------------|-------------------|--------------|
| idCliente   | String | CPF do cliente             | "987.654.321-09"  | Sim          |
| idEmprestimo| String | Identificador do empréstimo| "EMP-00456"       | Não          |

#### Processo
O sistema busca no banco de dados os dados gerais do contrato (valorEmprestimo, quantidadeParcelas, taxaJurosMensal, etc.) e a tabela de parcelas (numeroParcela, dataVencimento, dataPagamento, valorParcelaOriginal, multaAtraso, jurosMora, valorPago, status). Para parcelas vencidas:
- **Multa:** valorParcelaOriginal * 0.02
- **Juros Mora:** valorParcelaOriginal * 0.000333 * DiasAtraso
- **Total Devido:** valorParcelaOriginal + Multa + JurosMora

Dias de atraso são calculados com base na data atual do sistema no momento da consulta, comparada à dataVencimento.

##### Requisição (Exemplo)
```json
{
  "idCliente": "987.654.321-09",
  "idEmprestimo": "EMP-00456"
}
```

##### Saída
```json
{
  "idCliente": "987.654.321-09",
  "idEmprestimo": "EMP-00456",
  "valorEmprestimo": 15000.00,
  "quantidadeParcelas": 36,
  "dataInicioPagamento": "15/03/2025",
  "statusContrato": "ativo",
  "saldoDevedor": 15678.40,
  "totalPago": 525.50,
  "totalDevido": 18392.50,
  "limiteUtilizado": 525.50,
  "limiteRestante": 974.50,
  "parcelas": [
    {"numeroParcela": 1, "dataVencimento": "15/04/2025", "dataPagamento": "10/04/2025", "valorParcelaOriginal": 525.50, "multaAtraso": 0.00, "jurosMora": 0.00, "valorPago": 525.50, "status": "paga"},
    {"numeroParcela": 2, "dataVencimento": "15/05/2025", "dataPagamento": null, "valorParcelaOriginal": 525.50, "multaAtraso": 0.00, "jurosMora": 0.00, "valorPago": 0.00, "status": "a vencer"}
  ]
}
```

#### Erros
- "Erro: Cliente não encontrado"
- "Erro: Empréstimo não encontrado para o cliente"

### 5.2. Atualização

#### Entrada
| Campo       | Tipo   | Descrição                  | Exemplo           | Obrigatório? |
|-------------|--------|----------------------------|-------------------|--------------|
| idCliente   | String | CPF do cliente             | "987.654.321-09"  | Sim          |
| idEmprestimo| String | Identificador do empréstimo| "EMP-00456"       | Sim          |
| numeroParcela | Inteiro | Parcela a atualizar     | 3                 | Sim          |
| dataPagamento | Data | Data do pagamento          | "20/06/2025"      | Sim          |
| valorPago   | Decimal | Valor pago                | 536.89            | Sim          |

#### Processo
Calcula atrasos, se aplicável:
- **Multa:** valorParcelaOriginal * 0.02
- **Juros Mora:** valorParcelaOriginal * 0.000333 * (dataPagamento - dataVencimento)
- **Total Devido:** valorParcelaOriginal + Multa + JurosMora

Permite pagamentos parciais, ajustando saldoDevedorParcela. Se o valor pago for inferior ao devido, o status permanece "parcialmente paga" até quitação total.

##### Requisição (Exemplo - Pagamento Total)
```json
{
  "idCliente": "987.654.321-09",
  "idEmprestimo": "EMP-00456",
  "numeroParcela": 3,
  "dataPagamento": "20/06/2025",
  "valorPago": 536.89
}
```

##### Saída (Pagamento Total)
```json
{
  "idCliente": "987.654.321-09",
  "idEmprestimo": "EMP-00456",
  "numeroParcela": 3,
  "dataVencimento": "15/06/2025",
  "dataPagamento": "20/06/2025",
  "valorParcelaOriginal": 525.50,
  "multaAtraso": 10.51,
  "jurosMora": 0.88,
  "valorTotalDevido": 536.89,
  "valorPago": 536.89,
  "saldoDevedorParcela": 0.00,
  "status": "paga",
  "mensagem": "Parcela 3 atualizada com sucesso."
}
```

##### Requisição (Exemplo - Pagamento Parcial)
```json
{
  "idCliente": "987.654.321-09",
  "idEmprestimo": "EMP-00456",
  "numeroParcela": 3,
  "dataPagamento": "20/06/2025",
  "valorPago": 400.00
}
```

##### Saída (Pagamento Parcial)
```json
{
  "idCliente": "987.654.321-09",
  "idEmprestimo": "EMP-00456",
  "numeroParcela": 3,
  "dataVencimento": "15/06/2025",
  "dataPagamento": "20/06/2025",
  "valorParcelaOriginal": 525.50,
  "multaAtraso": 10.51,
  "jurosMora": 0.88,
  "valorTotalDevido": 536.89,
  "valorPago": 400.00,
  "saldoDevedorParcela": 136.89,
  "status": "parcialmente paga",
  "mensagem": "Parcela 3 parcialmente paga. Saldo restante: 136.89."
}
```

#### Erros
- "Erro: Parcela já totalmente paga"
- "Erro: Empréstimo não encontrado"

## 6. Antecipação de Parcelas

### 6.1. Entrada

| Campo        | Tipo   | Descrição                   | Exemplo            | Obrigatório? |
|--------------|--------|-----------------------------|--------------------|--------------|
| idCliente    | String | CPF do cliente              | "987.654.321-09"   | Sim          |
| idEmprestimo | String | Identificador do empréstimo | "EMP-00456"        | Sim          |
| parcelas     | Lista  | Parcelas a antecipar        | [2, 3]             | Sim          |

### Processo

Calcula o valor presente das parcelas antecipadas:
```
ValorPresente = valorParcelaOriginal / (1 + TaxaJurosMensal) ^ MesesAntecipados
```
Atualiza o saldoDevedor subtraindo os valores presentes.

### Requisição (Exemplo)

```json
{
  "idCliente": "987.654.321-09",
  "idEmprestimo": "EMP-00456",
  "parcelas": [2, 3]
}
```

### Saída

```json
{
  "idCliente": "987.654.321-09",
  "idEmprestimo": "EMP-00456",
  "parcelasAntecipadas": [
    {"numeroParcela": 2, "valorParcelaOriginal": 525.50, "valorPresente": 516.18, "mesesAntecipados": 1},
    {"numeroParcela": 3, "valorParcelaOriginal": 525.50, "valorPresente": 507.04, "mesesAntecipados": 2}
  ],
  "saldoDevedorAtualizado": 14655.18,
  "totalDesconto": 33.28,
  "mensagem": "Parcelas 2 e 3 antecipadas com sucesso."
}
```

## 7. Observações

- Taxas de juros refletem o perfil de risco do cliente, ajustadas por score, idade e prazo, conforme práticas de mercado.
- Encargos (TAC, IOF, seguro) são incorporados ao CET para transparência.
- Prazos máximos variam por idade: 60 meses (18-65 anos), 48 meses (66-70 anos), 36 meses (71-75 anos).
- Pagamentos parciais e antecipações são flexíveis, permitindo ajustes no saldo devedor.
- Todas as operações podem gerar logs para auditoria, dependendo da implementação do banco de dados.
