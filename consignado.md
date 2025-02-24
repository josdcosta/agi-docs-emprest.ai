# Documentação do Emprest.AI - Backend de Empréstimo Consignado

## Autores
- @Dalleth Martins
- @Josué Davi da Costa
- @Carollina Guedes
- @Victor Augusto Ferreira

## Referências
- Planilha de Cálculo
- Leis e Regulamentações: Lei 10.820/2003 (base para consignados), Lei 14.509/2022 (margem consignável de 35%), Regulamentação INSS, Resoluções do Banco Central, Código de Defesa do Consumidor (art. 52, §2º para multa e juros mora).

## 1. Objetivo
O Emprest.AI é um sistema backend desenvolvido para gerenciar de forma eficiente e automatizada todas as etapas relacionadas a empréstimos consignados. Isso inclui o cadastro de clientes, a análise de elegibilidade, a concessão de novos contratos (inclusive refinanciamentos e portabilidades), a consulta de status e parcelas, a atualização de pagamentos e o cancelamento de solicitações. O sistema respeita as normas brasileiras, como a margem consignável de 35% da remuneração líquida (Lei 14.509/2022), o teto de juros de 2,14% ao mês (Banco Central) e o limite de idade final de 80 anos (INSS). As taxas de juros são calculadas com base no vínculo do cliente e na idade, ajustadas dinamicamente pelo prazo escolhido, enquanto os prazos são oferecidos em múltiplos de 12 a partir de 24 meses. O seguro é opcional, e, caso o cliente não especifique a quantidade de parcelas, o sistema sugere opções viáveis.

## 2. Visão Geral do Funcionamento
O sistema é estruturado em cinco áreas principais:

- **Gerenciamento de Clientes**: Permite cadastrar e atualizar informações essenciais, como remuneração e margem consignável.
- **Concessão de Empréstimos**: Processa novos empréstimos, refinanciamentos e portabilidades com cálculos automáticos.
- **Consulta de Empréstimos**: Disponibiliza informações sobre elegibilidade, contratos ativos e parcelas.
- **Atualização de Dados**: Registra pagamentos e ajustes em contratos existentes.
- **Cancelamento de Contrato**: Gerencia a finalização de solicitações antes do início ou com reembolso, se aplicável.

### Regras Principais
- **Margem Consignável**: Calculada como 35% da remuneração líquida menos o valor das parcelas de outros empréstimos ativos, armazenada e obtida diretamente da tabela de clientes no banco de dados.
- **Taxas de Juros**: Determinadas pelo tipo de vínculo e idade do cliente, com um incremento específico por prazo, sempre respeitando o limite de 2,14% ao mês.
- **Prazos**: Oferecidos em múltiplos de 12 (ex.: 24, 36, 48 meses), com mínimo de 24 meses e máximo ajustado para que a idade do cliente ao fim do contrato não ultrapasse 80 anos.
- **Seguro**: Opcional, com custo baseado na idade e no valor do empréstimo.
- **IOF**: Incorporado ao valor financiado, conforme regulamentação fiscal.
- **Carência**: Período padrão de 30 dias entre a solicitação e o primeiro pagamento, ajustável até um máximo de 60 dias.

## 3. Gerenciamento de Clientes
### 3.1. Cadastro/Atualização de Cliente
#### Entrada
| Campo              | Tipo    | Descrição                         | Exemplo           | Obrigatório? |
|--------------------|---------|-----------------------------------|-------------------|--------------|
| idCliente          | String  | CPF do cliente                    | "123.456.789-00"  | Sim          |
| nome               | String  | Nome completo                     | "João Silva"      | Sim          |
| remuneracaoLiquida | Decimal | Renda líquida mensal              | 5000.00           | Sim          |
| idade              | Inteiro | Idade na data atual               | 75                | Sim          |
| tipoVinculo        | String  | Tipo de vínculo empregatício      | "aposentado"      | Sim          |
| margemConsignavel  | Decimal | Margem disponível (35% - parcelas) | 950.00            | Sim          |

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
### 4.1. Entrada
| Campo     | Tipo   | Descrição       | Exemplo          | Obrigatório? |
|-----------|--------|-----------------|------------------|--------------|
| idCliente | String | CPF do cliente  | "123.456.789-00" | Sim          |

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
| Campo              | Tipo    | Descrição                              | Exemplo           | Obrigatório? |
|--------------------|---------|----------------------------------------|-------------------|--------------|
| idCliente          | String  | CPF do cliente                         | "123.456.789-00"  | Sim          |
| valorEmprestimo    | Decimal | Valor solicitado                       | 10000.00          | Sim          |
| quantidadeParcelas | Inteiro | Número de parcelas (múltiplo de 12, ≥ 24) | 36                | Não          |
| dataInicioPagamento| Data    | Data do primeiro pagamento (futura, até 60 dias) | "01/04/2025" | Não          |
| contratarSeguro    | Boolean | Opção de contratar seguro              | true              | Sim          |

