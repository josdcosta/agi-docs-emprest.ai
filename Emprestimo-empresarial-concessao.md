# Documentação - Empréstimo para Empresas

## 1. Objetivo
O backend gerencia solicitações de empréstimos para empresas, utilizando o Sistema de Amortização Constante (SAC), onde a amortização é fixa e os juros diminuem ao longo do tempo. O sistema utiliza um score de crédito fornecido por um analisador de risco externo para definir taxas de juros e limites de crédito, processa a concessão e calcula parcelas. Taxas variam conforme o score, porte da empresa e prazo, com um teto de 3,5% ao mês (baseado em práticas de mercado). Prazos são flexíveis entre 12 e 60 meses, com opção de carência. Seguro é opcional, e o sistema retorna opções de parcelamento se o prazo não for especificado.

# Documentação - Empréstimo para Empresas

## 2. Funcionalidades

### 2.1. Entrada de Dados
Parâmetros recebidos:
- **idEmpresa**: Ex.: `"12.345.678/0001-99"` (CNPJ).
- **valorEmprestimo**: Ex.: `50000.00`.
- **quantidadeParcelas**: Opcional, entre 12 e 60 meses (ex.: `12`, `24`, `36`). Se omitido, retorna opções.
- **dataInicioPagamento**: Futura (ex.: `01/04/2025`).
- **contratarSeguro**: `true` ou `false`.
- **carencia**: Opcional, em dias (ex.: `30`, máximo 90).
- **scoreCredito**: Fornecido pelo analisador de risco, entre 0 e 1000 (ex.: `750`).

**Opcional**:
- **dataSolicitacao**: Ex.: `22/02/2025`.

### 2.2. Verificação Inicial
1. **Consulta ao banco de dados**:
   - Verificar informações via `idEmpresa` (CNPJ).
   - Obter **faturamento anual**, **dívidas atuais**, **tempo de operação** (em anos) e **porte** (`"micro"`, `"pequena"`, `"média"`, `"grande"`).
   - Receber **scoreCredito** do analisador de risco.

2. **Limite de crédito**:
   - Fórmula: `Limite = Faturamento anual * FatorScore * 0.3`
   - `FatorScore`: Varia de 0,5 a 1,5 com base no `scoreCredito` (ver 2.3).
   - Exemplo: Faturamento `1.000.000,00`, `scoreCredito` 750 (FatorScore 1,5) → `Limite = 450.000,00`.

### 2.3. Regras de Taxas de Juros e Prazos
As taxas de juros variam conforme o `scoreCredito` (fornecido), porte da empresa e prazo, com incremento de 0,01 a cada 12 meses acima de 12, teto de 3,5% ao mês.

- **FatorScore** (baseado no `scoreCredito`):
  - Score 0-400: 0,5 (alto risco).
  - Score 401-700: 1,0 (médio risco).
  - Score 701-1000: 1,5 (baixo risco).

- **Taxas de juros**:
  - Fórmula: `TaxaJurosMensal = TaxaBase + 0,01 * ((QuantidadeParcelas - 12) / 12)`
  - Limite: `min(TaxaCalculada, 3,5%)`
  - **Com seguro**:
    - Micro: Taxa base 2,0%, máximo 36 meses.
    - Pequena: Taxa base 1,8%, máximo 48 meses.
    - Média: Taxa base 1,6%, máximo 60 meses.
    - Grande: Taxa base 1,4%, máximo 60 meses.
  - **Sem seguro (taxa base + 0,3%)**:
    - Micro: Taxa base 2,3%, máximo 36 meses.
    - Pequena: Taxa base 2,1%, máximo 48 meses.
    - Média: Taxa base 1,9%, máximo 60 meses.
    - Grande: Taxa base 1,7%, máximo 60 meses.

### 2.4. Cálculos Realizados (SAC)
1. **Consulta de dados**:
   - Obter faturamento, dívidas, porte e `scoreCredito`.

