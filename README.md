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

# 2. Visão Geral do Funcionamento
O sistema é estruturado em quatro áreas principais, aplicáveis a ambas as modalidades com ajustes específicos:

- **Concessão de Empréstimos**: Análise de crédito adaptada (Consignado: margem consignável; Pessoal: score e renda). Simulação e aprovação de contratos.
- **Consulta de Empréstimos**: Acompanhamento de status, parcelas e histórico de pagamentos.
- **Atualização de Dados**: Registro de pagamentos, incluindo antecipações e ajustes.
- **Cancelamento de Contrato**: Gestão de desistências ou finalizações, com regras específicas por modalidade.

## Regras Principais

### Empréstimo Pessoal
- **Prazo**: Mínimo de 6 parcelas mensais e máximo de 30 parcelas mensais.
- **Valor**: Mínimo de R$ 100,00, sem limite máximo fixo (sujeito à análise).
- **Taxas de Juros**: De 8,49% a 9,99% ao mês (base 30 dias), conforme score, valor e prazo.
- **Carência**: Até 30 dias entre contratação e primeiro pagamento.
- **Análise de Crédito**: Baseada em score, renda líquida e idade.
- **Pagamento**: Débito automático em conta corrente, com opção de antecipação.
- **Cancelamento**: Direito de arrependimento em até 7 dias corridos, com devolução dos valores. Após isso, liquidação antecipada pode incluir tributos.

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
# Regras Específicas por Modalidade
## Empréstimo Pessoal
- **Valor do Empréstimo**: `R$ 100,00 ≤ valorEmprestimo ≤ R$ 20.000,00`, limitado pelo score de crédito.
- **Quantidade de Parcelas**: `6 ≤ quantidadeParcelas ≤ 30`, com limites por score:
  - 201-400: 6-12.
  - 401-600: 6-18.
  - 601-800: 6-24.
  - 801-1000: 6-30.
- **Score de Crédito**: `scoreCredito ≥ 201`, com tabela de limites, prazos e taxas.
- **Capacidade de Pagamento**: `Parcela ≤ 30% da rendaLiquida`.
- **Liquidação Antecipada**: Após 7 dias, com cálculo de valor presente + tributos (ex.: IOF).

## Empréstimo Consignado
- **Margem Consignável**: `Parcela ≤ margemConsignavel`, calculada como `(remuneracaoLiquida * 35%) - soma das parcelas ativas`.
- **Idade Máxima**: Idade final (`idade + quantidadeParcelas / 12`) ≤ 80 (ajustável).
- **Quantidade de Parcelas**: `24 ≤ quantidadeParcelas ≤ 92`, limitada por idade e `prazoMaximo`.
- **Taxa de Juros**: `TaxaJurosMensal ≤ 2,14%` (ajustável).
- **Tipo de Vínculo**: Exige vínculo válido (ex.: "aposentado", "servidor público").
- **Portabilidade**: Parcelas em dia, `bancoDestino` aceito, nova parcela ≤ `margemConsignavel`.

# Regras Gerais/Comuns
### Refinanciamento
- **Percentual Mínimo Pago**: ≥ 20% das parcelas do `idEmprestimoOriginal` pagas (status "paga" em Parcelas).
- **Limite de Pagamento**:
  - Pessoal: Nova parcela ≤ `min(rendaLiquida * 0.30, limiteMaximoPorScore)`.
  - Consignado: Nova parcela ≤ `margemConsignavel`.
- **Idade Final**:
  - Pessoal: Não aplicável diretamente, mas pode ser opcional.
  - Consignado: ≤ `idadeMaxima` (ex.: 80).
- **Erro**: "Pagamento mínimo não atingido" ou "Limite insuficiente".

#### Consulta Inicial
O sistema valida o `idCliente` na tabela de clientes, obtendo `remuneracaoLiquida`, `margemConsignavel`, `idade` (atualizada na data atual), e `tipoVinculo`. Consulta a tabela **ConfiguracoesLocais** para recuperar `taxaInicial`, `incrementoMensal`, `tetoJuros`, `prazoMinimo`, `prazoMaximo`, `idadeMaxima`, `carenciaMaxima` e `margemConsignavelPercentual`. Calcula a margem máxima como `remuneracaoLiquida * margemConsignavelPercentual`. Para refinanciamento ou portabilidade, consulta o `idEmprestimoOriginal` para recuperar o saldo devedor.