Nota: Se quantidadeParcelas não for fornecida, o sistema gera opções possíveis. Se dataInicioPagamento não for informada, assume carência de 30 dias a partir da data atual (ex.: 24/02/2025).

#### Dados Adicionais (Refinanciamento ou Portabilidade)
| Campo              | Tipo    | Descrição                              | Exemplo           | Obrigatório? |
|--------------------|---------|----------------------------------------|-------------------|--------------|
| idEmprestimoOriginal | String | Identificador do empréstimo existente  | "EMP-00123"       | Sim (ref/port)|
| novoValorEmprestimo  | Decimal | Valor adicional (refinanciamento)     | 2000.00           | Não          |
| novaQuantidadeParcelas | Inteiro | Novo prazo (múltiplo de 12, ≥ 24)    | 48                | Sim (ref/port)|
| bancoDestino         | String  | Banco receptor (portabilidade)        | "BANCOXYZ"        | Sim (port)   |

### 5.2. Processo de Cálculo
#### Consulta Inicial
O sistema valida o idCliente na tabela de clientes, obtendo remuneracaoLiquida, margemConsignavel, idade (atualizada na data atual), e tipoVinculo. Para refinanciamento ou portabilidade, consulta o idEmprestimoOriginal para recuperar o saldo devedor.

#### Validação
- Cliente inexistente: "Erro: Cliente não encontrado".
- Margem insuficiente: "Erro: Margem consignável insuficiente (X.XX)".
- Data inválida: "Erro: Data de início de pagamento inválida".

#### Margem Consignável
A margem é obtida diretamente do campo margemConsignavel na tabela de clientes, representando o valor disponível após descontar parcelas ativas de outros empréstimos.

#### Taxa de Juros
A taxa de juros mensal é calculada com base no tipo de vínculo, na idade e na opção de seguro, ajustada pelo prazo escolhido. A fórmula considera uma taxa base fixa e um incremento proporcional ao número de parcelas:

- **Fórmula**: TaxaJurosMensal = TaxaBase + 0.0025 * ((QuantidadeParcelas - 24) / 12)
- **TaxaBase**: Definida por vínculo e idade (ex.: 1,6% para aposentado de 75-78 anos com seguro).
- **Incremento**: Para cada 12 meses acima do prazo mínimo de 24 meses, a taxa aumenta em 0,0025 (ou 0,25% em termos percentuais). Por exemplo:
  - 24 meses: apenas a taxa base (ex.: 1,6% = 0,016).
  - 36 meses: TaxaBase + 0,0025 * (36 - 24) / 12 = TaxaBase + 0,0025 * 1 = TaxaBase + 0,0025 (ex.: 1,6% + 0,25% = 1,85%).
  - 48 meses: TaxaBase + 0,0025 * (48 - 24) / 12 = TaxaBase + 0,0025 * 2 = TaxaBase + 0,005 (ex.: 1,6% + 0,5% = 2,1%).
- **Teto**: O resultado é limitado a 2,14% (0,0214), conforme regulamentação do Banco Central.

