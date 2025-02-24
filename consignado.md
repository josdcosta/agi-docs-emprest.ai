# Documentações Emprest.AI 📄
# **EMPRÉSTIMO CONSIGNADO**

## Índice
- [Autores](#autores)
- [Referências](#referencias)
- [Objetivo](#1-objetivo)
- [Visão Geral do Funcionamento](#2-visao-geral-do-funcionamento)
- [Gerenciamento de Clientes](#3-gerenciamento-de-clientes)
  - [Cadastro/Atualização de Cliente](#31-cadastroupdate-cliente)
- [Consulta de Elegibilidade](#4-consulta-de-elegibilidade)
- [Concessão de Empréstimos](#5-concessao-de-emprestimos)
  - [Entrada de Dados](#51-entrada-de-dados)
  - [Processo de Cálculo](#52-processo-de-calculo)
- [Refinanciamento e Portabilidade](#6-refinanciamento-e-portabilidade)
  - [Refinanciamento](#61-refinanciamento)
  - [Portabilidade](#62-portabilidade)
- [Consulta e Atualização de Parcelas](#7-consulta-e-atualizacao-de-parcelas)
  - [Consulta](#71-consulta)
  - [Atualização](#72-atualizacao)
- [Cancelamento de Contrato](#8-cancelamento-de-contrato)
  - [Com Reembolso](#81-com-reembolso)
- [Tabelas Sugeridas e Campos](#tabelas-sugeridas-e-campos)
  - [Tabela: Clientes](#1-tabela-clientes)
  - [Tabela: Empréstimos](#2-tabela-emprestimos)
  - [Tabela: Parcelas](#3-tabela-parcelas)
  - [Tabela: TaxasBase (Opcional)](#4-tabela-taxasbase-opcional)
  - [Tabela: Logs (Opcional)](#5-tabela-logs-opcional)

## Autores
- [@Dalleth Martins](https://github.com/dalleth-martinss)
- [@Josué Davi da Costa](https://github.com/josdcosta)
- [@Carollina Guedes](https://github.com/CarollinaGuedes)
- [@Victor Augusto Ferreira](https://github.com/Victor-augusto-ferreira)


## Referências
- [Planilha de cálculo](https://docs.google.com/spreadsheets/d/1Y_vrP424Qpyh_nWdp_xtSSbsdswpp4XKPIOVeIV9B4E/edit?usp=sharing)
- Leis e Regulamentações: Lei 10.820/2003 (base para consignados), Lei 14.509/2022 (margem consignável de 35%), Regulamentação INSS, Resoluções do Banco Central, Código de Defesa do Consumidor (art. 52, §2º para multa e juros mora).

## 1. Objetivo
O Emprest.AI é um sistema backend desenvolvido para gerenciar de forma eficiente e automatizada todas as etapas relacionadas a empréstimos consignados. Isso inclui o cadastro de clientes, a análise de elegibilidade, a concessão de novos contratos (inclusive refinanciamentos e portabilidades), a consulta de status e parcelas, a atualização de pagamentos e o cancelamento de solicitações. O sistema respeita as normas brasileiras, como a margem consignável de 35% da remuneração líquida (Lei 14.509/2022), o teto de juros de 2,14% ao mês (Banco Central) e o limite de idade final de 80 anos (INSS). As taxas de juros são calculadas com base no vínculo do cliente e na idade, ajustadas dinamicamente pelo prazo escolhido, enquanto os prazos são oferecidos em múltiplos de 12 a partir de 24 meses. O seguro é opcional, e, caso o cliente não especifique a quantidade de parcelas, o sistema sugere opções viáveis.

## 2. Visão Geral do Funcionamento
O sistema é estruturado em cinco áreas principais:
- **Gerenciamento de Clientes:** Permite cadastrar e atualizar informações essenciais, como remuneração e margem consignável.
- **Concessão de Empréstimos:** Processa novos empréstimos, refinanciamentos e portabilidades com cálculos automáticos.
- **Consulta de Empréstimos:** Disponibiliza informações sobre elegibilidade, contratos ativos e parcelas.
- **Atualização de Dados:** Registra pagamentos e ajustes em contratos existentes.
- **Cancelamento de Contrato:** Gerencia a finalização de solicitações antes do início ou com reembolso, se aplicável.

### Regras Principais
- **Margem Consignável:** Calculada como 35% da remuneração líquida menos o valor das parcelas de outros empréstimos ativos, armazenada e obtida diretamente da tabela de clientes no banco de dados.
- **Taxas de Juros:** Determinadas pelo tipo de vínculo e idade do cliente, com um incremento específico por prazo, sempre respeitando o limite de 2,14% ao mês.
- **Prazos:** Oferecidos em múltiplos de 12 (ex.: 24, 36, 48 meses), com mínimo de 24 meses e máximo ajustado para que a idade do cliente ao fim do contrato não ultrapasse 80 anos.
- **Seguro:** Opcional, com custo baseado na idade e no valor do empréstimo.
- **IOF:** Incorporado ao valor financiado, conforme regulamentação fiscal.
- **Carência:** Período padrão de 30 dias entre a solicitação e o primeiro pagamento, ajustável até um máximo de 60 dias.

## 3. Gerenciamento de Clientes

### 3.1. Cadastro/Atualização de Cliente

#### Requisição
```json
{
  "idCliente": "123.456.789-00",
  "nome": "João Silva",
  "remuneracaoLiquida": 5000.00,
  "idade": 75,
  "tipoVinculo": "aposentado",
  "margemConsignavel": 950.00
}
```

#### Processo
Os dados são inseridos ou atualizados na tabela de clientes. A margem consignável é calculada como remuneracaoLiquida * 0.35 - soma das parcelas ativas e armazenada para uso em solicitações futuras, refletindo o limite disponível para novos empréstimos.

#### Saída
```json
{
  "idCliente": "123.456.789-00",
  "nome": "João Silva",
  "remuneracaoLiquida": 5000.00,
  "idade": 75,
  "tipoVinculo": "aposentado",
  "margemConsignavel": 950.00,
  "mensagem": "Cliente cadastrado/atualizado com sucesso."
}
```

## 4. Consulta de Elegibilidade

### 4.1. Requisição
```json
{
  "idCliente": "123.456.789-00"
}
```

### 4.2. Processo
O sistema consulta a tabela de clientes para recuperar remuneração líquida, margem consignável, idade e tipo de vínculo. Com base na idade, calcula o prazo máximo permitido (idade final ≤ 80) e verifica se a margem disponível suporta um empréstimo.

### 4.3. Saída
```json
{
  "idCliente": "123.456.789-00",
  "remuneracaoLiquida": 5000.00,
  "margemConsignavel": 950.00,
  "idade": 75,
  "tipoVinculo": "aposentado",
  "prazoMaximoPermitido": 48,
  "mensagem": "Cliente elegível para empréstimo."
}
```

## 5. Concessão de Empréstimos

### 5.1. Entrada de Dados
| Campo               | Tipo     | Descrição                                     | Exemplo         | Obrigatório? |
|---------------------|----------|-----------------------------------------------|-----------------|--------------|
| idCliente           | String   | CPF do cliente                                | "123.456.789-00"| Sim          |
| valorEmprestimo     | Decimal  | Valor solicitado                              | 10000.00        | Sim          |
| quantidadeParcelas  | Inteiro  | Número de parcelas (múltiplo de 12, ≥ 24)     | 36              | Não          |
| dataInicioPagamento | Data     | Data do primeiro pagamento (futura, até 60 dias)| "01/04/2025"   | Não          |
| contratarSeguro     | Booleano | Opção de contratar seguro                     | true            | Sim          |

#### Dados Adicionais (Refinanciamento ou Portabilidade)
| Campo                | Tipo     | Descrição                                     | Exemplo         | Obrigatório? |
|----------------------|----------|-----------------------------------------------|-----------------|--------------|
| idEmprestimoOriginal | String   | Identificador do empréstimo existente         | "EMP-00123"     | Sim (ref/port)|
| novoValorEmprestimo  | Decimal  | Valor adicional (refinanciamento)             | 2000.00         | Não          |
| novaQuantidadeParcelas| Inteiro | Novo prazo (múltiplo de 12, ≥ 24)             | 48              | Sim (ref/port)|
| bancoDestino         | String   | Banco receptor (portabilidade)                | "BANCOXYZ"      | Sim (port)   |

### 5.2. Processo de Cálculo

#### Consulta Inicial
O sistema valida o idCliente na tabela de clientes, obtendo remuneracaoLiquida, margemConsignavel, idade (atualizada na data atual), e tipoVinculo. Para refinanciamento ou portabilidade, consulta o idEmprestimoOriginal para recuperar o saldo devedor.

#### Validação
- Cliente inexistente: "Erro: Cliente não encontrado".
- Margem insuficiente: "Erro: Margem consignável insuficiente (X.XX)".
- Data inválida: "Erro: Data de início de pagamento inválida".

#### Taxa de Juros
A taxa de juros mensal é calculada com base no tipo de vínculo, na idade e na opção de seguro, ajustada pelo prazo escolhido:

- **Fórmula:** TaxaJurosMensal = TaxaBase + 0.0025 * ((QuantidadeParcelas - 24) / 12)
- **TaxaBase:** Definida por vínculo e idade (ex.: 1,6% para aposentado de 75-78 anos com seguro).
- **Incremento:** Para cada 12 meses acima de 24, a taxa aumenta em 0,0025 (0,25%). Exemplo: 48 meses adiciona 0,005 (0,5%) à taxa base.
- **Teto:** Limitada a 2,14% (0,0214).

#### Taxas Base por Vínculo e Idade
- **Com Seguro:** Aposentado 75-78 anos: 1,6% (0,016), máximo 48 meses.
- **Sem Seguro:** Taxa base + 0,2% (ex.: 1,8%), mesmo prazo.

#### Custo do Seguro, IOF, Valor Total Financiado, Parcela Mensal, CET
Mesmas fórmulas da versão anterior.

### 5.3. Saídas

#### Com quantidadeParcelas Informada

##### Requisição
```json
{
  "idCliente": "123.456.789-00",
  "valorEmprestimo": 10000.00,
  "quantidadeParcelas": 48,
  "dataInicioPagamento": "01/04/2025",
  "contratarSeguro": true
}
```

##### Saída
```json
{
  "idCliente": "123.456.789-00",
  "valorEmprestimo": 10000.00,
  "parcela": 350.13,
  "quantidadeParcelas": 48,
  "dataInicioPagamento": "01/04/2025",
  "dataFimContrato": "01/04/2029",
  "taxaJurosMensal": 0.0170,
  "taxaEfetivaMensal": 0.0192,
  "contratarSeguro": true,
  "custoSeguro": 1150.00,
  "iof": 157.99,
  "valorTotalFinanciado": 11496.87,
  "saldoDevedor": 11496.87,
  "totalPago": 0.00,
  "totalDevido": 16806.24,
  "margemUtilizada": 350.13,
  "margemRestante": 599.87,
  "prazoMaximoPermitido": 48
}
```

#### Sem quantidadeParcelas

##### Requisição
```json
{
  "idCliente": "123.456.789-00",
  "valorEmprestimo": 10000.00,
  "contratarSeguro": true
}
```

##### Saída
```json
{
  "idCliente": "123.456.789-00",
  "valorEmprestimo": 10000.00,
  "contratarSeguro": true,
  "opcoesParcelamento": [
    {"quantidadeParcelas": 24, "parcela": 582.45, "taxaJurosMensal": 0.0165, "valorTotalFinanciado": 11396.87, "totalDevido": 13978.80},
    {"quantidadeParcelas": 36, "parcela": 416.78, "taxaJurosMensal": 0.01675, "valorTotalFinanciado": 11446.87, "totalDevido": 15004.08},
    {"quantidadeParcelas": 48, "parcela": 350.13, "taxaJurosMensal": 0.0170, "valorTotalFinanciado": 11496.87, "totalDevido": 16806.24}
  ]
}
```

## 6. Refinanciamento e Portabilidade

### 6.1. Refinanciamento

#### Requisição
```json
{
  "idCliente": "123.456.789-00",
  "idEmprestimoOriginal": "EMP-00123",
  "novoValorEmprestimo": 2000.00,
  "novaQuantidadeParcelas": 48,
  "dataInicioPagamento": "01/04/2025",
  "contratarSeguro": true
}
```

#### Saída
```json
{
  "idCliente": "123.456.789-00",
  "idEmprestimoOriginal": "EMP-00123",
  "saldoDevedorOriginal": 8000.00,
  "novoValorEmprestimo": 2000.00,
  "parcela": 375.50,
  "novaQuantidadeParcelas": 48,
  "dataInicioPagamento": "01/04/2025",
  "dataFimContrato": "01/04/2029",
  "taxaJurosMensal": 0.0170,
  "valorTotalFinanciado": 11045.60,
  "saldoDevedor": 11045.60,
  "totalDevido": 18024.00,
  "margemUtilizada": 375.50,
  "margemRestante": 574.50
}
```

### 6.2. Portabilidade

#### Requisição
```json
{
  "idCliente": "123.456.789-00",
  "idEmprestimoOriginal": "EMP-00123",
  "bancoDestino": "BANCOXYZ",
  "novaQuantidadeParcelas": 48,
  "dataInicioPagamento": "01/04/2025",
  "contratarSeguro": false
}
```

#### Saída
```json
{
  "idCliente": "123.456.789-00",
  "idEmprestimoOriginal": "EMP-00123",
  "bancoDestino": "BANCOXYZ",
  "saldoDevedorOriginal": 8000.00,
  "parcela": 360.20,
  "novaQuantidadeParcelas": 48,
  "dataInicioPagamento": "01/04/2025",
  "dataFimContrato": "01/04/2029",
  "taxaJurosMensal": 0.0155,
  "valorTotalFinanciado": 8120.00,
  "saldoDevedor": 8120.00,
  "totalDevido": 17289.60,
  "margemUtilizada": 360.20,
  "margemRestante": 589.80,
  "statusContratoOriginal": "portado"
}
```

## 7. Consulta e Atualização de Parcelas

### 7.1. Consulta

#### Requisição
```json
{
  "idCliente": "123.456.789-00",
  "idEmprestimo": "EMP-00123"
}
```

#### Saída
```json
{
  "idCliente": "123.456.789-00",
  "idEmprestimo": "EMP-00123",
  "valorEmprestimo": 10000.00,
  "quantidadeParcelas": 48,
  "dataInicioPagamento": "01/04/2025",
  "statusContrato": "ativo",
  "saldoDevedor": 9650.87,
  "totalPago": 350.13,
  "totalDevido": 16456.11,
  "margemUtilizada": 350.13,
  "margemRestante": 599.87,
  "parcelas": [
    {"numeroParcela": 1, "dataVencimento": "01/05/2025", "dataPagamento": "30/04/2025", "valorParcelaOriginal": 350.13, "multaAtraso": 0.00, "jurosMora": 0.00, "valorPago": 350.13, "status": "paga"},
    {"numeroParcela": 2, "dataVencimento": "01/06/2025", "dataPagamento": null, "valorParcelaOriginal": 350.13, "multaAtraso": 0.00, "jurosMora": 0.00, "valorPago": 0.00, "status": "a vencer"}
  ]
}
```

### 7.2. Atualização

#### Requisição
```json
{
  "idCliente": "123.456.789-00",
  "idEmprestimo": "EMP-00123",
  "numeroParcela": 3,
  "dataPagamento": "15/07/2025",
  "valorPago": 300.00
}
```

#### Saída (Pagamento Parcial)
```json
{
  "idCliente": "123.456.789-00",
  "idEmprestimo": "EMP-00123",
  "numeroParcela": 3,
  "dataVencimento": "01/07/2025",
  "dataPagamento": "15/07/2025",
  "valorParcelaOriginal": 350.13,
  "multaAtraso": 7.00,
  "jurosMora": 1.63,
  "valorTotalDevido": 358.76,
  "valorPago": 300.00,
  "saldoDevedorParcela": 58.76,
  "status": "parcialmente paga",
  "mensagem": "Parcela 3 parcialmente paga. Saldo restante: 58.76."
}
```

## 8. Cancelamento de Contrato

### 8.1. Requisição
```json
{
  "idCliente": "123.456.789-00",
  "idEmprestimo": "EMP-00123"
}
```

### 8.2. Saída (Com Reembolso)
```json
{
  "idCliente": "123.456.789-00",
  "idEmprestimo": "EMP-00123",
  "statusContrato": "cancelado",
  "valorReembolso": 10033.25,
  "mensagem": "Contrato EMP-00123 cancelado. Reembolso devido: 10033.25 (juros proporcionais inclusos)."
}
```

## Observações
- A margem consignável é mantida atualizada na tabela de clientes, refletindo o impacto de contratos ativos.
- O incremento de 0,0025 na taxa de juros por cada 12 meses adicionais acima de 24 reflete o risco associado a prazos mais longos, sempre limitado a 2,14%.
- Todas as operações podem gerar logs para auditoria, dependendo da implementação do banco de dados.
# Tabelas Sugeridas e Campos  (Sem análise ainda de DER)

## 1. Tabela: Clientes
Responsável pelo gerenciamento de dados dos clientes, incluindo informações pessoais e financeiras.

| Campo             | Tipo          | Descrição                                | Exemplo            | Chave?    |
|-------------------|---------------|------------------------------------------|--------------------|-----------|
| idCliente         | VARCHAR(14)   | CPF do cliente                           | "123.456.789-00"   | Primária  |
| nome              | VARCHAR(100)  | Nome completo                            | "João Silva"       | -         |
| remuneracaoLiquida| DECIMAL(10,2) | Renda líquida mensal                     | 5000.00            | -         |
| idade             | INT           | Idade atual do cliente                   | 75                 | -         |
| tipoVinculo       | VARCHAR(20)   | Tipo de vínculo (ex.: "aposentado")      | "aposentado"       | -         |
| margemConsignavel | DECIMAL(10,2) | Margem disponível para novos empréstimos | 950.00             | -         |
| dataAtualizacao   | DATE          | Data da última atualização dos dados     | "2025-02-24"       | -         |

### Notas:
- **margemConsignavel** deve ser recalculada automaticamente sempre que houver alteração em empréstimos ativos (inclusão, pagamento ou cancelamento).
- **idade** pode ser derivada de uma data de nascimento (campo opcional a adicionar, como **dataNascimento**), mas como a documentação usa idade diretamente, mantive assim.

## 2. Tabela: Emprestimos
Armazena informações sobre cada contrato de empréstimo (novo, refinanciamento ou portabilidade).

| Campo                | Tipo         | Descrição                                            | Exemplo          | Chave?     |
|----------------------|--------------|------------------------------------------------------|------------------|------------|
| idEmprestimo         | VARCHAR(20)  | Identificador único do empréstimo                    | "EMP-00123"      | Primária   |
| idCliente            | VARCHAR(14)  | CPF do cliente                                       | "123.456.789-00" | Estrangeira|
| valorEmprestimo      | DECIMAL(12,2)| Valor solicitado                                     | 10000.00         | -          |
| valorTotalFinanciado | DECIMAL(12,2)| Valor total com taxas (IOF, seguro, etc.)            | 11496.87         | -          |
| quantidadeParcelas   | INT          | Número de parcelas                                   | 48               | -          |
| taxaJurosMensal      | DECIMAL(6,5) | Taxa de juros mensal aplicada                        | 0.0170           | -          |
| taxaEfetivaMensal    | DECIMAL(6,5) | Custo efetivo total (CET)                            | 0.0192           | -          |
| contratarSeguro      | BOOLEAN      | Indicador de seguro contratado                       | true             | -          |
| custoSeguro          | DECIMAL(10,2)| Custo do seguro (se aplicável)                       | 1150.00          | -          |
| iof                  | DECIMAL(10,2)| Valor do IOF                                         | 157.99           | -          |
| dataSolicitacao      | DATE         | Data da solicitação                                  | "2025-02-24"     | -          |
| dataInicioPagamento  | DATE         | Data do primeiro pagamento                           | "2025-04-01"     | -          |
| dataFimContrato      | DATE         | Data prevista para o fim do contrato                 | "2029-04-01"     | -          |
| saldoDevedor         | DECIMAL(12,2)| Saldo restante a pagar                               | 11496.87         | -          |
| totalPago            | DECIMAL(12,2)| Total já pago                                        | 0.00             | -          |
| totalDevido          | DECIMAL(12,2)| Valor total a ser pago (parcela * n)                 | 16806.24         | -          |
| margemUtilizada      | DECIMAL(10,2)| Margem consumida por este empréstimo                 | 350.13           | -          |
| statusContrato       | VARCHAR(20)  | Status (ativo, cancelado, portado, etc.)             | "ativo"          | -          |
| idEmprestimoOriginal | VARCHAR(20)  | ID do empréstimo original (ref/port)                 | "EMP-00099"      | Estrangeira (opcional) |
| bancoDestino         | VARCHAR(50)  | Banco receptor (portabilidade)                       | "BANCOXYZ"       | - (opcional) |

### Notas:
- **idEmprestimoOriginal** é necessário para refinanciamentos e portabilidades, referenciando outro registro nesta mesma tabela.
- **margemUtilizada** reflete o valor da parcela mensal e deve atualizar a **margemConsignavel** na tabela Clientes.
- **statusContrato** suporta os estados "ativo", "cancelado", "portado", etc., conforme descrito.

## 3. Tabela: Parcelas
Registra as parcelas de cada empréstimo, incluindo pagamentos e atrasos.

| Campo                | Tipo         | Descrição                                        | Exemplo          | Chave?     |
|----------------------|--------------|--------------------------------------------------|------------------|------------|
| idParcela            | VARCHAR(25)  | Identificador único da parcela                   | "PAR-00123-01"   | Primária   |
| idEmprestimo         | VARCHAR(20)  | ID do empréstimo relacionado                     | "EMP-00123"      | Estrangeira|
| numeroParcela        | INT          | Número da parcela no contrato                    | 1                | -          |
| dataVencimento       | DATE         | Data de vencimento da parcela                    | "2025-05-01"     | -          |
| dataPagamento        | DATE         | Data do pagamento (se pago)                      | "2025-04-30"     | -          |
| valorParcelaOriginal | DECIMAL(10,2)| Valor original da parcela                        | 350.13           | -          |
| multaAtraso          | DECIMAL(10,2)| Multa por atraso (se aplicável)                  | 7.00             | -          |
| jurosMora            | DECIMAL(10,2)| Juros de mora por atraso                         | 1.63             | -          |
| valorPago            | DECIMAL(10,2)| Valor efetivamente pago                          | 350.13           | -          |
| saldoDevedorParcela  | DECIMAL(10,2)| Saldo restante da parcela (se parcial)           | 0.00             | -          |
| status               | VARCHAR(20)  | Status (paga, a vencer, parcialmente paga)       | "paga"           | -          |

### Notas:
- **idParcela** pode ser composto (ex.: **idEmprestimo** + **numeroParcela**) ou um identificador único.
- Campos como **multaAtraso** e **jurosMora** permitem lidar com atrasos, conforme exemplo na atualização de parcelas.

## 4. Tabela: TaxasBase (Opcional)
Armazena as taxas base por vínculo e faixa etária para cálculos dinâmicos.

| Campo                | Tipo         | Descrição                                        | Exemplo          | Chave?     |
|----------------------|--------------|--------------------------------------------------|------------------|------------|
| tipoVinculo          | VARCHAR(20)  | Tipo de vínculo                                  | "aposentado"     | Primária (composta) |
| idadeMin             | INT          | Faixa de idade mínima                            | 75               | Primária (composta) |
| idadeMax             | INT          | Faixa de idade máxima                            | 78               | Primária (composta) |
| taxaBaseComSeguro    | DECIMAL(6,5) | Taxa base com seguro                             | 0.016            | -          |
| taxaBaseSemSeguro    | DECIMAL(6,5) | Taxa base sem seguro                             | 0.018            | -          |
| prazoMaximo          | INT          | Prazo máximo em meses                            | 48               | -          |

### Notas:
- Essa tabela é opcional, mas facilita a manutenção das taxas base e prazos máximos mencionados na documentação (ex.: aposentado 75-78 anos, taxa 1,6%, prazo 48 meses).
- Pode ser substituída por lógica no código, mas um banco de dados relacional oferece maior flexibilidade.

## 5. Tabela: Logs (Opcional)
Para auditoria de operações, como sugerido nas observações.

| Campo                | Tipo         | Descrição                                        | Exemplo          | Chave?     |
|----------------------|--------------|--------------------------------------------------|------------------|------------|
| idLog                | BIGINT       | Identificador único do log                       | 1                | Primária   |
| idCliente            | VARCHAR(14)  | CPF do cliente                                   | "123.456.789-00" | -          |
| idEmprestimo         | VARCHAR(20)  | ID do empréstimo (se aplicável)                  | "EMP-00123"      | -          |
| operacao             | VARCHAR(50)  | Tipo de operação (cadastro, pagamento, etc.)     | "concessao"      | -          |
| dataOperacao         | DATETIME     | Data e hora da operação                          | "2025-02-24 10:00"| -          |
| detalhes             | TEXT         | Informações adicionais                           | "Empréstimo concedido" | -          |