#### Validação
- **Cliente inexistente:** "Erro: Cliente não encontrado".
- **Margem insuficiente:** "Erro: Margem consignável insuficiente (X.XX)".
- **Data inválida:** "Erro: Data de início de pagamento inválida ou excede carenciaMaxima".
- **Prazo inválido:** "Erro: Quantidade de parcelas fora do intervalo (prazoMinimo a prazoMaximo) ou idade final excede idadeMaxima".

#### Taxa de Juros
A taxa de juros mensal é calculada com base nos valores locais:
- **Fórmula:** TaxaJurosMensal = taxaInicial + incrementoMensal * (QuantidadeParcelas - prazoMinimo)
- **Taxa Inicial:** Obtida de `taxaInicial` na tabela **ConfiguracoesLocais**.
- **Incremento:** Usa `incrementoMensal` para cada mês acima do `prazoMinimo`.
- **Teto:** Limitada ao valor de `tetoJuros`.
- **Exemplo:** Para uma configuração com taxaInicial = 0.018, incrementoMensal = 0.00005, prazoMinimo = 24, e quantidadeParcelas = 48, a taxa será 0.0192 (0.018 + 0.00005 * (48 - 24)), limitada ao tetoJuros.

#### Validação Final
- Parcela ≤ margemConsignavel.
- Idade final (idade + prazo em anos) ≤ idadeMaxima (obtida de ConfiguracoesLocais).
- quantidadeParcelas entre prazoMinimo e prazoMaximo.
- Dias de carência ≤ carenciaMaxima.
- TaxaJurosMensal ≤ tetoJuros.

#### Taxas Base por Vínculo e Idade
- A taxa inicial é fixa em 1,80% (0,018) para todos os vínculos e idades em 24 meses, independentemente da opção de seguro, ajustada apenas pelo prazo conforme a fórmula TaxaJurosMensal = 0,018 + 0,00005 * (QuantidadeParcelas - 24).
- **Exemplo:** 48 meses resulta em 1,92% (0,0192), e 92 meses em 2,14% (0,0214).

  ### PESSOAL

| Faixa de Score | Nível de Risco     | Taxa               | Limite Crédito     | Meses              |
|----------------|--------------------|--------------------|--------------------|--------------------| 
| 0-200          | Altíssimo risco    | N/A                | N/A                | N/A                |
| 201-400        | Alto risco         | 9,99%              | R$ 100 a R$ 1.000  | 6 a 12             |
| 401-600        | Risco moderado     | 9,49% a 9,99%      | R$ 100 a R$ 5.000  | 6 a 18             |
| 601-800        | Risco baixo        | 8,99% a 9,49%      | R$ 100 a R$ 15.000 | 6 a 24             |
| 801-1000       | Risco muito baixo  | 8,49% a 8,99%      | R$ 100 a R$ 20.000 | 6 a 30             |
|----------------|--------------------|--------------------|--------------------|--------------------|

8,49: Taxa mínima (em %).
9,99: Taxa máxima (em %).

Interpolação linear
Taxa = TaxaMáxima - (score - ScoreMínimo) * (TaxaMáxima - TaxaMínima) / (ScoreMáximo - ScoreMínimo)
TaxaJurosMensal (%) = 9,99 - [(score - 201) * 1,5] / 799

Valor = ValorInicial + (score - ScoreInicial) * (ValorFinal - ValorInicial) / (ScoreFinal - ScoreInicial)
LimiteCalculado (R$) = 100 + [(score - 201) * 19.900] / 799
**Limite máximo deve respeitar a margem de crédito em relação a renda

### Custo do Seguro (se contratado)

**Fórmula:**
```
CustoSeguro = [0,04 + (0,001 * idade)] * ValorEmprestimo
```

**Exemplo:**
```
Idade 75, R$ 10.000 → Custo = [0,04 + (0,001 * 75)] * 10000 = 1.150,00.
```

### IOF

**Fórmula:**
```
IOF = (0,0038 * ValorEmprestimo) + (0,000082 * ValorEmprestimo * min(NúmeroDeDias, 365))
```