2. **Determinação de taxa e prazo**:
   - `TaxaJurosMensal` ajustada por porte, `scoreCredito` e `quantidadeParcelas`.

3. **Custo do seguro (se contratado)**:
   - Fórmula: `CustoSeguro = 0,05 * ValorEmprestimo`
   - Exemplo: Valor 50.000 → `CustoSeguro = 2.500,00`.

4. **Cálculo do IOF**:
   - `IOF_Fixo = 0,0038 * ValorEmprestimo`
   - `IOF_Variavel = 0,000082 * ValorEmprestimo * min(NúmeroDeDias, 365)`
   - `IOF_Total = IOF_Fixo + IOF_Variavel`
   - Exemplo: Valor 50.000, 48 meses (1.460 dias) → `IOF = 190 + 1.495,40 = 1.685,40`.

5. **Ajuste com carência**:
   - `ValorInicial = ValorEmprestimo + IOF_Total + CustoSeguro`
   - `ValorTotalFinanciado = ValorInicial * (1 + TaxaJurosMensal / 30) ^ Carência`

6. **Cálculo das parcelas (SAC)**:
   - `Amortização = ValorTotalFinanciado / QuantidadeParcelas`
   - `Juros = SaldoDevedor * TaxaJurosMensal`
   - `Parcela = Amortização + Juros`
   - Exemplo: Valor 50.000, 24 meses, taxa 2,0%, sem carência/seguro:
     - Amortização = 50.000 / 24 = 2.083,33
     - Parcela 1: Juros = 50.000 * 0,02 = 1.000 → Parcela = 3.083,33
     - Parcela 2: Saldo = 47.916,67, Juros = 958,33 → Parcela = 3.041,66.

7. **Taxa efetiva mensal (CET)**:
   - Resolve numericamente o fluxo de caixa para refletir custo total (juros, IOF, seguro).

8. **Validação do limite**:
   - Compara `ValorEmprestimo` com `Limite`. Se exceder, retorna erro.

9. **Retorno de opções (se `quantidadeParcelas` omitida)**:
   - Lista opções de 12 a 60 meses (respeitando máximo por porte), com taxa, parcelas e CET.

### 2.5. Validações
- Verificar `idEmpresa`, `dataInicioPagamento` futura.
- Validar `valorEmprestimo` positivo e dentro do limite de crédito.
- `quantidadeParcelas` entre 12 e 60, respeitando máximo por porte.
- `carencia` ≤ 90 dias.
- `scoreCredito` entre 0 e 1000.

