# Documentações Emprest.AI 📄

## EMPRÉSTIMO CONSIGNADO E PESSOAL

### Índice
1. [Autores](#autores)
2. [Referências](#referências)
3. [Objetivo](#objetivo)
4. [Variáveis Configuráveis](#variáveis-configuráveis)
5. [Visão Geral do Funcionamento](#visão-geral-do-funcionamento)
6. [Dados Armazenados do Cliente](#dados-armazenados-do-cliente)
7. [Simulação de Empréstimo](#simulação-de-empréstimo)
8. [Concessão de Empréstimo](#concessão-de-empréstimo)
9. [Consulta de Dados de Empréstimo](#consulta-de-dados-de-empréstimo)
10. [Pagamento de Empréstimo](#pagamento-de-empréstimo)
11. [Refinanciamento de Empréstimo](#refinanciamento-de-empréstimo)
12. [Portabilidade de Empréstimo](#portabilidade-de-empréstimo)
13. [Elegibilidade](#elegibilidade)
14. [Cálculos](#cálculos)

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

### 2. Variáveis Configuráveis
Os parâmetros abaixo são ajustáveis no sistema Emprest.AI, podendo ser alterados conforme políticas internas, regulamentações ou condições comerciais:

| Variável                   | Descrição                                                 | Valor Padrão            |
|----------------------------|-----------------------------------------------------------|-------------------------|
| jurosMinimoPessoal         | Taxa mínima de juros mensal (Empréstimo Pessoal)          | 8,49% ao mês            |
| jurosMaximoPessoal         | Taxa máxima de juros mensal (Empréstimo Pessoal)          | 9,99% ao mês            |
| jurosMinimoConsignado      | Taxa mínima de juros mensal (Empréstimo Consignado, 24 meses) | 1,80% ao mês            |
| jurosMaximoConsignado      | Taxa máxima de juros mensal (Empréstimo Consignado, 92 meses) | 2,14% ao mês            |
| valorMinimoPessoal         | Valor mínimo do Empréstimo Pessoal                        | R$ 100,00               |
| valorMaximoPessoal         | Valor máximo do Empréstimo Pessoal                        | R$ 20.000,00            |
| valorMinimoConsignado      | Valor mínimo do Empréstimo Consignado                     | Depende de margemConsignavel (35%) |
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
| taxaMaximaSeguroAnual      | Taxa máxima anual permitida para seguro                   | 0,01 (1%)               |

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
#### 5.1. Requisição
##### Empréstimo Consignado
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
Consignado:
- Aplica 11.1.5. Tipo de Vínculo.
- Usa 11.1.2. Idade Máxima para calcular idade + quantidadeParcelas / 12.
- Aplica 11.1.3. Quantidade de Parcelas.
- Calcula dias de carência e aplica 11.1.6. Carência.  

Pessoal:
- Aplica 11.2.1. Valor do Empréstimo.
- Aplica 11.2.2. Quantidade de Parcelas.
- Aplica 11.2.3. Score de Crédito.
- Calcula dias de carência e aplica 11.2.5. Carência.

**Passo 3: Determinação da Capacidade de Pagamento**  
Consignado: Executa 12.1. Margem Consignável.  
Pessoal: Executa 12.2. Capacidade de Pagamento.

**Passo 4: Definição da Taxa de Juros**  
Consignado: Aplica 12.3. Taxa de Juros Mensal (Consignado) e verifica 11.1.4. Taxa de Juros.  
Pessoal: Aplica 12.3. Taxa de Juros Mensal (Pessoal) e verifica 11.2.4. Taxa de Juros.

**Passo 5: Cálculo do Custo do Seguro**  
Se `contratarSeguro = true`, aplica 12.4. Custo do Seguro.

**Passo 6: Cálculo do IOF**  
Executa 12.5. IOF.

**Passo 7: Cálculo do Valor Total Financiado**  
Aplica 12.6. Valor Total Financiado.

**Passo 8: Cálculo da Parcela Mensal**  
Executa 12.7. Parcela Mensal.

**Passo 9: Validação Final de Elegibilidade**  
Consignado: Aplica 11.1.1. Margem Consignável.  
Pessoal: Aplica 11.2.4. Capacidade de Pagamento.

**Passo 10: Retorno da Simulação**  
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
  "cetMensal": 0.0201,
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
  "parcelaMensal": 392.27,
  "cetMensal": 0.0940,
  "mensagem": "Simulação realizada com sucesso."
}
```

### 6. Concessão de Empréstimo
#### 6.1. Requisição
(Idêntica à Simulação)

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
#### 7.2. Processo Passo a Passo
- Consulta de Dados do Cliente: Valida `idCliente`. Se não encontrado, "Erro: Cliente não encontrado".
- Verificação do Empréstimo: Busca `idEmprestimo`. Se inválido, "Erro: Empréstimo não encontrado ou inválido".
- Recuperação dos Dados do Empréstimo: Obtém dados do contrato.
- Consulta do Histórico de Pagamentos: Verifica parcelas pagas e restantes.
- Cálculo do Saldo Devedor: Executa 12.8. Saldo Devedor.
- Retorno dos Dados: Compila e retorna as informações.

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
  "mensagem": "Consulta realizada com sucesso."
}
```

### 8. Pagamento de Empréstimo
#### 8.1. Requisição
##### Pagamento Parcial
```json
{
  "idCliente": "123.456.789-00",
  "idEmprestimo": "EMP001",
  "tipoPagamento": "parcial",
  "valorPagamento": 1000.00,
  "dataPagamento": "26/02/2025"
}
```
##### Pagamento Total
```json
{
  "idCliente": "123.456.789-00",
  "idEmprestimo": "EMP001",
  "tipoPagamento": "total",
  "dataPagamento": "26/02/2025"
}
```
#### 8.2. Processo Passo a Passo
- Consulta de Dados do Cliente: Valida `idCliente`.
- Verificação do Empréstimo: Busca `idEmprestimo`.
- Validação do Status do Empréstimo: Confirma se está ativo. Se quitado, "Erro: Empréstimo já liquidado".
- Consulta do Saldo Devedor Atual: Executa 12.8. Saldo Devedor.
- Processamento do Pagamento:
  - Total: Registra quitação total.
  - Parcial: Aplica o valor às parcelas mais antigas. Se insuficiente, "Erro: Valor insuficiente".
- Atualização do Contrato: Marca como "quitado" (total) ou ajusta parcelas (parcial).
- Retorno da Confirmação: Retorna status atualizado.

#### 8.3. Saída
##### Pagamento Parcial
```json
{
  "idCliente": "123.456.789-00",
  "idEmprestimo": "EMP001",
  "tipoPagamento": "parcial",
  "valorPagamento": 1000.00,
  "saldoDevedor": 8278.07,
  "mensagem": "Pagamento parcial registrado com sucesso."
}
```
##### Pagamento Total
```json
{
  "idCliente": "123.456.789-00",
  "idEmprestimo": "EMP001",
  "tipoPagamento": "total",
  "valorPagamento": 9278.07,
  "saldoDevedor": 0.00,
  "mensagem": "Empréstimo quitado com sucesso."
}
```

# 9. Refinanciamento de Empréstimo

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

1. Consulta de Dados do Cliente: Valida idCliente.
2. Verificação do Empréstimo Original: Busca idEmprestimoOriginal.
3. Validação do Status: Confirma se ativo.
4. Verificação de Elegibilidade:
   - Aplica 11.3.1. Percentual Mínimo Pago.
   - Consignado: 11.1.2, 11.1.3, 11.1.6.
   - Pessoal: 11.2.2, 11.2.5.
5. Cálculo do Saldo Devedor: Executa 12.8. Saldo Devedor.
6. Determinação da Capacidade:
   - Consignado: 12.1
   - Pessoal: 12.2
7. Definição da Taxa de Juros: 12.3.
8. Cálculo do Custo do Seguro: 12.4.
9. Cálculo do IOF: 12.5.
10. Cálculo do Valor Total Financiado: 12.6.
11. Cálculo da Nova Parcela: 12.7.
12. Validação Final:
    - Consignado: 11.1.1
    - Pessoal: 11.2.4
13. Registro do Refinanciamento: Cria novo contrato e marca o original como "refinanciado".

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
  "mensagem": "Refinanciamento realizado com sucesso."
}
```

# 10. Portabilidade de Empréstimo (Apenas Consignado)

## 10.1. Requisição

```json
{
  "idCliente": "123.456.789-00",
  "idEmprestimoOriginal": "EMP001",
  "bancoDestino": "Banco XYZ",
  "novaQuantidadeParcelas": 60,
  "contratarSeguro": true,
  "dataInicioPagamento": "01/04/2025"
}
```

## 10.2. Processo Passo a Passo

1. Consulta de Dados do Cliente: Valida idCliente.
2. Verificação do Empréstimo Original: Busca idEmprestimoOriginal.
3. Validação do Status: Confirma se ativo.
4. Verificação de Elegibilidade: Aplica 11.4.1, 11.4.2, 11.1.2, 11.1.3, 11.1.6.
5. Cálculo do Saldo Devedor: 12.8.
6. Determinação da Margem: 12.1.
7. Definição da Taxa de Juros: 12.3.
8. Cálculo do Custo do Seguro: 12.4.
9. Cálculo do IOF: 12.5.
10. Cálculo do Valor Total Financiado: 12.6.
11. Cálculo da Nova Parcela: 12.7.
12. Validação Final: 11.1.1.
13. Registro da Portabilidade: Marca como "portado" e notifica o banco destino.

## 10.3. Saída

```json
{
  "idCliente": "123.456.789-00",
  "idEmprestimoOriginal": "EMP001",
  "bancoDestino": "Banco XYZ",
  "saldoDevedorOriginal": 9278.07,
  "novaQuantidadeParcelas": 60,
  "taxaJurosMensal": 0.0198,
  "custoSeguro": 250.00,
  "iof": 337.30,
  "valorTotalFinanciado": 9865.37,
  "parcelaMensal": 248.67,
  "mensagem": "Portabilidade realizada com sucesso."
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

## 11.4. Portabilidade (Consignado)

### 11.4.1. Parcelas em Dia

Sem parcelas vencidas.

### 11.4.2. Aceitação do Banco Destino

bancoDestino deve aceitar a portabilidade.

# 12. Cálculos

## 12.1. Margem Consignável (Consignado)

margemMaxima = remuneracaoLiquida * 0,35

## 12.2. Capacidade de Pagamento (Pessoal)

capacidadeMaxima = remuneracaoLiquida * 0,30

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
