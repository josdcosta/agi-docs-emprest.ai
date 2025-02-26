# Documentações Emprest.AI 📄

## EMPRÉSTIMO EMPRESARIAL

### Índice
1. [Autores](#autores)
2. [Referências](#referências)
3. [Objetivo](#1-objetivo)
4. [Variáveis Configuráveis](#2-variáveis-configuráveis)
5. [Visão Geral do Funcionamento](#3-visão-geral-do-funcionamento)
6. [Dados Armazenados do Cliente](#4-dados-armazenados-do-cliente)
7. [Simulação de Empréstimo](#5-simulação-de-empréstimo)
8. [Concessão de Empréstimo](#6-concessão-de-empréstimo)
9. [Consulta de Dados de Empréstimo](#7-consulta-de-dados-de-empréstimo)
10. [Pagamento de Empréstimo](#8-pagamento-de-empréstimo)
11. [Refinanciamento de Empréstimo](#9-refinanciamento-de-empréstimo)
12. [Portabilidade de Empréstimo](#10-portabilidade-de-empréstimo)
13. [Elegibilidade](#11-elegibilidade)
14. [Cálculos](#12-cálculos)

## Autores
- @Dalleth Martins
- @João Pedro de C. Formigari
- @Josué Davi da Costa
- @Carollina Guedes
- @Victor Augusto Ferreira

## Referências
- [Planilha de cálculo](https://docs.google.com/spreadsheets/d/1Y_vrP424Qpyh_nWdp_xtSSbsdswpp4XKPIOVeIV9B4E/edit?usp=sharing)
- Regulamentações: Resoluções do Banco Central, Código de Defesa do Consumidor (art. 52, §2º para multa e juros mora), práticas de mercado para taxas de juros empresariais.

## 1. Objetivo
O Emprest.AI é um backend projetado para gerenciar de forma eficiente e transparente o ciclo completo de empréstimos empresariais. Suas funcionalidades abrangem a simulação de condições, concessão de novos contratos, consulta de dados, pagamento (total ou parcial), refinanciamento e cancelamento, considerando o porte da empresa, faturamento líquido anual e análise de risco. As taxas de juros variam conforme o porte e o prazo, utilizando o Sistema de Amortização Constante (SAC), com prazos entre 12 e 120 meses.

## 2. Variáveis Configuráveis
Os parâmetros abaixo são aplicáveis ao sistema Emprest.AI para empréstimos empresariais:

| Variável                  | Descrição                                                 | Valor Padrão            |
|---------------------------|-----------------------------------------------------------|-------------------------|
| jurosMinimoMicro          | Taxa mínima de juros mensal (microempresa, com seguro)    | 1,80% ao mês            |
| jurosMinimoPequena        | Taxa mínima de juros mensal (pequena empresa, com seguro) | 1,60% ao mês            |
| jurosMinimoMedia          | Taxa mínima de juros mensal (média empresa, com seguro)   | 1,40% ao mês            |
| jurosMinimoGrande         | Taxa mínima de juros mensal (grande empresa, com seguro)  | 1,20% ao mês            |
| acrescimoSemSeguro        | Acréscimo na taxa base sem seguro                         | 0,30% ao mês            |
| incrementoPrazo           | Incremento na taxa por cada 12 meses acima de 12          | 0,50% ao mês            |
| valorMinimoEmpresarial    | Valor mínimo do empréstimo empresarial                    | R$ 5.000,00             |
| valorMaximoEmpresarial    | Valor máximo do empréstimo empresarial                    | R$ 5.000.000,00         |
| prazoMinimoEmpresarial    | Prazo mínimo em parcelas                                  | 12 parcelas             |
| prazoMaximoMicro          | Prazo máximo (microempresa)                               | 48 parcelas             |
| prazoMaximoPequena        | Prazo máximo (pequena empresa)                            | 72 parcelas             |
| prazoMaximoMedia          | Prazo máximo (média empresa)                              | 96 parcelas             |
| prazoMaximoGrande         | Prazo máximo (grande empresa)                             | 120 parcelas            |
| carenciaMaximaEmpresarial | Período máximo de carência                                | 90 dias                 |
| percentualFaturamento     | Percentual do faturamento líquido anual para capacidade   | 20%                     |
| percentualMinimoRefinanciamento | Percentual mínimo de parcelas pagas para refinanciamento | 20%                     |
| taxaSeguro                | Taxa fixa do seguro sobre o valor do empréstimo           | 5%                      |
| iofFixo                   | Alíquota fixa do IOF                                      | 0,38%                   |
| iofVariavel               | Alíquota variável do IOF por dia (máximo 365 dias)        | 0,041%                  |

## 3. Visão Geral do Funcionamento
O sistema é estruturado em áreas principais específicas para empréstimos empresariais:

- **Concessão de Empréstimos**: Análise de crédito baseada em faturamento líquido anual, porte da empresa e dívidas existentes. Simulação e aprovação de contratos via SAC.
- **Consulta de Empréstimos**: Acompanhamento de status, parcelas e histórico de pagamentos.
- **Pagamento de Empréstimos**: Registro de pagamentos (totais ou parciais), incluindo antecipações.
- **Refinanciamento**: Renegociação de contratos existentes com ajuste de taxas e prazos.
- **Cancelamento de Contrato**: Gestão de desistências ou finalizações.
- **Simulação de Opções**: Retorno de alternativas de parcelamento se o prazo não for especificado.

## 4. Dados Armazenados do Cliente
```json
{
  "idEmpresa": "12.345.678/0001-90",
  "razaoSocial": "Empresa Exemplo Ltda",
  "faturamentoLiquidoAnual": 600000.00,
  "porteEmpresa": "grande",
  "dividasExistentes": 5000.00,
  "scoreCredito": 700
}
```
## 5. Simulação de Empréstimo
### 5.1. Requisição
```json
{
  "idEmpresa": "12.345.678/0001-90",
  "valorEmprestimo": 50000.00,
  "quantidadeParcelas": 24,
  "contratarSeguro": true,
  "dataInicioPagamento": "01/04/2025"
}
```
### 5.2. Processo Passo a Passo
- Passo 1: Consulta de Dados do Cliente
  - O sistema busca o `idEmpresa` na base e retorna `faturamentoLiquidoAnual`, `porteEmpresa` e `dividasExistentes`. Se não encontrado, "Erro: Empresa não encontrada".

- Passo 2: Verificação Inicial de Elegibilidade
  - Aplica 11.1.1. Valor do Empréstimo.
  - Aplica 11.1.2. Quantidade de Parcelas conforme porte.
  - Calcula dias de carência e aplica 11.1.5. Carência.

- Passo 3: Determinação da Capacidade de Pagamento
  - Executa 12.1. Capacidade de Pagamento.

- Passo 4: Definição da Taxa de Juros
  - Aplica 12.2. Taxa de Juros Mensal com base em `porteEmpresa` e `quantidadeParcelas`.

- Passo 5: Cálculo do Custo do Seguro
  - Se `contratarSeguro = true`, aplica 12.3. Custo do Seguro.

- Passo 6: Cálculo do IOF
  - Executa 12.4. IOF.

- Passo 7: Cálculo do Valor Total Financiado
  - Aplica 12.5. Valor Total Financiado considerando carência.

- Passo 8: Cálculo da Parcela Mensal (SAC)
  - Executa 12.6. Parcela Mensal com amortização constante.

- Passo 9: Validação Final de Elegibilidade
  - Aplica 11.1.3. Capacidade de Pagamento (valida primeira parcela).

- Passo 10: Retorno da Simulação
  - Retorna os valores calculados sem gravar o contrato.

### 5.3. Saída
```json
{
  "idEmpresa": "12.345.678/0001-90",
  "valorEmprestimo": 50000.00,
  "quantidadeParcelas": 24,
  "taxaJurosMensal": 0.017,
  "custoSeguro": 2500.00,
  "iof": 789.95,
  "valorTotalFinanciado": 54094.41,
  "primeiraParcela": 3173.53,
  "ultimaParcela": 2283.93,
  "cetMensal": 0.0185,
  "mensagem": "Simulação realizada com sucesso."
}
```

## 6. Concessão de Empréstimo
### 6.1. Requisição
```json
{
  "idEmpresa": "12.345.678/0001-90",
  "valorEmprestimo": 50000.00,
  "quantidadeParcelas": 24,
  "contratarSeguro": true,
  "dataInicioPagamento": "01/04/2025"
}
```
### 6.2. Processo Passo a Passo
- Consulta de Dados do Cliente: Mesmo que Simulação.
- Verificação Inicial de Elegibilidade: Mesmo que Simulação.
- Determinação da Capacidade de Pagamento: Mesmo que Simulação.
- Definição da Taxa de Juros: Mesmo que Simulação.
- Cálculo do Custo do Seguro: Mesmo que Simulação.
- Cálculo do IOF: Mesmo que Simulação.
- Cálculo do Valor Total Financiado: Mesmo que Simulação.
- Cálculo da Parcela Mensal: Mesmo que Simulação.
- Validação Final de Elegibilidade: Mesmo que Simulação.
- Análise de Score de Crédito: Veja seção 6.3.
- Registro do Contrato: Cria o contrato e associa o pagamento (ex.: débito automático) apenas se o Score atender aos critérios de aprovação.

### 6.3 Aprovação Baseada em Score de Crédito
- O sistema integra-se a um serviço externo de análise de crédito para determinar a aprovação do empréstimo com base no Score do cliente. O processo é o seguinte:

- Envio de Dados do Cliente:
  - O sistema envia os dados armazenados do cliente (ex.: `idEmpresa`, `razaoSocial`, `faturamentoLiquidoAnual`, `porteEmpresa`, `dividasExistentes`) ao serviço externo via uma API segura.
  - Exemplo de requisição ao serviço externo:
```json
{
  "idEmpresa": "12.345.678/0001-90",
  "razaoSocial": "Empresa Exemplo Ltda",
  "faturamentoLiquidoAnual": 600000.00,
  "porteEmpresa": "grande",
  "dividasExistentes": 5000.00
}
```
- Retorno do Score:
  - O serviço externo retorna um valor de Score em uma escala de 0 a 1000, onde Scores mais altos indicam menor risco de inadimplência.
  - Exemplo de resposta:
```json
{
  "idEmpresa": "12.345.678/0001-90",
  "scoreCredito": 720
}
```
- Critérios de Aprovação:
 - O sistema define um limite mínimo de Score para aprovação, configurável conforme o porte da empresa:
   - Microempresa: Score ≥ 600
   - Pequena empresa: Score ≥ 650
   - Média empresa: Score ≥ 700
   - Grande empresa: Score ≥ 750
 - Se o Score retornado for inferior ao limite correspondente ao `porteEmpresa`, o empréstimo é automaticamente rejeitado.

- Registro do Resultado:
  - O Score e o resultado da análise (aprovado ou rejeitado) são armazenados no sistema para auditoria e consulta futura.

### 6.4. Saída
- Empréstimo Aprovado:
```json
{
  "idEmpresa": "12.345.678/0001-90",
  "valorEmprestimo": 50000.00,
  "quantidadeParcelas": 24,
  "taxaJurosMensal": 0.017,
  "custoSeguro": 2500.00,
  "iof": 789.95,
  "valorTotalFinanciado": 54094.41,
  "primeiraParcela": 3173.53,
  "ultimaParcela": 2283.93,
  "cetMensal": 0.0185,
  "scoreCredito": 720,
  "mensagem": "Empréstimo concedido com sucesso."
}
```
- Empréstimo Rejeitado:
```json
{
  "idEmpresa": "12.345.678/0001-90",
  "valorEmprestimo": 50000.00,
  "quantidadeParcelas": 24,
  "scoreCredito": 620,
  "mensagem": "Empréstimo rejeitado devido a Score de crédito insuficiente."
}
```

## 7. Consulta de Dados de Empréstimo
### 7.1. Requisição
```json
{
  "idEmpresa": "12.345.678/0001-90",
  "idEmprestimo": "EME001"
}
```
### 7.2. Processo Passo a Passo
- Consulta de Dados do Cliente: Valida `idEmpresa`. Se não encontrado, "Erro: Empresa não encontrada".
- Verificação do Empréstimo: Busca `idEmprestimo`. Se inválido, "Erro: Empréstimo não encontrado".
- Recuperação dos Dados do Empréstimo: Obtém dados do contrato.
- Consulta do Histórico de Pagamentos: Verifica parcelas pagas e restantes.
- Cálculo do Saldo Devedor: Executa 12.7. Saldo Devedor.
- Retorno dos Dados: Compila e retorna as informações.

### 7.3. Saída
```json
{
  "idEmpresa": "12.345.678/0001-90",
  "idEmprestimo": "EME001",
  "valorEmprestimo": 50000.00,
  "quantidadeParcelas": 24,
  "taxaJurosMensal": 0.017,
  "custoSeguro": 2500.00,
  "iof": 789.95,
  "valorTotalFinanciado": 54094.41,
  "primeiraParcela": 3173.53,
  "totalParcelasPagas": 5,
  "totalParcelasRestantes": 19,
  "saldoDevedor": 42824.76,
  "mensagem": "Consulta realizada com sucesso."
}
```
## 8. Pagamento de Empréstimo
### 8.1. Requisição
- Pagamento Parcial
```json
{
  "idEmpresa": "12.345.678/0001-90",
  "idEmprestimo": "EME001",
  "tipoPagamento": "parcial",
  "valorPagamento": 5000.00,
  "dataPagamento": "26/02/2025"
}
```
- Pagamento Total
```json
{
  "idEmpresa": "12.345.678/0001-90",
  "idEmprestimo": "EME001",
  "tipoPagamento": "total",
  "dataPagamento": "26/02/2025"
}
```
### 8.2. Processo Passo a Passo
- Consulta de Dados do Cliente: Valida `idEmpresa`.
- Verificação do Empréstimo: Busca `idEmprestimo`.
- Validação do Status do Empréstimo: Confirma se está ativo. Se quitado, "Erro: Empréstimo já liquidado".
- Consulta do Saldo Devedor Atual: Executa 12.7. Saldo Devedor.
- Processamento do Pagamento:
  - Total: Registra quitação total.
  - Parcial: Aplica o valor às parcelas mais antigas (amortização + juros). Se insuficiente, "Erro: Valor insuficiente".
- Atualização do Contrato: Marca como "quitado" (total) ou ajusta parcelas (parcial).
- Retorno da Confirmação: Retorna status atualizado.

### 8.3. Saída
- Pagamento Parcial
```json
{
  "idEmpresa": "12.345.678/0001-90",
  "idEmprestimo": "EME001",
  "tipoPagamento": "parcial",
  "valorPagamento": 5000.00,
  "saldoDevedor": 37824.76,
  "mensagem": "Pagamento parcial registrado com sucesso."
}
```
- Pagamento Total
```json
{
  "idEmpresa": "12.345.678/0001-90",
  "idEmprestimo": "EME001",
  "tipoPagamento": "total",
  "valorPagamento": 42824.76,
  "saldoDevedor": 0.00,
  "mensagem": "Empréstimo quitado com sucesso."
}
```
## 9. Refinanciamento de Empréstimo
### 9.1. Requisição
```json
{
  "idEmpresa": "12.345.678/0001-90",
  "idEmprestimoOriginal": "EME001",
  "novoValorEmprestimo": 10000.00,
  "novaQuantidadeParcelas": 36,
  "contratarSeguro": true,
  "dataInicioPagamento": "01/04/2025"
}
```
### 9.2. Processo Passo a Passo
- Consulta de Dados do Cliente: Valida `idEmpresa`.
- Verificação do Empréstimo Original: Busca `idEmprestimoOriginal`.
- Validação do Status: Confirma se ativo.
- Verificação de Elegibilidade: Aplica 11.2.1. Percentual Mínimo Pago e 11.1.2, 11.1.5.
- Cálculo do Saldo Devedor: Executa 12.7. Saldo Devedor.
- Determinação da Capacidade: Executa 12.1.
- Definição da Taxa de Juros: Executa 12.2.
- Cálculo do Custo do Seguro: Executa 12.3.
- Cálculo do IOF: Executa 12.4.
- Cálculo do Valor Total Financiado: Executa 12.5.
- Cálculo da Nova Parcela: Executa 12.6.
- Validação Final: Aplica 11.1.3.
- Registro do Refinanciamento: Cria novo contrato e marca o original como "refinanciado".

### 9.3. Saída
```json
{
  "idEmpresa": "12.345.678/0001-90",
  "idEmprestimoOriginal": "EME001",
  "idNovoEmprestimo": "EME002",
  "saldoDevedorOriginal": 42824.76,
  "novoValorEmprestimo": 10000.00,
  "novaQuantidadeParcelas": 36,
  "taxaJurosMensal": 0.0175,
  "custoSeguro": 2000.00,
  "iof": 315.98,
  "valorTotalFinanciado": 55140.74,
  "primeiraParcela": 2469.31,
  "ultimaParcela": 1532.24,
  "mensagem": "Refinanciamento realizado com sucesso."
}
```
## 10. Portabilidade de Empréstimo
### 10.1. Observação
 - A portabilidade não é aplicável a empréstimos empresariais neste sistema, pois não há regulamentação específica como no consignado. Caso necessária, seria tratada como refinanciamento em outra instituição, fora do escopo atual.

### 10.2. Saída
```json
{
  "mensagem": "Portabilidade não disponível para empréstimos empresariais."
}
```
## 11. Elegibilidade
### 11.1. Empréstimo Empresarial
#### 11.1.1. Valor do Empréstimo
 - `valorMinimoEmpresarial ≤ valorEmprestimo ≤ valorMaximoEmpresarial`.

#### 11.1.2. Quantidade de Parcelas
 - Microempresa: `12 ≤ quantidadeParcelas ≤ 48`.
 - Pequena empresa: `12 ≤ quantidadeParcelas ≤ 72`.
 - Média empresa: `12 ≤ quantidadeParcelas ≤ 96`.
 - Grande empresa: `12 ≤ quantidadeParcelas ≤ 120`.

#### 11.1.3. Capacidade de Pagamento
 - `primeiraParcela ≤ (faturamentoLiquidoAnual * 0.20) / 12 - dividasExistentes`.

#### 11.1.4. Taxa de Juros
 - Calculada dinamicamente, sem teto fixo, mas alinhada às práticas de mercado.

#### 11.1.5. Carência
 - Dias até o primeiro pagamento ≤ 90.

### 11.2. Refinanciamento
#### 11.2.1. Percentual Mínimo Pago
 - ≥ 20% das parcelas pagas.

## 12. Cálculos
### 12.1. Capacidade de Pagamento
- `capacidadeMaxima = (faturamentoLiquidoAnual * percentualFaturamento) / 12 - dividasExistentes`.

### 12.2. Taxa de Juros Mensal
- `TaxaJurosMensal = TaxaBase + 0.005 * ((quantidadeParcelas - 12) / 12)`

- TaxaBase:
  - Microempresa: 1,8% (com seguro) ou 2,1% (sem seguro).
  - Pequena empresa: 1,6% (com seguro) ou 1,9% (sem seguro).
  - Média empresa: 1,4% (com seguro) ou 1,7% (sem seguro).
  - Grande empresa: 1,2% (com seguro) ou 1,5% (sem seguro).

### 12.3. Custo do Seguro
- `CustoSeguro = valorEmprestimo * taxaSeguro (se contratarSeguro = true)`.

### 12.4. IOF
- `IOF = (iofFixo * valorEmprestimo) + (iofVariavel * valorEmprestimo * min(diasFinanciamento, 365))`.

### 12.5. Valor Total Financiado
- `ValorInicial = valorEmprestimo + IOF + CustoSeguro`

- `ValorTotalFinanciado = ValorInicial * (1 + TaxaJurosMensal / 30) ^ diasCarencia`.

### 12.6. Parcela Mensal (SAC)
- `Amortização = ValorTotalFinanciado / quantidadeParcelas`

- `Parcela = Amortização + (SaldoDevedorAnterior * TaxaJurosMensal)`.

### 12.7. Saldo Devedor
- `SaldoDevedor = ValorTotalFinanciado - (Amortização * totalParcelasPagas)`.
