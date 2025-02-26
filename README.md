# Documentações Emprest.AI 📄
# **EMPRÉSTIMO CONSIGNADO**

## Índice
- [Autores](#autores)
- [Referências](#referencias)
- [Objetivo](#1-objetivo)

## Autores
- [@Dalleth Martins](https://github.com/dalleth-martinss)
- [@Josué Davi da Costa](https://github.com/josdcosta)
- [@Carollina Guedes](https://github.com/CarollinaGuedes)
- [@Victor Augusto Ferreira](https://github.com/Victor-augusto-ferreira)
- [@Joao Formigari](https://github.com/Joao-Formigari)

## Referências
- [Planilha de cálculo](https://docs.google.com/spreadsheets/d/1Y_vrP424Qpyh_nWdp_xtSSbsdswpp4XKPIOVeIV9B4E/edit?usp=sharing)
- Leis e Regulamentações: Lei 10.820/2003 (base para consignados), Lei 14.509/2022 (margem consignável de 35%), Regulamentação INSS, Resoluções do Banco Central, Código de Defesa do Consumidor (art. 52, §2º para multa e juros mora).

# 1. Objetivo
O Emprest.AI é um backend projetado para gerenciar de forma eficiente e transparente o ciclo completo de empréstimos, abrangendo as modalidades Empréstimo Pessoal e Empréstimo Consignado. Suas funcionalidades incluem concessão de novos contratos, simulação de condições, pagamento antecipado, refinanciamento (quando aplicável), portabilidade (para consignado) e cancelamento, com critérios adaptados a cada modalidade.

## Variáveis Configuráveis
Os parâmetros abaixo são ajustáveis no sistema Emprest.AI, podendo ser alterados conforme políticas internas, regulamentações ou condições comerciais. Os valores atuais estão listados na tabela:

## Variáveis Configuráveis
Os parâmetros abaixo são ajustáveis no sistema Emprest.AI, podendo ser alterados conforme políticas internas, regulamentações ou condições comerciais. Os valores atuais estão listados na tabela:

# Variáveis Configuráveis
Os parâmetros abaixo são ajustáveis no sistema Emprest.AI, podendo ser alterados conforme políticas internas, regulamentações ou condições comerciais. Os valores atuais estão listados na tabela:

| **Variável**               | **Descrição**                                     | **Valor Atual**            |
|----------------------------|--------------------------------------------------|----------------------------|
| `jurosMinimoPessoal`      | Taxa mínima de juros mensal (Empréstimo Pessoal) | 8,49% ao mês              |
| `jurosMaximoPessoal`      | Taxa máxima de juros mensal (Empréstimo Pessoal) | 9,99% ao mês              |
| `jurosMinimoConsignado`   | Taxa mínima de juros mensal (Empréstimo Consignado, 24 meses) | 1,80% ao mês       |
| `jurosMaximoConsignado`   | Taxa máxima de juros mensal (Empréstimo Consignado, 92 meses) | 2,14% ao mês       |
| `valorMinimoPessoal`      | Valor mínimo do Empréstimo Pessoal              | R$ 100,00                 |
| `valorMaximoPessoal`      | Valor máximo do Empréstimo Pessoal              | R$ 20.000,00              |
| `valorMinimoConsignado`   | Valor mínimo do Empréstimo Consignado           | Depende de `margemConsignavel` (35%) |
| `prazoMinimoPessoal`      | Prazo mínimo em parcelas (Empréstimo Pessoal)   | 6 parcelas                |
| `prazoMaximoPessoal`      | Prazo máximo em parcelas (Empréstimo Pessoal)   | 30 parcelas               |
| `prazoMinimoConsignado`   | Prazo mínimo em parcelas (Empréstimo Consignado)| 24 parcelas               |
| `prazoMaximoConsignado`   | Prazo máximo em parcelas (Empréstimo Consignado)| 92 parcelas               |
| `carenciaMaximaPessoal`   | Período máximo de carência (Empréstimo Pessoal) | 30 dias                   |
| `carenciaMaximaConsignado`| Período máximo de carência (Empréstimo Consignado) | 60 dias                |
| `idadeMaximaConsignado`   | Idade máxima ao final (Empréstimo Consignado)   | 80 anos                   |
| `margemConsignavel`       | Percentual da remuneração líquida para margem   | 35%                       |
| `iof`                     | Imposto sobre Operações Financeiras             | Conforme legislação       |
| `percentualRendaPessoal`  | Percentual máximo da renda líquida para parcela (Empréstimo Pessoal) | 30%          |
| `percentualMinimoRefinanciamento` | Percentual mínimo de parcelas pagas para refinanciamento | 20%        |

# 2. Visão Geral do Funcionamento
O sistema é estruturado em quatro áreas principais, aplicáveis a ambas as modalidades com ajustes específicos:

- **Concessão de Empréstimos**: Análise de crédito adaptada (Consignado: margem consignável; Pessoal: score e renda). Simulação e aprovação de contratos.
- **Consulta de Empréstimos**: Acompanhamento de status, parcelas e histórico de pagamentos.
- **Atualização de Dados**: Registro de pagamentos, incluindo antecipações e ajustes.
- **Cancelamento de Contrato**: Gestão de desistências ou finalizações, com regras específicas por modalidade.

## Regras Principais

### Empréstimo Pessoal
- **Prazo**: Entre `prazoMinimoPessoal` e `prazoMaximoPessoal` parcelas mensais.
- **Valor**: Entre `valorMinimoPessoal` e `valorMaximoPessoal`, sujeito à análise de crédito.
- **Taxas de Juros**: De `jurosMinimoPessoal` a `jurosMaximoPessoal` ao mês (base 30 dias), conforme score, valor e prazo.
- **Carência**: Até `carenciaMaximaPessoal` dias entre contratação e primeiro pagamento.
- **Análise de Crédito**: Baseada em score, renda líquida e idade.
- **Pagamento**: Débito automático em conta corrente, com opção de antecipação.
- **Cancelamento**: Direito de arrependimento em até 7 dias corridos, com devolução dos valores. Após isso, liquidação antecipada pode incluir tributos como `iof`.

### Empréstimo Consignado
- **Prazo**: Mínimo de 24 parcelas mensais e máximo de 92 parcelas mensais.
- **Valor**: Definido pela margem consignável (35% da remuneração líquida, menos outros empréstimos ativos).
- **Taxas de Juros**: De 1,80% ao mês (24 meses) a 2,14% ao mês (92 meses), com incremento gradual.
- **Carência**: Até 60 dias entre solicitação e primeiro pagamento.
- **Análise de Crédito**: Baseada na margem consignável, sem consulta de score.
- **Pagamento**: Desconto direto em folha de pagamento, com opção de antecipação.
- **Cancelamento**: Possível antes do início ou com reembolso, conforme regulamentação.

## Regras Comuns
- **Seguro**: Opcional, com custo baseado em idade e valor do empréstimo.
- **IOF**: Incorporado ao valor financiado, conforme legislação fiscal.
- **Condições**: Sujeitas a análise de crédito e variações comerciais no momento da contratação.

## 3. Dados armazenados do Clientes
Exemplo:
```json
{
  "idCliente": "123.456.789-00",
  "nome": "João Silva",
  "remuneracaoLiquidaMensal": 5000.00,
  "idade": 75,
  "tipoVinculo": "aposentado"
}
```

# 4. Simulação de Empréstimo

## 4.1. Requisição
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

## 4.2. Processo
- **Consulta de Elegibilidade**: Verifica em Elegibilidade.
- **Cálculo**: Executa Cálculos do Empréstimo sem registrar o contrato.
- **Saída Simulada**: Retorna os valores calculados ou erro.

## 4.3. Saída
```json
{
  "idCliente": "123.456.789-00",
  "valorEmprestimo": 10000.00,
  "quantidadeParcelas": 48,
  "taxaJurosMensal": 0.0192,
  "custoSeguro": 1150.00,
  "iof": 62.60,
  "valorTotalFinanciado": 11284.47,
  "parcelaMensal": 319.68,
  "cetMensal": 0.0201,
  "mensagem": "Simulação realizada com sucesso."
}
```

# 5. Concessão de Empréstimo

## 5.1. Requisição
```json
{
  "idCliente": "123.456.789-00",
  "valorEmprestimo": 10000.00,
  "quantidadeParcelas": 48,
  "contratarSeguro": true,
  "dataInicioPagamento": "01/04/2025"
}
```

## 5.2. Processo
- **Consulta de Elegibilidade**: Verifica em Elegibilidade.
- **Cálculo**: Executa Cálculos do Empréstimo sem registrar o contrato.
- **Saída Simulada**: Retorna os valores calculados.

## 5.3. Saída
```json
{
  "idCliente": "123.456.789-00",
  "valorEmprestimo": 10000.00,
  "quantidadeParcelas": 48,
  "taxaJurosMensal": 0.0192,
  "custoSeguro": 1150.00,
  "iof": 62.60,
  "valorTotalFinanciado": 11284.47,
  "parcelaMensal": 319.68,
  "cetMensal": 0.0201,
  "mensagem": "Simulação realizada com sucesso."
}
```

# 5. Elegibilidade
## Regras Específicas por Modalidade
### Empréstimo Pessoal
- **Valor do Empréstimo**: `valorMinimoPessoal ≤ valorEmprestimo ≤ valorMaximoPessoal`, limitado pelo score de crédito.
- **Quantidade de Parcelas**: `prazoMinimoPessoal ≤ quantidadeParcelas ≤ prazoMaximoPessoal`, com limites por score:
  - 201-400: 6-12.
  - 401-600: 6-18.
  - 601-800: 6-24.
  - 801-1000: 6-30.
- **Score de Crédito**: `scoreCredito ≥ 201`, com tabela de limites, prazos e taxas.
- **Capacidade de Pagamento**: `Parcela ≤ percentualRendaPessoal da rendaLiquida`.
- **Liquidação Antecipada**: Após 7 dias, com cálculo de valor presente + tributos (ex.: `iof`).

### Empréstimo Consignado
- **Margem Consignável**: `Parcela ≤ margemConsignavel`, calculada como `(remuneracaoLiquida * margemConsignavel) - soma das parcelas ativas`.
- **Idade Máxima**: Idade final (`idade + quantidadeParcelas / 12`) ≤ `idadeMaximaConsignado`.
- **Quantidade de Parcelas**: `prazoMinimoConsignado ≤ quantidadeParcelas ≤ prazoMaximoConsignado`.
- **Taxa de Juros**: `TaxaJurosMensal ≤ jurosMaximoConsignado`.
- **Tipo de Vínculo**: Exige vínculo válido (ex.: "aposentado", "servidor público").
- **Portabilidade**: Parcelas em dia, `bancoDestino` aceito, nova parcela ≤ `margemConsignavel`.

## Regras Gerais/Comuns
### Refinanciamento
- **Percentual Mínimo Pago**: ≥ 20% das parcelas do `idEmprestimoOriginal` pagas (status "paga" em Parcelas).
- **Limite de Pagamento**:
  - Pessoal: Nova parcela ≤ `min(rendaLiquida * 0.30, limiteMaximoPorScore)`.
  - Consignado: Nova parcela ≤ `margemConsignavel`.
- **Idade Final**:
  - Pessoal: Não aplicável diretamente, mas pode ser opcional.
  - Consignado: ≤ `idadeMaxima` (ex.: 80).
- **Erro**: "Pagamento mínimo não atingido" ou "Limite insuficiente".

