# Emprest.AI Backend Documentation

## 1. Objetivo
O backend do Emprest.AI gerencia solicitações de empréstimos consignados, incluindo:

- Verificação de consignados anteriores;
- Cálculo da margem consignável (35% da remuneração líquida menos parcelas de outros empréstimos);
- Processamento da concessão com taxas de juros variáveis por vínculo e idade, prazos em múltiplos de 12 e limite de idade final de 80 anos;
- Inclusão opcional de seguro;
- Consulta e atualização de parcelas no banco de dados.

Se `quantidadeParcelas` não for informada, o sistema retorna opções de parcelamento. As regras seguem a Lei 10.820/2003, regulamentações do Banco Central (teto de 2,14% nas taxas) e do INSS.

## 2. Funcionalidades

### 2.1. Entrada de Dados

#### 2.1.1. Concessão de Empréstimo

- `idCliente`: CPF do cliente (ex.: "123.456.789-00").
- `valorEmprestimo`: Valor solicitado (ex.: 10000.00).
- `quantidadeParcelas`: Opcional, múltiplo de 12, mínimo 24 (ex.: 24, 36). Se omitido, retorna opções.
- `dataInicioPagamento`: Opcional, data futura (ex.: 01/04/2025). Se ausente, registra apenas como solicitação.
- `contratarSeguro`: Booleano (true ou false).
- `dataSolicitacao`: Automática (data de inserção no banco).

#### 2.1.2. Consulta de Empréstimos

- `idCliente`: CPF do cliente (ex.: "123.456.789-00").
- `idEmprestimo`: Opcional (ex.: "EMP-00123"). Se omitido, retorna todos os empréstimos do cliente.
- `dataConsulta`: Opcional (ex.: "22/02/2025"). Padrão: data atual.

#### 2.1.3. Atualização de Parcelas

- `idCliente`: CPF do cliente (ex.: "123.456.789-00").
- `idEmprestimo`: Identificador do empréstimo (ex.: "EMP-00123").
- `numeroParcela`: Parcela a atualizar (ex.: 3).
- `dataPagamento`: Data do pagamento (ex.: "15/07/2025").
- `valorPago`: Valor pago (ex.: 358.76).

Opcional (refinanciamento/portabilidade):

- `acao`: "refinanciar" ou "portar".
- `idEmprestimoNovo`: Novo contrato (ex.: "EMP-00124").
- `bancoDestino`: Banco receptor (ex.: "BANCOXYZ", obrigatório para portabilidade).

### 2.2. Verificação Inicial

#### Consulta ao Banco de Dados:

- Valida `idCliente` e, se fornecido, `idEmprestimo`.
- Obtém: remuneração líquida, parcelas anteriores, idade e `tipoVinculo` ("servidor_federal", "servidor_estadual", "servidor_municipal", "aposentado").
- Para refinanciamento/portabilidade: consulta saldo devedor e status de `idEmprestimoOriginal`.

#### Cálculo da Margem Consignável:

- Fórmula: `Margem = (Remuneração líquida * 0.35) - Parcelas anteriores`.
- Exemplo: Remuneração 5000.00, parcelas 800.00 → Margem = (5000.00 * 0.35) - 800.00 = 950.00.

#### Estrutura de Dados do Empréstimo:

- Geral: `valorEmprestimo`, `quantidadeParcelas`, `taxaJurosMensal`, `dataInicioPagamento`, `contratarSeguro`.
- Parcelas: `numeroParcela`, `dataVencimento`, `dataPagamento`, `valorParcelaOriginal`, `multaAtraso`, `jurosMora`, `valorPago`, `status` ("paga", "vencida", "a vencer").
- Refinanciamento/Portabilidade: `statusContrato` ("ativo", "refinanciado", "portado"), `idEmprestimoNovo`, `saldoDevedorOriginal`, `bancoDestino`.

### 2.3. Regras de Taxas e Prazos

