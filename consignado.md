# Documentação do Emprest.AI - Backend de Empréstimo Consignado

## Autores
- @Dalleth Martins
- @Josué Davi da Costa
- @Carollina Guedes
- @Victor Augusto Ferreira

## Referências
- Planilha de Cálculo
- Leis e Regulamentações: Lei 10.820/2003, Lei 14.509/2022, Regulamentação INSS, Resoluções do Banco Central.

## 1. Objetivo
O Emprest.AI é um sistema backend projetado para gerenciar solicitações, consultas e atualizações de empréstimos consignados. Ele automatiza o processo de concessão, refinanciamento e portabilidade, respeitando as regras legais brasileiras, como a margem consignável de 35% da remuneração líquida (Lei 14.509/2022), o teto de juros de 2,14% ao mês (Banco Central) e o limite de idade final de 80 anos (INSS). O sistema calcula taxas de juros variáveis com base no vínculo do cliente e na idade, oferece prazos em múltiplos de 12 (mínimo 24 meses) e inclui a opção de contratar seguro. Se o número de parcelas não for informado, retorna opções viáveis.

## 2. Visão Geral do Funcionamento
O sistema abrange três grandes áreas:

- **Concessão de Empréstimos**: Processa novos empréstimos, refinanciamentos e portabilidades.
- **Consulta de Empréstimos**: Permite verificar o status de contratos e parcelas.
- **Atualização de Dados**: Registra pagamentos e gerencia alterações como refinanciamento ou portabilidade.

### Regras Principais
- **Margem Consignável**: 35% da remuneração líquida menos parcelas de outros empréstimos.
- **Taxas de Juros**: Variam por vínculo e idade, com incremento de 0,0025 por cada 12 meses acima de 24, limitadas a 2,14%.
- **Prazos**: Múltiplos de 12 (24, 36, 48...), até o máximo permitido pela idade (≤ 80 anos no fim do contrato).
- **Seguro**: Opcional, calculado com base na idade e no valor do empréstimo.
- **IOF**: Incluído no valor financiado, conforme regras fiscais.

## 3. Concessão de Empréstimos

### 3.1. Entrada de Dados
Os dados necessários para processar uma solicitação são:

| Campo               | Tipo    | Descrição                                 | Exemplo           | Obrigatório? |
|---------------------|---------|-------------------------------------------|-------------------|--------------|
| idCliente           | String  | CPF do cliente                            | "123.456.789-00"  | Sim          |
| valorEmprestimo     | Decimal | Valor solicitado                          | 10000.00          | Sim          |
| quantidadeParcelas  | Inteiro | Número de parcelas (múltiplo de 12, ≥ 24) | 36                | Não          |
| dataInicioPagamento | Data    | Data do primeiro pagamento (futura)       | "01/04/2025"      | Não          |
| contratarSeguro     | Booleano| Opção de contratar seguro                 | true              | Sim          |

**Nota**: Se `quantidadeParcelas` não for fornecida, o sistema retorna opções possíveis. Se `dataInicioPagamento` não for informada, a solicitação é registrada sem processamento imediato.

#### Dados Adicionais (Refinanciamento ou Portabilidade)

| Campo               | Tipo    | Descrição                                 | Exemplo           | Obrigatório? |
|---------------------|---------|-------------------------------------------|-------------------|--------------|
| idEmprestimoOriginal| String  | Identificador do empréstimo existente     | "EMP-00123"       | Sim (ref/port)|
| novoValorEmprestimo | Decimal | Valor adicional (refinanciamento)         | 2000.00           | Não          |
| bancoDestino        | String  | Banco receptor (portabilidade)            | "BANCOXYZ"        | Sim (port)   |

### 3.2. Processo de Cálculo

#### Consulta Inicial:
Valida o `idCliente` e busca no banco de dados:
- Remuneração líquida.
- Parcelas de outros empréstimos.
- Idade e tipo de vínculo ("servidor_federal", "servidor_estadual", "servidor_municipal", "aposentado").
- Para refinanciamento/portabilidade: saldo devedor do `idEmprestimoOriginal`.