### 2.6. Saídas Geradas
- **Com `quantidadeParcelas` fornecida**:
  ```json
  {
    "idEmpresa": "12.345.678/0001-99",
    "valorEmprestimo": 50000.00,
    "quantidadeParcelas": 24,
    "dataInicioPagamento": "01/04/2025",
    "dataFimContrato": "01/04/2027",
    "taxaJurosMensal": 0.02,
    "taxaEfetivaMensal": 0.021,
    "contratarSeguro": false,
    "custoSeguro": 0.00,
    "iof": 789.50,
    "carencia": 30,
    "valorTotalFinanciado": 51666.67,
    "scoreCredito": 750,
    "parcelas": [
      {"numero": 1, "valor": 3083.33, "juros": 1000.00, "amortizacao": 2083.33},
      {"numero": 2, "valor": 3041.66, "juros": 958.33, "amortizacao": 2083.33},
      {"numero": 24, "valor": 2124.99, "juros": 41.66, "amortizacao": 2083.33}
    ],
    "limiteCredito": 450000.00
  }
  ```
  
  - **Com seguro**:
  ```json
   {
     "idEmpresa": "12.345.678/0001-99",
     "valorEmprestimo": 50000.00,
     "quantidadeParcelas": 24,
     "dataInicioPagamento": "01/04/2025",
     "taxaJurosMensal": 0.018,
     "taxaEfetivaMensal": 0.0205,
     "contratarSeguro": true,
     "custoSeguro": 2500.00,
     "iof": 839.50,
     "carencia": 30,
     "valorTotalFinanciado": 54166.67,
     "scoreCredito": 750,
     "parcelas": [
       {"numero": 1, "valor": 3232.50, "juros": 975.00, "amortizacao": 2256.94},
       {"numero": 24, "valor": 2297.49, "juros": 40.55, "amortizacao": 2256.94}
     ],
     "limiteCredito": 450000.00
   }
  ```

  **Erro (limite excedido):**
  ```json
   {
     "erro": "Valor solicitado (500000.00) excede o limite de crédito disponível (450000.00)"
   }
  ```

  **Sem quantidadeParcelas:**
  ```json
   {
     "idEmpresa": "12.345.678/0001-99",
     "valorEmprestimo": 50000.00,
     "dataInicioPagamento": "01/04/2025",
     "contratarSeguro": true,
     "scoreCredito": 750,
     "opcoesParcelamento": [
       {"quantidadeParcelas": 12, "taxaJurosMensal": 0.018, "parcelaInicial": 5142.36, "parcelaFinal": 4234.72, "custoTotal": 56538.24},
       {"quantidadeParcelas": 24, "taxaJurosMensal": 0.019, "parcelaInicial": 3282.50, "parcelaFinal": 2347.49, "custoTotal": 62897.76},
       {"quantidadeParcelas": 36, "taxaJurosMensal": 0.020, "parcelaInicial": 2388.89, "parcelaFinal": 1416.67, "custoTotal": 69250.02}
     ]
   }
  ```
# Documentação - Empréstimo para Empresas

## 3. Estrutura dos Cálculos

### 3.1. Fórmulas
- **Limite de crédito**:  
  `Limite = Faturamento anual * FatorScore * 0.3`

- **Taxa**:  
  `TaxaJurosMensal = TaxaBase + 0,01 * ((QuantidadeParcelas - 12) / 12)`, teto 3,5%.

- **Seguro**:  
  `CustoSeguro = 0,05 * ValorEmprestimo`

- **Parcela (SAC)**:  
  `Amortização = ValorTotalFinanciado / QuantidadeParcelas`  
  `Juros = SaldoDevedor * TaxaJurosMensal`  
  `Parcela = Amortização + Juros`

- **Taxa Efetiva Mensal (CET)**:  
  Resolve: `ValorEmprestimo = Σ (Parcela_t / (1 + CET)^t)` para t = 1 a n.

### 3.2. Exemplo Prático
- **Entrada**:  
  `idEmpresa`: "12.345.678/0001-99", `valorEmprestimo`: 50.000,00, `quantidadeParcelas`: 24, `contratarSeguro`: true, `scoreCredito`: 750.

- **Dados coletados**:  
  Faturamento: 1.000.000,00, Porte: "pequena".

- **Cálculos**:  
  - Limite: `1.000.000 * 1,5 * 0,3 = 450.000,00` (FatorScore 1,5 para score 750)  
  - Taxa: `1,8% + 0,01 * (12 / 12) = 1,9%`  
  - Seguro: `0,05 * 50.000 = 2.500,00`  
  - IOF: `190 + 1.495,40 = 1.685,40`  
  - ValorTotalFinanciado: `50.000 + 1.685,40 + 2.500 = 54.185,40`  
  - Amortização: `54.185,40 / 24 = 2.257,73`  
  - Parcela 1: Juros = `54.185,40 * 0,019 = 1.029,52`, Parcela = `3.287,25`.

# Documentação - Empréstimo para Empresas

## 4. Observações
- O SAC reduz os juros ao longo do tempo, beneficiando empresas com fluxo de caixa inicial menor.
- O score de crédito é central para definir limite e taxa, refletindo risco real.
- Seguro e carência são opcionais, impactando o `ValorTotalFinanciado`.