### Valor Total Financiado

**Inclui a carência (padrão 30 dias, máximo 60):**

```
ValorInicial = ValorEmprestimo + IOF + CustoSeguro
ValorTotalFinanciado = ValorInicial * (1 + TaxaJurosMensal / 30) ^ DiasCarencia
```

### Parcela Mensal (Método Price)

**Fórmula:**
```
Parcela = [ValorTotalFinanciado * TaxaJurosMensal] / [1 - (1 + TaxaJurosMensal)^(-QuantidadeParcelas)]
```

### Taxa Efetiva Mensal (CET)

**Resolvida numericamente para incluir todos os custos (juros, IOF, seguro):**

```
ValorEmprestimo = Parcela * [(1 - (1 + CET)^(-QuantidadeParcelas)) / CET]
```

### Validação Final

- Parcela ≤ margemConsignavel.
- Idade final (idade + prazo em anos) ≤ 80.
- quantidadeParcelas múltiplo de 12, ≥ 24.
- Dias de carência ≤ 60.
- TaxaJurosMensal ≤ 0,0214.

### Registro

Após validação bem-sucedida, o contrato é registrado no banco de dados com status "ativo", associado ao `idCliente`, para acompanhamento futuro.

## 5.3. Saídas

### Com quantidadeParcelas Informada

**Requisição**

```json
{
  "idCliente": "123.456.789-00",
  "valorEmprestimo": 10000.00,
  "quantidadeParcelas": 48,
  "dataInicioPagamento": "01/04/2025",
  "contratarSeguro": true
}
```

**Saída**

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

# 6. Refinanciamento e Portabilidade

Essa seção descreve os processos de refinanciamento e portabilidade de empréstimos consignados, que permitem ao cliente renegociar um contrato existente ou transferi-lo para outro banco, respectivamente. Ambos os processos utilizam o saldo devedor do contrato original como base e aplicam regras específicas para elegibilidade e cálculo.

## 6.1. Refinanciamento

#### Dados Adicionais (Refinanciamento ou Portabilidade)
| Campo                | Tipo     | Descrição                                     | Exemplo         | Obrigatório? |
|----------------------|----------|-----------------------------------------------|-----------------|--------------|
| idEmprestimoOriginal | String   | Identificador do empréstimo existente         | "EMP-00123"     | Sim (ref/port)|
| novoValorEmprestimo  | Decimal  | Valor adicional (refinanciamento)             | 2000.00         | Não          |
| novaQuantidadeParcelas| Inteiro | Novo prazo (múltiplo de 12, ≥ 24)             | 48              | Sim (ref/port)|
| bancoDestino         | String   | Banco receptor (portabilidade)                | "BANCOXYZ"      | Sim (port)   |


O refinanciamento é uma operação que renegocia um empréstimo consignado existente no mesmo banco, possibilitando ao cliente estender o prazo, adicionar um valor extra ou ajustar as condições de pagamento. Isso geralmente é feito para reduzir a parcela mensal ou obter recursos adicionais.

### Condições
- **Pagamento Mínimo**: Pelo menos 20% das parcelas do contrato original devem estar pagas, verificado pela contagem de parcelas com status "paga" no banco de dados.
- **Idade Final**: O novo prazo (novaQuantidadeParcelas) deve garantir que a idade do cliente ao fim do contrato não exceda o valor de `idadeMaxima` obtido da tabela **ConfiguracoesLocais**, respeitando o limite de `prazoMaximo`.
- **Margem Consignável**: A nova parcela deve ser inferior ou igual à margem consignável disponível, obtida da tabela de clientes.

### Processo

1. **Consulta ao Contrato Original**: O sistema busca o idEmprestimoOriginal no banco de dados para obter:
   - Saldo devedor atual (saldoDevedorOriginal), calculado como o valor total financiado menos as amortizações das parcelas pagas.
   - Quantidade de parcelas pagas e restantes.
