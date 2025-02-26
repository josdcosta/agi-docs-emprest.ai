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

### 1. Objetivo
O Emprest.AI é um backend projetado para gerenciar de forma eficiente e transparente o ciclo completo de empréstimos, abrangendo as modalidades Empréstimo Pessoal e Empréstimo Consignado. Suas funcionalidades incluem concessão de novos contratos, simulação de condições, consulta de dados, pagamento antecipado (total ou parcial), refinanciamento (quando aplicável), portabilidade (para consignado) e cancelamento, com critérios adaptados a cada modalidade.

### 2. Variáveis
Os parâmetros abaixo do sistema Emprest.AI:

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
| margemConsignavel          | Percentual da remuneração líquida para margem             | 35%                     |
| iof                        | Imposto sobre Operações Financeiras                       | Conforme legislação     |
| percentualRendaPessoal     | Percentual máximo da renda líquida para parcela (Empréstimo Pessoal)  | 30%            |
| percentualMinimoRefinanciamento | Percentual mínimo de parcelas pagas para refinanciamento | 20%                     |
### 3. Visão Geral do Funcionamento
O sistema é estruturado em áreas principais, aplicáveis a ambas as modalidades com ajustes específicos:

- Concessão de Empréstimos: Análise de crédito adaptada (Consignado: margem consignável; Pessoal: score e renda). Simulação e aprovação de contratos.
- Consulta de Empréstimos: Acompanhamento de status, parcelas e histórico de pagamentos.
- Pagamento de Empréstimos: Registro de pagamentos (totais ou parciais), incluindo antecipações.
- Refinanciamento: Renegociação de contratos existentes (ambas as modalidades).
- Portabilidade: Transferência de consignados para outro banco.
- Cancelamento de Contrato: Gestão de desistências ou finalizações.

### 4. Dados Armazenados do Cliente
```json
{
  "idCliente": "123.456.789-00",
  "nome": "João Silva",
  "remuneracaoLiquidaMensal": 5000.00,
  "idade": 75,
  "tipoVinculo": "aposentado",
  "scoreCredito": 600
}
```

### 5. Simulação de Empréstimo
#### 5.1. Requisição - Sistema recebe informações do usúario identificando o tipo de modalidade.
##### Empréstimo Consignado - Aposentado, pensionistas, funcionários públicos
```json
{
  "idCliente": "123.456.789-00",
  "valorEmprestimo": 10000.00,
  "tipoEmprestimo": "consignado",
  "quantidadeParcelas": 48,
  "contratarSeguro": true,
  "dataInicioPagamento": "01/04/2025"
}
```
##### Empréstimo Pessoal
```json
{
  "idCliente": "123.456.789-00",
  "valorEmprestimo": 5000.00,
  "tipoEmprestimo": "pessoal",
  "quantidadeParcelas": 18,
  "contratarSeguro": false,
  "dataInicioPagamento": "01/04/2025"
}
```
#### 5.2. Processo Passo a Passo
**Passo 1: Consulta de Dados do Cliente**  
O sistema busca o `idCliente` na base e retorna `remuneracaoLiquidaMensal`, `idade`, `tipoVinculo` (para consignado) e `scoreCredito` (para pessoal). Se não encontrado, "Erro: Cliente não encontrado".