#### Taxas Base por Vínculo e Idade
- **Com Seguro**:
  - Servidor Federal: 1,3% (0,013), máximo 96 meses.
  - Servidor Estadual: 1,4% (0,014), máximo 84 meses.
  - Servidor Municipal: 1,5% (0,015), máximo 72 meses.
  - Aposentado:
    - ≤ 66 anos: 1,3% (0,013), máximo 96 meses.
    - 67-70 anos: 1,45% (0,0145), máximo 84 meses.
    - 71-74 anos: 1,45% (0,0145), máximo 72 meses.
    - 75-78 anos: 1,6% (0,016), máximo 48 meses.
    - 79 anos: 1,6% (0,016), máximo 24 meses.
- **Sem Seguro**: Taxa base + 0,2% (ex.: 1,6% → 1,8%), mesmos prazos, teto 2,14%.

Esse ajuste reflete o impacto do prazo maior no risco do empréstimo, mantendo a conformidade legal.

#### Custo do Seguro (se contratado)
- **Fórmula**: CustoSeguro = [0,04 + (0,001 * idade)] * ValorEmprestimo
- Exemplo: Idade 75, R$ 10.000 → Custo = [0,04 + (0,001 * 75)] * 10000 = 1.150,00.

#### IOF
- **Fórmula**: IOF = (0,0038 * ValorEmprestimo) + (0,000082 * ValorEmprestimo * min(NúmeroDeDias, 365))

#### Valor Total Financiado
Inclui a carência (padrão 30 dias, máximo 60):
- **ValorInicial** = ValorEmprestimo + IOF + CustoSeguro
- **ValorTotalFinanciado** = ValorInicial * (1 + TaxaJurosMensal / 30) ^ DiasCarencia

#### Parcela Mensal (Método Price)
- **Fórmula**: Parcela = [ValorTotalFinanciado * TaxaJurosMensal] / [1 - (1 + TaxaJurosMensal)^(-QuantidadeParcelas)]

#### Taxa Efetiva Mensal (CET)
Resolvida numericamente para incluir todos os custos (juros, IOF, seguro):
- **ValorEmprestimo** = Parcela * [(1 - (1 + CET)^(-QuantidadeParcelas)) / CET]

#### Validação Final
- Parcela ≤ margemConsignavel.
- Idade final (idade + prazo em anos) ≤ 80.
- quantidadeParcelas múltiplo de 12, ≥ 24.
- Dias de carência ≤ 60.
- TaxaJurosMensal ≤ 0,0214.

### 5.3. Saídas
#### Com quantidadeParcelas Informada
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

Nota: Aqui, taxaJurosMensal = 0,0170 reflete a base de 1,6% + incremento de 0,5% para 48 meses (0,0025 * 2).

#### Sem quantidadeParcelas
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

Nota: As taxas aumentam gradualmente: 1,65% (24 meses), 1,675% (36 meses), 1,7% (48 meses), refletindo o incremento de 0,0025 por cada 12 meses adicionais.

#### Erros
- "Erro: Cliente não encontrado"
- "Erro: Margem consignável insuficiente (X.XX)"
- "Parcela (496.69) excede margem disponível (950.00)"
- "Quantidade de parcelas (60) excede o máximo (48) para idade 75"
- "Carência (70 dias) excede o limite máximo de 60 dias"
- "Taxa de juros (2.15%) excede o teto máximo de 2,14%"

## 6. Refinanciamento e Portabilidade
### 6.1. Refinanciamento
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

# 7. Consulta e Atualização de Parcelas

## 7.1. Consulta

### Saída

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

## 7.2. Atualização

### Saída (Pagamento Parcial)

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

# 8. Cancelamento de Contrato

## 8.1. Saída (Com Reembolso)

```json
{
  "idCliente": "123.456.789-00",
  "idEmprestimo": "EMP-00123",
  "statusContrato": "cancelado",
  "valorReembolso": 10033.25,
  "mensagem": "Contrato EMP-00123 cancelado. Reembolso devido: 10033.25 (juros proporcionais inclusos)."
}
```

# Observações

- A margem consignável é mantida atualizada na tabela de clientes, refletindo o impacto de contratos ativos.
- O incremento de 0,0025 na taxa de juros por cada 12 meses adicionais acima de 24 reflete o risco associado a prazos mais longos, sempre limitado a 2,14%.
- Todas as operações podem gerar logs para auditoria, dependendo da implementação do banco de dados.
 