2. **Cálculo do Novo Empréstimo**:
   - O valor base é o saldoDevedorOriginal, acrescido de novoValorEmprestimo (se fornecido), mais o custo do seguro (se contratado) e o IOF.
   - A taxa de juros é recalculada com base na taxa inicial fixa de 1,80%, ajustada pelo prazo conforme a fórmula TaxaJurosMensal = 0,018 + 0,00005 * (novaQuantidadeParcelas - 24), respeitando o teto de 2,14%.
   - A nova parcela é calculada pelo método Price: Parcela = [ValorTotalFinanciado * TaxaJurosMensal] / [1 - (1 + TaxaJurosMensal)^(-novaQuantidadeParcelas)].
3. **Validação**: Confirma que a parcela não excede a margem consignável e que o prazo é compatível com a idade limite.
4. **Atualização**: O contrato original é marcado como "refinanciado", e um novo contrato é gerado com os dados calculados.

### Requisição

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

### Saída

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

### Explicação do Exemplo

O saldo devedor original (R$ 8.000,00) é combinado com o novo valor (R$ 2.000,00), mais seguro e IOF, resultando em valorTotalFinanciado de R$ 11.045,60.  
A taxa de 1,7% (0,017) reflete a base de 1,6% (aposentado, 75 anos, com seguro) + 0,5% (incremento de 0,0025 * 2 para 48 meses).

## 6.2. Portabilidade

A portabilidade permite transferir um empréstimo consignado para outro banco, geralmente para aproveitar taxas de juros menores ou melhores condições. O saldo devedor é quitado pelo banco de destino, que assume o contrato com um novo prazo e taxa.

### Condições
- **Parcelas em Dia**: Todas as parcelas do contrato original devem estar pagas até a data atual (nenhuma com status "vencida").
- **Aceitação do Banco Destino**: O banco receptor (bancoDestino) deve aceitar a portabilidade, o que pode ser validado por integração externa ou configuração manual.
- **Idade Final**: O novo prazo (novaQuantidadeParcelas) deve respeitar o limite de `idadeMaxima` da instalação atual, obtido da tabela **ConfiguracoesLocais**, e estar dentro do `prazoMaximo`. (Nota: As regras do banco de destino não são aplicadas localmente, mas podem ser informadas externamente.)
- **Margem Consignável**: A nova parcela deve caber na margem disponível.

### Processo

1. **Consulta ao Contrato Original**: O sistema busca o idEmprestimoOriginal para obter:
   - Saldo devedor atual (saldoDevedorOriginal), calculado como o valor total financiado menos as amortizações pagas.
   - Status de pagamento (confirma que está em dia).
2. **Cálculo no Banco de Destino**:
   - O valor base é o saldoDevedorOriginal, acrescido do custo do seguro (se contratado) e o IOF.
   - A taxa de juros pode ser informada pelo banco de destino ou calculada com base na taxa inicial fixa de 1,80%, ajustada pelo prazo conforme a fórmula TaxaJurosMensal = 0,018 + 0,00005 * (novaQuantidadeParcelas - 24), limitada a 2,14%.
   - A nova parcela é calculada pelo método Price, semelhante ao refinanciamento.
3. **Validação**: Verifica a margem consignável e o limite de idade.
4. **Transferência**: O contrato original é marcado como "portado", e o novo banco registra o contrato com os dados calculados.

### Requisição

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

### Saída

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

### Explicação do Exemplo

O saldo devedor original (R$ 8.000,00) é ajustado com IOF, resultando em valorTotalFinanciado de R$ 8.120,00 (sem seguro).  
A taxa de 1,55% (0,0155) é assumida como fornecida pelo banco destino, inferior à base padrão, justificando a portabilidade.

## 7. Consulta e Atualização de Parcelas

### 7.1. Consulta
(Requisição e saída mantidas como na versão anterior)

### 7.2. Atualização

**Requisição**

```json
{
  "idCliente": "123.456.789-00",
  "idEmprestimo": "EMP-00123",
  "numeroParcela": 3,
  "dataPagamento": "15/07/2025",
  "valorPago": 300.00
}
```

**Processo**

- **Validação:** Confirma a existência de `idCliente`, `idEmprestimo` e `numeroParcela`. Verifica se a parcela não está paga, a menos que seja refinanciamento/portabilidade.
- **Cálculo de Atraso:** Se `dataPagamento` > `dataVencimento`:
  - **Multa:** `valorParcelaOriginal * 0.02`.
  - **Juros de mora:** `valorParcelaOriginal * 0.000333 * diasAtraso`.
  - **Valor total devido:** `valorParcelaOriginal + Multa + JurosMora`.