- Taxa de Juros: Baseada em `tipoVinculo`, idade e `contratarSeguro`. Incremento de 0,0025 a cada 12 meses acima de 24, teto de 2,14%.
- Fórmula: `TaxaJurosMensal = TaxaBase + 0.0025 * ((QuantidadeParcelas - 24) / 12)`.
- Prazos: Múltiplos de 12 (mínimo 24), idade final ≤ 80 anos.

#### Taxas e Prazos por Vínculo

Com Seguro:

- Servidor Federal: Taxa base 1,3%, até 96 meses.
- Servidor Estadual: Taxa base 1,4%, até 84 meses.
- Servidor Municipal: Taxa base 1,5%, até 72 meses.
- Aposentado:
  - Até 66 anos: 1,3%, até 96 meses.
  - 67-70 anos: 1,45%, até 84 meses.
  - 71-74 anos: 1,45%, até 72 meses.
  - 75-78 anos: 1,6%, até 48 meses.
  - 79 anos: 1,6%, até 24 meses.

Sem Seguro:

- Taxa base + 0,2%, teto 2,14%.

#### Refinanciamento e Portabilidade

- Refinanciamento: Taxa ajustada por condições atuais, mesma fórmula de incremento, teto 2,14%.
- Portabilidade: Taxa definida pelo banco de destino, respeitando o teto 2,14%.

### 2.4. Cálculos Realizados

- Consulta Inicial: Obtém remuneração líquida, parcelas anteriores, idade, e, para refinanciamento/portabilidade, saldoDevedor.
- Taxa e Prazo:
  - `TaxaJurosMensal = TaxaBase + 0.0025 * ((QuantidadeParcelas - 24) / 12)`, teto 2,14%.
- Custo do Seguro:
  - Fórmula: `CustoSeguro = [0,04 + (0,001 * idade)] * ValorEmprestimo`.
  - Exemplo: Idade 75, 10000.00 → `CustoSeguro = 1150.00`.
- IOF:
  - `IOF_Fixo = 0,0038 * ValorEmprestimo`.
  - `IOF_Variavel = 0,000082 * ValorEmprestimo * min(NúmeroDeDias, 365)`.
  - `IOF_Total = IOF_Fixo + IOF_Variavel`.
- Carência:
  - `ValorInicial = ValorEmprestimo + IOF_Total + CustoSeguro`.
  - `ValorTotalFinanciado = ValorInicial * (1 + TaxaJurosMensal / 30) ^ Carência`.
- Parcela (Price):
  - `Parcela = [ValorTotalFinanciado * TaxaJurosMensal] / [1 - (1 + TaxaJurosMensal)^(-QuantidadeParcelas)]`.
- Taxa Efetiva Mensal (CET):
  - Resolvida numericamente: `ValorEmprestimo = Parcela * [(1 - (1 + TaxaEfetivaMensal)^(-QuantidadeParcelas)) / TaxaEfetivaMensal]`.
- Refinanciamento:
  - `SaldoDevedor = ValorOriginal - AmortizaçõesPagas`.
  - `ValorTotalFinanciado = SaldoDevedor + novoValorEmprestimo + CustoSeguro + IOF`.
- Portabilidade:
  - `ValorTotalFinanciado = SaldoDevedor + CustoSeguro + IOF`.
- Multa e Juros de Mora (Parcelas Vencidas):
  - Multa: `valorParcelaOriginal * 0.02`.
  - Juros: `valorParcelaOriginal * 0.000333 * DiasAtraso`.
  - Total: `valorParcelaOriginal + Multa + JurosMora`.
- Validação da Margem: `Parcela ≤ Margem`.

### 2.5. Validações

- `idCliente` válido, `dataInicioPagamento` futura, `valorEmprestimo` positivo.
- `quantidadeParcelas`: Múltiplo de 12, ≥ 24, idade final ≤ 80.
- Refinanciamento: Mínimo 20% das parcelas pagas.
- Portabilidade: Parcelas em dia, banco de destino compatível.
- Atualização: Parcela não paga previamente (exceto refinanciamento/portabilidade).

### 2.6. Saídas Geradas

#### Com quantidadeParcelas Fornecida

Exemplo (sem seguro):