**Passo 2: Verificação Inicial de Elegibilidade**  
Empréstimo Consignado:
- Aplica [11.1.5. Tipo de Vínculo](#1115-tipo-de-vínculo).
- Usa [11.1.2. Idade Máxima](#1112-idade-máxima) para calcular idade + quantidadeParcelas / 12.
- Aplica [11.1.3. Quantidade de Parcelas](#1113-quantidade-de-parcelas).
- Calcula dias de carência e aplica [11.1.6. Carência](#1116-carência).  

Empréstimo Pessoal:
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
##### Empréstimo Consignado
```json
{
  "idCliente": "123.456.789-00",
  "valorEmprestimo": 10000.00,
  "tipoEmprestimo": "consignado",
  "quantidadeParcelas": 48,
  "taxaJurosMensal": 0.0192,
  "custoSeguro": 250.00,
  "iof": 337.30,
  "valorTotalFinanciado": 10809.47,
  "parcelaMensal": 306.28,
  "cetMensal": 0.0200,
  "tabelaParcelas": [
    {
      "numeroParcela": 1,
      "dataVencimento": "01/04/2025",
      "valorParcela": 306.28,
      "juros": 207.54,
      "amortizacao": 98.74,
      "saldoDevedor": 10710.73
    },
    {
      "numeroParcela": 2,
      "dataVencimento": "01/05/2025",
      "valorParcela": 306.28,
      "juros": 205.65,
      "amortizacao": 100.63,
      "saldoDevedor": 10610.10
    },
    // ... (continua até a parcela 48)
    {
      "numeroParcela": 48,
      "dataVencimento": "01/03/2029",
      "valorParcela": 306.28,
      "juros": 5.85,
      "amortizacao": 300.43,
      "saldoDevedor": 0.00
    }
  ],
  "mensagem": "Simulação realizada com sucesso."
}
```
##### Empréstimo Pessoal
```json
{
  "idCliente": "123.456.789-00",
  "valorEmprestimo": 5000.00,
  "tipoEmprestimo": "pessoal",
  "quantidadeParcelas": 18,
  "taxaJurosMensal": 0.0924,
  "custoSeguro": 0.00,
  "iof": 168.65,
  "valorTotalFinanciado": 5486.91,
  "parcelaMensal": 639.99,
  "cetMensal": 0.0940,
  "tabelaParcelas": [
    {
      "numeroParcela": 1,
      "dataVencimento": "01/04/2025",
      "valorParcela": 639.99,
      "juros": 507.00,
      "amortizacao": 132.99,
      "saldoDevedor": 5353.92,
      "status": "pendente",
      "dataPagamento": null
    },
    {
      "numeroParcela": 2,
      "dataVencimento": "01/05/2025",
      "valorParcela": 639.99,
      "juros": 494.70,
      "amortizacao": 145.29,
      "saldoDevedor": 5208.63,
      "status": "pendente",
      "dataPagamento": null
    },
    // ... (continua até a parcela 18)
    {
      "numeroParcela": 18,
      "dataVencimento": "01/09/2026",
      "valorParcela": 639.99,
      "juros": 8.09,
      "amortizacao": 631.90,
      "saldoDevedor": 0.00,
      "status": "pendente",
      "dataPagamento": null
    }
  ],
  "mensagem": "Empréstimo concedido com sucesso."
}
```

### 6. Concessão de Empréstimo
#### 6.1. Requisição
Empréstimo Consignado
```json
{
  "idCliente": "123.456.789-00",
  "valorEmprestimo": 10000.00,
  "tipoEmprestimo": "consignado",
  "quantidadeParcelas": 48,
  "contratarSeguro": true,
  "dataInicioPagamento": "01/04/2025"
}
```

Empréstimo Pessoal
```json
{
  "idCliente": "123.456.789-00",
  "valorEmprestimo": 5000.00,
  "tipoEmprestimo": "pessoal",
  "quantidadeParcelas": 18,
  "contratarSeguro": false,
  "dataInicioPagamento": "01/04/2025"
}
```

#### 6.2. Processo Passo a Passo
- Consulta de Dados do Cliente: Mesmo que Simulação.
- Verificação Inicial de Elegibilidade: Mesmo que Simulação.
- Determinação da Capacidade de Pagamento: Mesmo que Simulação.
- Definição da Taxa de Juros: Mesmo que Simulação.
- Cálculo do Custo do Seguro: Mesmo que Simulação.
- Cálculo do IOF: Mesmo que Simulação.
- Cálculo do Valor Total Financiado: Mesmo que Simulação.
- Cálculo da Parcela Mensal: Mesmo que Simulação.
- Validação Final de Elegibilidade: Mesmo que Simulação.
- Gera a tabela de parcelas para o contrato: Idêntico ao passo da simulação, gera a tabela de parcelas para o contrato.
- Registro do Contrato: Cria o contrato e associa o pagamento (folha para consignado, débito automático para pessoal).

#### 6.3. Saída
(Idêntica à Simulação, com "mensagem": "Empréstimo concedido com sucesso.")

### 7. Consulta de Dados de Empréstimo
#### 7.1. Requisição
```json
{
  "idCliente": "123.456.789-00",
  "idEmprestimo": "EMP001"
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
  "idCliente": "123.456.789-00",
  "idEmprestimo": "EMP001",
  "valorEmprestimo": 10000.00,
  "quantidadeParcelas": 48,
  "taxaJurosMensal": 0.0192,
  "custoSeguro": 250.00,
  "iof": 337.30,
  "valorTotalFinanciado": 10809.47,
  "parcelaMensal": 306.28,
  "totalParcelasPagas": 5,
  "totalParcelasRestantes": 43,
  "saldoDevedor": 9278.07,
  "tabelaParcelas": [
    {
      "numeroParcela": 1,
      "dataVencimento": "01/04/2025",
      "valorParcela": 306.28,
      "juros": 207.54,
      "amortizacao": 98.74,
      "saldoDevedor": 10710.73,
      "status": "paga",
      "dataPagamento": "01/04/2025"
    },
    {
      "numeroParcela": 2,
      "dataVencimento": "01/05/2025",
      "valorParcela": 306.28,
      "juros": 205.65,
      "amortizacao": 100.63,
      "saldoDevedor": 10610.10,
      "status": "paga",
      "dataPagamento": "01/05/2025"
    },
    // ... (parcelas 3 a 5 também "paga")
    {
      "numeroParcela": 6,
      "dataVencimento": "01/09/2025",
      "valorParcela": 306.28,
      "juros": 198.00,
      "amortizacao": 108.28,
      "saldoDevedor": 9278.07,
      "status": "pendente",
      "dataPagamento": null
    },
    // ... (continua até a parcela 48)
    {
      "numeroParcela": 48,
      "dataVencimento": "01/03/2029",
      "valorParcela": 306.28,
      "juros": 5.85,
      "amortizacao": 300.43,
      "saldoDevedor": 0.00,
      "status": "pendente",
      "dataPagamento": null
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
  "idCliente": "123.456.789-00",
  "idEmprestimo": "EMP001",
  "valorPagamento": 306.28,
  "numeroParcela": 6,
  "dataPagamento": "26/02/2025"
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
  "idCliente": "123.456.789-00",
  "idEmprestimo": "EMP001",
  "valorPagamento": 313.41,  // Ajustado para cobrir multa e juros de mora
  "numeroParcela": 6,
  "saldoDevedor": 9278.07,
  "tabelaParcelas": [
    {
      "numeroParcela": 1,
      "dataVencimento": "01/04/2025",
      "valorParcela": 306.28,
      "juros": 207.54,
      "amortizacao": 98.74,
      "saldoDevedor": 10710.73,
      "status": "paga",
      "dataPagamento": "01/04/2025"
    },
    // ... (parcelas 2 a 5 já pagas)
    {
      "numeroParcela": 6,
      "dataVencimento": "01/09/2025",
      "valorParcela": 306.28,
      "multa": 6.12,  // 2% de 306.28
      "jurosMora": 1.01,  // 0,033% ao dia por 10 dias
      "valorTotalAjustado": 313.41,  // 306.28 + 6.12 + 1.01
      "juros": 198.00,
      "amortizacao": 108.28,
      "saldoDevedor": 9278.07,
      "status": "paga",
      "dataPagamento": "11/09/2025"  // Atraso de 10 dias
    },
    {
      "numeroParcela": 7,
      "dataVencimento": "01/10/2025",
      "valorParcela": 306.28,
      "juros": 195.92,
      "amortizacao": 110.36,
      "saldoDevedor": 9167.71,
      "status": "pendente",
      "dataPagamento": null
    },
    // ... (continua até a parcela 48)
    {
      "numeroParcela": 48,
      "dataVencimento": "01/03/2029",
      "valorParcela": 306.28,
      "juros": 5.85,
      "amortizacao": 300.43,
      "saldoDevedor": 0.00,
      "status": "pendente",
      "dataPagamento": null
    }
  ],
  "mensagem": "Pagamento da parcela 6 registrado com sucesso, com multa e juros de mora aplicados."
}
```

#### 8.4. Tratamento de Atrasos
- Se uma parcela não for paga até a `dataVencimento`, aplica-se:
  - **Multa**: 2% sobre o valor da parcela (limite do Código de Defesa do Consumidor, art. 52, §2º).
  - **Juros de Mora**: 1% ao mês (0,033% ao dia) sobre o valor da parcela, proporcional aos dias de atraso.
- Exemplo: Parcela de R$ 306,28 vencida em 01/09/2025, paga em 11/09/2025 (10 dias de atraso):
  - Multa: `306,28 * 0,02 = 6,12`.
  - Juros de mora: `306,28 * 0,00033 * 10 = 1,01`.
  - Total a pagar: `306,28 + 6,12 + 1,01 = 313,41`.

## 9. Refinanciamento de Empréstimo
## 9.1. Requisição

```json
{
  "idCliente": "123.456.789-00",
  "idEmprestimoOriginal": "EMP001",
  "novoValorEmprestimo": 2000.00,
  "novaQuantidadeParcelas": 60,
  "contratarSeguro": true,
  "dataInicioPagamento": "01/04/2025"
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
   - Aplica 11.3.1. Percentual Mínimo Pago.
   - Consignado: 11.1.2. Idade Máxima, 11.1.3. Quantidade de Parcelas, 11.1.6. Carência.
   - Pessoal: 11.2.2. Quantidade de Parcelas, 11.2.5. Carência.

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
    - Consignado: 11.1.1. Margem Consignável.
    - Pessoal: 11.2.4. Capacidade de Pagamento.

14. **Registro do Refinanciamento:**
    - Cria novo contrato com a tabela de parcelas e marca o original como "refinanciado".

## 9.3. Saída

```json
{
  "idCliente": "123.456.789-00",
  "idEmprestimoOriginal": "EMP001",
  "idNovoEmprestimo": "EMP002",
  "saldoDevedorOriginal": 9278.07,
  "novoValorEmprestimo": 2000.00,
  "novaQuantidadeParcelas": 60,
  "taxaJurosMensal": 0.0198,
  "custoSeguro": 312.50,
  "iof": 438.70,
  "valorTotalFinanciado": 14029.27,
  "parcelaMensal": 353.45,
  "tabelaParcelas": [
    {
      "numeroParcela": 1,
      "dataVencimento": "01/04/2025",
      "valorParcela": 353.45,
      "juros": 277.78,
      "amortizacao": 75.67,
      "saldoDevedor": 13953.60,
      "status": "pendente",
      "dataPagamento": null
    },
    {
      "numeroParcela": 2,
      "dataVencimento": "01/05/2025",
      "valorParcela": 353.45,
      "juros": 276.28,
      "amortizacao": 77.17,
      "saldoDevedor": 13876.43,
      "status": "pendente",
      "dataPagamento": null
    },
    // ... (continua até a parcela 60)
    {
      "numeroParcela": 60,
      "dataVencimento": "01/03/2030",
      "valorParcela": 353.45,
      "juros": 6.99,
      "amortizacao": 346.46,
      "saldoDevedor": 0.00,
      "status": "pendente",
      "dataPagamento": null
    }
  ],
  "mensagem": "Refinanciamento realizado com sucesso."
}
```

# 11. Elegibilidade

## 11.1. Empréstimo Consignado

### 11.1.1. Margem Consignável

Parcela ≤ (remuneracaoLiquida * 0,35) - soma de parcelas ativas.

### 11.1.2. Idade Máxima

idade + quantidadeParcelas / 12 ≤ 80 anos.

### 11.1.3. Quantidade de Parcelas

Entre 24 e 92 parcelas.

### 11.1.4. Taxa de Juros

Taxa mensal ≤ 2,14%.

### 11.1.5. Tipo de Vínculo

"Aposentado", "servidor público" ou outro válido.

### 11.1.6. Carência

Dias até o primeiro pagamento ≤ 60.

## 11.2. Empréstimo Pessoal

### 11.2.1. Valor do Empréstimo

valorMinimoPessoal ≤ valorEmprestimo ≤ valorMaximoPessoal.

### 11.2.2. Quantidade de Parcelas

prazoMinimoPessoal ≤ quantidadeParcelas ≤ prazoMaximoPessoal, conforme score:
- 201-400: 6-12.
- 401-600: 6-18.
- 601-800: 6-24.
- 801-1000: 6-30.

### 11.2.3. Score de Crédito

scoreCredito ≥ 201.

### 11.2.4. Capacidade de Pagamento

Parcela ≤ percentualRendaPessoal * remuneracaoLiquida.

### 11.2.5. Carência

Dias até o primeiro pagamento ≤ 30.

## 11.3. Refinanciamento (Comum)

### 11.3.1. Percentual Mínimo Pago

≥ 20% das parcelas pagas.

## 11.4. Portabilidade (Empréstimo Consignado)

### 11.4.1. Parcelas em Dia

Sem parcelas vencidas.

### 11.4.2. Aceitação do Banco Destino

bancoDestino deve aceitar a portabilidade.

# 12. Cálculos

## 12.1. Margem Consignável (Empréstimo Consignado)

margemMaxima = remuneracaoLiquida * margemConsignavel

## 12.2. Capacidade de Pagamento (Empréstimo Pessoal)

capacidadeMaxima = remuneracaoLiquida * percentualRendaPessoal

## 12.3. Taxa de Juros Mensal

- Consignado: TaxaJurosMensal = 0,018 + 0,00005 * (quantidadeParcelas - 24), limitada a 2,14%.
- Pessoal: Interpolação entre jurosMinimoPessoal (8,49%) e jurosMaximoPessoal (9,99%) com base em scoreCredito.

## 12.4. Custo do Seguro

CustoSeguro = valorBase * (0,0025 + 0,00005 * idade) * (quantidadeParcelas / 12)

## 12.5. IOF

IOF = (0,0038 * valorBase) + (0,000082 * valorBase * min(diasFinanciamento, 365))

## 12.6. Valor Total Financiado

ValorInicial = valorBase + IOF + CustoSeguro
ValorTotalFinanciado = ValorInicial * (1 + TaxaJurosMensal / 30) ^ diasCarencia

## 12.7. Parcela Mensal

ParcelaMensal = [ValorTotalFinanciado * TaxaJurosMensal] / [1 - (1 + TaxaJurosMensal)^(-quantidadeParcelas)]

## 12.8. Saldo Devedor

SaldoDevedor = ValorTotalFinanciado * [(1 + TaxaJurosMensal)^quantidadeParcelasRestantes - 1] / [(1 + TaxaJurosMensal)^quantidadeParcelasTotais - 1]