#### Margem Consignável:
- Fórmula: `Margem = (Remuneração líquida * 0.35) - Parcelas anteriores`.
- Exemplo: Renda R$ 5.000, parcelas R$ 800 → `Margem = (5000 * 0.35) - 800 = R$ 950,00`.

#### Taxa de Juros:
- Baseada em vínculo, idade e seguro, com incremento de 0,0025 por cada 12 meses acima de 24, até 2,14%.
- Fórmula: `TaxaJurosMensal = TaxaBase + 0.0025 * ((QuantidadeParcelas - 24) / 12)`.

#### Taxas por Vínculo e Idade
**Com Seguro**:
- Servidor Federal: 1,3%, até 96 meses.
- Servidor Estadual: 1,4%, até 84 meses.
- Servidor Municipal: 1,5%, até 72 meses.
- Aposentado:
  - ≤ 66 anos: 1,3%, até 96 meses.
  - 67-70 anos: 1,45%, até 84 meses.
  - 71-74 anos: 1,45%, até 72 meses.
  - 75-78 anos: 1,6%, até 48 meses.
  - 79 anos: 1,6%, até 24 meses.

**Sem Seguro**: Taxa base + 0,2%, mesmo limite de prazos.

#### Custo do Seguro (se contratado):
- Fórmula: `CustoSeguro = [0,04 + (0,001 * idade)] * ValorEmprestimo`.
- Exemplo: Idade 75, R$ 10.000 → `Custo = [0,04 + (0,001 * 75)] * 10000 = R$ 1.150,00`.

#### IOF:
- `IOF_Fixo = 0,0038 * ValorEmprestimo`.
- `IOF_Variavel = 0,000082 * ValorEmprestimo * min(NúmeroDeDias, 365)`.
- Exemplo: R$ 10.000, 48 meses → IOF ≈ R$ 157,99.

#### Valor Total Financiado:
- Inclui carência (30 dias padrão):
  - `ValorInicial = ValorEmprestimo + IOF + CustoSeguro`.
  - `ValorTotalFinanciado = ValorInicial * (1 + TaxaJurosMensal / 30) ^ 30`.

#### Parcela Mensal (Método Price):
- Fórmula: `Parcela = [ValorTotalFinanciado * TaxaJurosMensal] / [1 - (1 + TaxaJurosMensal)^(-QuantidadeParcelas)]`.

#### Taxa Efetiva Mensal (CET):
- Calculada numericamente para refletir o custo total (juros, IOF, seguro):
  - `ValorEmprestimo = Parcela * [(1 - (1 + CET)^(-QuantidadeParcelas)) / CET]`.

#### Validação:
- `Parcela ≤ Margem`.
- Idade final (idade atual + prazo em anos) ≤ 80.
- `QuantidadeParcelas` múltiplo de 12, ≥ 24.

### 3.3. Saídas

