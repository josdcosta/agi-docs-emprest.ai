> **📄 Documentação 1**
## Autores
- [@Dalleth Martins](https://github.com/dalleth-martinss)
- [@João Pedro de C. Formigari](https://github.com/Joao-Formigari)
- [@Josué Davi da Costa](https://github.com/josdcosta)
- [@Carollina Guedes](https://github.com/CarollinaGuedes)
- [@Victor Augusto Ferreira](https://github.com/Victor-augusto-ferreira)

## Referência
- [Planilha de cálculo](https://docs.google.com/spreadsheets/d/1Y_vrP424Qpyh_nWdp_xtSSbsdswpp4XKPIOVeIV9B4E/edit?usp=sharing)

# Documentação - Empréstimo Empresarial

## 1. Objetivo
O backend gerencia solicitações de empréstimos empresariais, verificando histórico de crédito, calculando a capacidade de pagamento com base no faturamento líquido anual da empresa e processando a concessão. As taxas de juros variam conforme o porte da empresa e o prazo do empréstimo, com um incremento de 0,005 (0,5%) a cada 12 meses acima de 12, sem teto fixo como no consignado, mas alinhado às práticas de mercado. Os prazos são flexíveis, começando em 12 meses e indo até 120 meses (10 anos), dependendo da análise de risco. Um custo opcional de seguro pode ser incluído. O cálculo das parcelas utiliza o **Sistema de Amortização Constante (SAC)**, onde a amortização é fixa e os juros diminuem ao longo do tempo. Se a quantidade de parcelas não for fornecida, retorna opções de parcelamento com valores.

## 2. Funcionalidades

### 2.1. Entrada de Dados
Parâmetros recebidos:
- **idEmpresa**: Ex.: `12.345.678/0001-90` (CNPJ).
- **valorEmprestimo**: Ex.: `50000.00`.
- **quantidadeParcelas**: Opcional, mínimo 12, máximo 120 (ex.: `12`, `24`, `36`, ..., `120`). Se omitido, retorna opções.
- **dataInicioPagamento**: Futura (ex.: `01/04/2025`).
- **contratarSeguro**: `true` ou `false`.

**Opcional**:
- **dataSolicitacao**: Ex.: `22/02/2025`.

### 2.2. Verificação Inicial
1. **Consulta ao banco de dados**:
   - Verificar empréstimos anteriores via `idEmpresa` (CNPJ).
   - Obter **faturamento líquido anual**, **dívidas existentes** e **porte da empresa**.
   - **porteEmpresa**: `micro`, `pequena`, `média`, `grande`.
   - *Para refinanciamento*: Consultar `idEmprestimoOriginal` para obter saldo devedor atual, status de pagamento e detalhes do contrato original (taxa de juros, prazo restante).

2. **Cálculo da capacidade de pagamento**:
   - Fórmula: `Capacidade = (Faturamento líquido anual * 0.20) / 12 - Parcelas de dívidas existentes`
   - Exemplo: Faturamento líquido anual `600.000,00`, dívidas existentes `5.000,00` → Capacidade = `(600.000,00 * 0.20) / 12 - 5.000,00 = 5.000,00`

### 2.3. Regras de Taxas de Juros e Prazos
Taxas baseiam-se em `porteEmpresa` e `quantidadeParcelas`, com incremento de `0,005 (0,5%) a cada 12 meses` acima de 12. Prazos variam de 12 a 120 meses:

- **Fórmula da taxa**:
  - `TaxaJurosMensal = TaxaBase + 0.005 * ((QuantidadeParcelas - 12) / 12)`
  - Não há teto fixo, mas segue práticas de mercado.

- **Com seguro**:
  - **Microempresa**: Taxa base 1,8% (0.018), máximo 48 meses (12, 24, 36, 48).
  - **Pequena empresa**: Taxa base 1,6% (0.016), máximo 72 meses (12, 24, ..., 72).
  - **Média empresa**: Taxa base 1,4% (0.014), máximo 96 meses (12, 24, ..., 96).
  - **Grande empresa**: Taxa base 1,2% (0.012), máximo 120 meses (12, 24, ..., 120).

- **Sem seguro (taxa base + 0,3%)**:
  - **Microempresa**: Taxa base 2,1% (0.021), máximo 48 meses.
  - **Pequena empresa**: Taxa base 1,9% (0.019), máximo 72 meses.
  - **Média empresa**: Taxa base 1,7% (0.017), máximo 96 meses.
  - **Grande empresa**: Taxa base 1,5% (0.015), máximo 120 meses.

- **Refinanciamento**:
  - A taxa de juros é ajustada com base nas condições atuais da empresa (`porteEmpresa`, `faturamento`), usando a mesma fórmula de incremento.

### 2.4. Cálculos Realizados
1. **Consulta de dados**:
   - Obter `faturamento líquido anual`, `dívidas existentes` e `porte da empresa`.
   - *Para refinanciamento*: Obter `saldoDevedor` e status de pagamento via `idEmprestimoOriginal`.

2. **Determinação de taxa e prazo**:
   - `TaxaJurosMensal = TaxaBase + 0.005 * ((QuantidadeParcelas - 12) / 12)`.

3. **Custo do seguro (se contratado)**:
   - Fórmula: `CustoSeguro = 0.05 * ValorEmprestimo`
   - Exemplo: Valor 50.000,00 → `CustoSeguro = 2.500,00`

4. **Cálculo do IOF**:
   - `IOF_Fixo = 0.0038 * ValorEmprestimo`
   - `IOF_Variavel = 0.000082 * ValorEmprestimo * min(NúmeroDeDias, 365)`
   - `IOF_Total = IOF_Fixo + IOF_Variavel`
   - Nota: NúmeroDeDias é a diferença entre dataInicioPagamento e dataFimContrato.

5. **Ajuste com carência (juros compostos)**:
   - `ValorInicial = ValorEmprestimo + IOF_Total + CustoSeguro`
   - `ValorTotalFinanciado = ValorInicial * (1 + TaxaJurosMensal / 30) ^ Carência`

6. **Cálculo da parcela (SAC)**:
   - `Amortização = ValorTotalFinanciado / QuantidadeParcelas`
   - Para cada parcela: `Juros = SaldoDevedorAnterior * TaxaJurosMensal`
   - `Parcela = Amortização + Juros`
   - Saldo devedor diminui linearmente: `SaldoDevedor = SaldoDevedorAnterior - Amortização`

7. **Cálculo da taxa efetiva mensal (CET)**:
   - Resolve numericamente o fluxo de caixa para encontrar a taxa que iguala o valor presente líquido:
     - `ValorEmprestimo = SUM(Parcela[i] / (1 + TaxaEfetivaMensal)^i)` para i de 1 a QuantidadeParcelas.
   - Inclui IOF e seguro no ValorTotalFinanciado.

8. **Refinanciamento**:
   - `SaldoDevedor = ValorOriginal - AmortizaçõesPagas`
   - `ValorTotalFinanciado = SaldoDevedor + novoValorEmprestimo (se fornecido) + CustoSeguro (se aplicável) + IOF`
   - Nova parcela calculada via SAC.

9. **Validação da capacidade**:
   - Compara a primeira parcela (maior no SAC) com a capacidade de pagamento. Se exceder, retorna erro.

10. **Retorno de opções (se quantidadeParcelas omitida)**:
    - Gera lista de parcelamentos possíveis (12 até o máximo permitido), com taxa, primeira parcela, custo total e taxa efetiva mensal.

### 2.5. Validações
- Verificar `idEmpresa`, `dataInicioPagamento` futura (ex.: `22/02/2025`).
- Validar `valorEmprestimo` positivo; se `quantidadeParcelas` fornecida, deve ser ≥ 12 e ≤ máximo por porte.
- Ajustar ao máximo permitido conforme `porteEmpresa`.
- Verificar capacidade de pagamento.
- **Refinanciamento**:
  - Verificar se pelo menos 20% das parcelas do empréstimo original foram pagas.

### 2.6. Saídas Geradas
- **Com `quantidadeParcelas` fornecida (sem seguro)**:
```json
{
  "idEmpresa": "12.345.678/0001-90",
  "valorEmprestimo": 50000.00,
  "primeiraParcela": 2291.67,
  "ultimaParcela": 1041.67,
  "quantidadeParcelas": 24,
  "dataInicioPagamento": "01/04/2025",
  "dataFimContrato": "01/04/2027",
  "taxaJurosMensal": 0.015,
  "taxaEfetivaMensal": 0.0158,
  "contratarSeguro": false,
  "custoSeguro": 0.00,
  "iof": 789.95,
  "carencia": 30,
  "valorTotalFinanciado": 51594.41,
  "capacidadeUtilizada": 2291.67,
  "capacidadeRestante": 2708.33,
  "prazoMaximoPermitido": 120
}
```
- **Com quantidadeParcelas fornecida (com seguro)**:
```json
{
  "idEmpresa": "12.345.678/0001-90",
  "valorEmprestimo": 50000.00,
  "primeiraParcela": 2395.83,
  "ultimaParcela": 1145.83,
  "quantidadeParcelas": 24,
  "dataInicioPagamento": "01/04/2025",
  "dataFimContrato": "01/04/2027",
  "taxaJurosMensal": 0.012,
  "taxaEfetivaMensal": 0.0135,
  "contratarSeguro": true,
  "custoSeguro": 2500.00,
  "iof": 789.95,
  "carencia": 30,
  "valorTotalFinanciado": 54094.41,
  "capacidadeUtilizada": 2395.83,
  "capacidadeRestante": 2604.17,
  "prazoMaximoPermitido": 120
}
```
- **Cálculo da Amortização (SAC)**:
 - Exemplo com ValorTotalFinanciado = 54.094,41, QuantidadeParcelas = 24, TaxaJurosMensal = 0.012:
 - Amortização = 54.094,41 / 24 = 2.253,93

| Parcela | Saldo Devedor Anterior | Juros    | Amortização | Parcela  | Saldo Devedor Restante |
|---------|-----------------------|----------|-------------|----------|------------------------|
| 1       | 54.094,41             | 649,13   | 2.253,93    | 2.903,06 | 51.840,48              |
| 2       | 51.840,48             | 622,09   | 2.253,93    | 2.876,02 | 49.586,55              |
| 3       | 49.586,55             | 595,04   | 2.253,93    | 2.848,97 | 47.332,62              |
| 4       | 47.332,62             | 568,00   | 2.253,93    | 2.821,93 | 45.078,69              |
| 5       | 45.078,69             | 540,95   | 2.253,93    | 2.794,88 | 42.824,76              |
| 6       | 42.824,76             | 513,90   | 2.253,93    | 2.767,83 | 40.570,83              |
| 7       | 40.570,83             | 486,85   | 2.253,93    | 2.740,78 | 38.316,90              |
| 8       | 38.316,90             | 459,80   | 2.253,93    | 2.713,73 | 36.062,97              |
| 9       | 36.062,97             | 432,76   | 2.253,93    | 2.686,69 | 33.809,04              |
| ...     | ...                   | ...      | ...         | ...      | ...                    |
| 23      | 4.507,95              | 54,10    | 2.253,93    | 2.308,03 | 2.254,02               |
| 24      | 2.254,02              | 27,05    | 2.253,93    | 2.280,98 | 0,09                   |

 - **Erro de Capacidade Excedida**:
 ```json
 {
  "erro": "Primeira parcela solicitada (2395.83) excede a capacidade de pagamento disponível (2000.00)"
}
```
- **Sem quantidadeParcelas**:
 - Sem quantidadeParcelas:
 - Retorna opções possíveis de parcelamento.
 ### 2.7. Refinanciamento de Empréstimo Empresarial
 - O refinanciamento permite renegociar um empréstimo empresarial existente no mesmo banco.

- **Entrada de Dados Adicionais**:
```json
{
 "idEmprestimoOriginal: Ex.: EMP123456."
 "novoValorEmprestimo: Opcional, ex.: 10000.00."
 "novaQuantidadeParcelas: Ex.: 36."
}
```
- **Processo**:

- Consultar idEmprestimoOriginal: saldo devedor, parcelas pagas, taxa original.
- Verificar elegibilidade: mínimo de 20% das parcelas pagas.
- **Calcular novo empréstimo**:
 - `ValorTotalFinanciado = SaldoDevedor + novoValorEmprestimo (se fornecido) + CustoSeguro (se aplicável) + IOF`
 - Nova taxa ajustada conforme porteEmpresa.
 - Nova parcela via SAC.
 - Validar capacidade de pagamento.
 - **Saída**:
 ```json
 {
  "idEmpresa": "12.345.678/0001-90",
  "idEmprestimoOriginal": "EMP123456",
  "saldoDevedorOriginal": 30000.00,
  "novoValorEmprestimo": 10000.00,
  "primeiraParcela": 1483.33,
  "ultimaParcela": 583.33,
  "novaQuantidadeParcelas": 36,
  "taxaJurosMensal": 0.015,
  "taxaEfetivaMensal": 0.0162,
  "contratarSeguro": true,
  "custoSeguro": 2000.00,
  "iof": 315.98,
  "valorTotalFinanciado": 42315.98,
  "capacidadeUtilizada": 1483.33,
  "capacidadeRestante": 3516.67
}
```
## 3.  Estrutura dos Cálculos
### 3.1. Fórmulas
- **Capacidade de Pagamento**:
  - `Capacidade = (Faturamento líquido anual * 0.20) / 12 - Parcelas de dívidas existentes`
- **Taxa**:
  - `TaxaJurosMensal = TaxaBase + 0.005 * ((QuantidadeParcelas - 12) / 12)`
- **Seguro**:
  - `CustoSeguro = 0.05 * ValorEmprestimo`
- **Parcela (SAC)**:
  - `Amortização = ValorTotalFinanciado / QuantidadeParcelas`
  - `Parcela = Amortização + (SaldoDevedorAnterior * TaxaJurosMensal)`
- **Taxa Efetiva Mensal (CET)**:
  - **Resolve**: `ValorEmprestimo = SUM(Parcela[i] / (1 + TaxaEfetivaMensal)^i)`
### 3.2. Exemplo Prático
- **Entrada**:
```json
{
  "idEmpresa": "12.345.678/0001-90"
  "valorEmprestimo": 50000.00
  "quantidadeParcelas": 24
  "contratarSeguro": true
}
```
- **Dados Coletados**:
  - porteEmpresa: "grande", faturamento: `600.000,00`, dívidas: `5.000,00`
- **Cálculos**:
  - `Capacidade = (600.000 * 0.20) / 12 - 5.000 = 5.000,00`
  - `TaxaJurosMensal = 0.012 + 0.005 * ((24 - 12) / 12) = 0.017`
  - `CustoSeguro = 0.05 * 50.000 = 2.500,00`
  - `IOF = 789.95, ValorTotalFinanciado = 54.094,41`
  - `Amortização = 54.094,41 / 24 = 2.253,93`
  - `Primeira parcela: 2.253,93 + (54.094,41 * 0.017) = 3.173,53`
- **Saída**: Veja JSON em 2.6 (com seguro).
## 4. Observações
  - Sem `quantidadeParcelas`, retorna opções viáveis até o prazo máximo por porte.
  - Seguro é fixo (5% do valor) e incluído no `ValorTotalFinanciado`.
  - O SAC resulta em parcelas decrescentes, com a primeira sendo a mais alta, usada para validar a capacidade.
  - A taxa efetiva mensal (CET) reflete o custo total (juros, IOF, seguro).