```json
{
  "idCliente": "123.456.789-00",
  "valorEmprestimo": 10000.00,
  "parcela": 319.81,
  "quantidadeParcelas": 48,
  "dataInicioPagamento": "01/04/2025",
  "dataFimContrato": "01/04/2029",
  "taxaJurosMensal": 0.018,
  "taxaEfetivaMensal": 0.0185,
  "contratarSeguro": false,
  "custoSeguro": 0.00,
  "iof": 157.99,
  "carencia": 30,
  "valorTotalFinanciado": 10318.84,
  "margemUtilizada": 319.81,
  "margemRestante": 380.19,
  "prazoMaximoPermitido": 48
}
```

Exemplo (com seguro):

```json
{
  "idCliente": "123.456.789-00",
  "valorEmprestimo": 10000.00,
  "parcela": 350.13,
  "quantidadeParcelas": 48,
  "dataInicioPagamento": "01/04/2025",
  "dataFimContrato": "01/04/2029",
  "taxaJurosMensal": 0.0165,
  "taxaEfetivaMensal": 0.0192,
  "contratarSeguro": true,
  "custoSeguro": 1150.00,
  "iof": 157.99,
  "carencia": 30,
  "valorTotalFinanciado": 11496.87,
  "margemUtilizada": 350.13,
  "margemRestante": 349.87,
  "prazoMaximoPermitido": 48
}
```

#### Erros

- Margem excedida: `"Parcela (496.69) excede a margem disponível (700.00)"`.
- Prazo inválido: `"Quantidade de parcelas (60) excede o prazo máximo (48) para idade final ≤ 80"`.

#### Refinanciamento

```json
{
  "idCliente": "123.456.789-00",
  "idEmprestimoOriginal": "EMP-00123",
  "saldoDevedorOriginal": 8000.00,
  "novoValorEmprestimo": 2000.00,
  "parcela": 375.50,
  "novaQuantidadeParcelas": 48,
  "taxaJurosMensal": 0.0165,
  "taxaEfetivaMensal": 0.0190,
  "contratarSeguro": true,
  "custoSeguro": 920.00,
  "iof": 125.60,
  "valorTotalFinanciado": 11045.60,
  "margemUtilizada": 375.50,
  "margemRestante": 324.50
}
```

#### Portabilidade

```json
{
  "idCliente": "123.456.789-00",
  "idEmprestimoOriginal": "EMP-00123",
  "bancoDestino": "BANCOXYZ",
  "saldoDevedorOriginal": 8000.00,
  "parcela": 360.20,
  "novaQuantidadeParcelas": 48,
  "taxaJurosMensal": 0.0155,
  "taxaEfetivaMensal": 0.0180,
  "contratarSeguro": false,
  "custoSeguro": 0.00,
  "iof": 120.00,
  "valorTotalFinanciado": 8120.00,
  "margemUtilizada": 360.20,
  "margemRestante": 339.80
}
```

## 3. Fórmulas

- Margem: `(Remuneração líquida * 0.35) - Parcelas anteriores`.
- Taxa: `TaxaBase + 0.0025 * ((QuantidadeParcelas - 24) / 12)`, teto 2,14%.
- Seguro: `[0,04 + (0,001 * idade)] * ValorEmprestimo`.
- Parcela: `[ValorTotalFinanciado * TaxaJurosMensal] / [1 - (1 + TaxaJurosMensal)^(-QuantidadeParcelas)]`.
- CET: Resolvida numericamente com base em ValorTotalFinanciado.

## 4. Observações

- Sem `quantidadeParcelas`, retorna opções viáveis até o prazo máximo.
- Taxas respeitam o teto de 2,14% (Banco Central).
- Seguro é incluído no ValorTotalFinanciado.
- CET reflete o custo total (juros, IOF, seguro).

## 5. Referências Legais

- Lei 10.820/2003: Base para consignados.
- Lei 14.509/2022: Margem de 35% para servidores.
- INSS: Regulamentação para aposentados.
- Banco Central: Teto de 2,14% nas taxas.