- **Atualização:**
  - **Pagamento total:** Atualiza `dataPagamento`, `valorPago`, `multaAtraso`, `jurosMora` e status para "paga".
  - **Pagamento parcial (se configurado):** Registra o `valorPago`, calcula o `saldoDevedorParcela` (total devido - valor pago) e ajusta o saldoDevedor total do contrato proporcionalmente, mantendo status "parcialmente paga".
  - **Impacto no Contrato:** O saldoDevedor do contrato é reduzido pelo valor amortizado (parcela - juros), refletido na próxima consulta.

**Saída (Pagamento Parcial)**

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

### 8.1. Condições

- **Antes do Início:** Solicitação deve ocorrer antes da `dataInicioPagamento`, sem parcelas pagas.
- **Reembolso:** Se o valor foi liberado antecipadamente, exige devolução com juros proporcionais ao período entre liberação e cancelamento.

### 8.2. Processo

- **Validação:** Confirma que o contrato existe, está ativo e não tem parcelas pagas.
- **Cálculo de Reembolso (se aplicável):** `ValorReembolso = ValorEmprestimo * (1 + TaxaJurosMensal / 30) ^ DiasDesdeLiberacao`.
- **Atualização:** Altera o `statusContrato` para "cancelado" no banco de dados.

### 8.3. Requisição

```json
{
  "idCliente": "123.456.789-00",
  "idEmprestimo": "EMP-00123"
}
```

### 8.4. Saída (Com Reembolso)

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
Não mais necessária, pois a taxa inicial é fixa em 1,80% para todos os casos, ajustada apenas pelo prazo conforme TaxaJurosMensal = 0,018 + 0,00005 * (QuantidadeParcelas - 24). Caso mantida para configuração, pode armazenar apenas o incremento e o limite:

| Campo                | Tipo         | Descrição                                        | Exemplo          | Chave?     |
|----------------------|--------------|--------------------------------------------------|------------------|------------|
| taxaInicial          | DECIMAL(6,5) | Taxa inicial fixa                                | 0.018            | -          |
| incrementoMensal     | DECIMAL(6,5) | Incremento por mês acima de 24                   | 0.00005          | -          |
| tetoJuros            | DECIMAL(6,5) | Limite máximo de juros                           | 0.0214           | -          |
| prazoMinimo          | INT          | Prazo mínimo em meses                            | 24               | -          |
| prazoMaximo          | INT          | Prazo máximo em meses                            | 92               | -          |

### Notas:
- A tabela agora é simplificada, armazenando valores fixos globais em vez de taxas por vínculo e idade.

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

## 6. Tabela: ConfiguracoesLocais (Opcional)
Armazena as configurações específicas da instalação atual do sistema, permitindo que regras como idade máxima, taxa de juros inicial, prazos e margem consignável sejam definidas localmente para o banco em que o sistema está instalado.

| Campo                  | Tipo         | Descrição                                        | Exemplo          | Chave?     |
|-----------------------|--------------|--------------------------------------------------|------------------|------------|
| idConfig              | INT          | Identificador único (esperado apenas 1 registro) | 1                | Primária   |
| idadeMaxima           | INT          | Idade máxima permitida ao fim do contrato        | 80               | -          |
| taxaInicial           | DECIMAL(6,5) | Taxa de juros inicial fixa                       | 0.018            | -          |
| incrementoMensal      | DECIMAL(6,5) | Incremento na taxa por mês acima do prazo mínimo | 0.00005          | -          |
| tetoJuros             | DECIMAL(6,5) | Limite máximo de juros mensal                    | 0.0214           | -          |
| prazoMinimo           | INT          | Prazo mínimo em meses                            | 24               | -          |
| prazoMaximo           | INT          | Prazo máximo em meses                            | 92               | -          |
| carenciaMaxima        | INT          | Período máximo de carência em dias               | 60               | -          |
| margemConsignavelPercentual | DECIMAL(4,2) | Percentual da margem consignável (ex.: 0.35 = 35%) | 0.35          | -          |