#### Com quantidadeParcelas Informada

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
  "valorTotalFinanciado": 11496.87,
  "margemUtilizada": 350.13,
  "margemRestante": 349.87,
  "prazoMaximoPermitido": 48
}
```

#### Sem quantidadeParcelas

Retorna opções viáveis:

```json
{
  "idCliente": "123.456.789-00",
  "valorEmprestimo": 10000.00,
  "contratarSeguro": true,
  "opcoesParcelamento": [
    {"quantidadeParcelas": 24, "parcela": 582.45, "taxaJurosMensal": 0.0165, "valorTotalFinanciado": 11396.87},
    {"quantidadeParcelas": 36, "parcela": 416.78, "taxaJurosMensal": 0.01675, "valorTotalFinanciado": 11446.87},
    {"quantidadeParcelas": 48, "parcela": 350.13, "taxaJurosMensal": 0.0170, "valorTotalFinanciado": 11496.87}
  ]
}
```

#### Erros

- Margem excedida: `"Parcela (496.69) excede margem disponível (950.00)"`.
- Prazo inválido: `"Quantidade de parcelas (60) excede o máximo (48) para idade 75"`.

## 4. Refinanciamento e Portabilidade

### 4.1. Refinanciamento
Renegocia um empréstimo existente no mesmo banco.

#### Condições:
- Mínimo 20% das parcelas pagas.
- Novo prazo respeita idade ≤ 80.

#### Cálculo:
- `ValorTotalFinanciado = SaldoDevedor + NovoValorEmprestimo + CustoSeguro + IOF`.
- Nova taxa ajustada conforme vínculo e idade.

#### Saída

```json
{
  "idCliente": "123.456.789-00",
  "idEmprestimoOriginal": "EMP-00123",
  "saldoDevedorOriginal": 8000.00,
  "novoValorEmprestimo": 2000.00,
  "parcela": 375.50,
  "novaQuantidadeParcelas": 48,
  "taxaJurosMensal": 0.0165,
  "valorTotalFinanciado": 11045.60
}
```

### 4.2. Portabilidade
Transfere o empréstimo para outro banco.

#### Condições:
- Parcelas em dia.
- Banco de destino aceita portabilidade.

#### Cálculo:
- `ValorTotalFinanciado = SaldoDevedor + CustoSeguro + IOF`.
- Taxa definida pelo banco destino (≤ 2,14%).

#### Saída

```json
{
  "idCliente": "123.456.789-00",
  "idEmprestimoOriginal": "EMP-00123",
  "bancoDestino": "BANCOXYZ",
  "saldoDevedorOriginal": 8000.00,
  "parcela": 360.20,
  "novaQuantidadeParcelas": 48,
  "taxaJurosMensal": 0.0155,
  "valorTotalFinanciado": 8120.00
}
```

## 5. Consulta e Atualização de Parcelas

### 5.1. Consulta

#### Entrada

| Campo         | Tipo   | Descrição               | Exemplo           | Obrigatório? |
|---------------|--------|-------------------------|-------------------|--------------|
| idCliente     | String | CPF do cliente          | "123.456.789-00"  | Sim          |
| idEmprestimo  | String | Identificador do empréstimo | "EMP-00123"   | Não          |
| dataConsulta  | Data   | Data da consulta        | "22/02/2025"      | Não          |

#### Saída

```json
{
  "idCliente": "123.456.789-00",
  "idEmprestimo": "EMP-00123",
  "valorEmprestimo": 10000.00,
  "quantidadeParcelas": 48,
  "dataInicioPagamento": "01/04/2025",
  "parcelas": [
    {"numeroParcela": 1, "dataVencimento": "01/05/2025", "valorParcelaOriginal": 350.13, "status": "paga"},
    {"numeroParcela": 2, "dataVencimento": "01/06/2025", "valorParcelaOriginal": 350.13, "status": "a vencer"}
  ]
}
```

### 5.2. Atualização

#### Entrada

| Campo         | Tipo   | Descrição               | Exemplo           | Obrigatório? |
|---------------|--------|-------------------------|-------------------|--------------|
| idCliente     | String | CPF do cliente          | "123.456.789-00"  | Sim          |
| idEmprestimo  | String | Identificador do empréstimo | "EMP-00123"   | Sim          |
| numeroParcela | Inteiro| Parcela a atualizar     | 3                 | Sim          |
| dataPagamento | Data   | Data do pagamento       | "15/07/2025"      | Sim          |
| valorPago     | Decimal| Valor pago              | 358.76            | Sim          |

**Atraso**:
- Multa: 2% (`valorParcelaOriginal * 0.02`).
- Juros de mora: 0,0333% ao dia (`valorParcelaOriginal * 0.000333 * diasAtraso`).

#### Saída

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
  "valorPago": 358.76,
  "status": "paga"
}
```

## 6. Observações
- Todas as taxas respeitam o teto de 2,14% do Banco Central.
- O sistema ajusta automaticamente prazos para garantir idade final ≤ 80.
- Seguro e IOF são somados ao valor financiado, impactando a parcela.
- A CET reflete o custo total para transparência ao cliente.
