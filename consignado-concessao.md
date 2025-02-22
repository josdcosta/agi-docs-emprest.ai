# Documentação Emprest.AI

## Autores
- [@dalleth](https://github.com/dalleth-martinss)
- [@josdcosta](https://github.com/josdcosta)
- [@CarollinaGuedes](https://github.com/CarollinaGuedes)

## Referência
- [Planilha de cálculo](https://docs.google.com/spreadsheets/d/1Y_vrP424Qpyh_nWdp_xtSSbsdswpp4XKPIOVeIV9B4E/edit?usp=sharing)

# Documentação - Empréstimo Consignado

## 1. Objetivo
O backend gerencia solicitações de empréstimos consignados, verificando consignados anteriores, calculando a margem consignável (30% dos vencimentos líquidos menos parcelas existentes) e processando a concessão. Taxas de juros variam por vínculo e idade, aumentando 0,025 a cada 12 meses acima de 24, com teto de 1,80%. Prazos são múltiplos de 12 a partir de 24, limitando a idade final a 80 anos. Um custo fixo de seguro pode ser incluído opcionalmente. Se `quantidadeParcelas` não for fornecida, retorna os possíveis parcelamentos com valores.

---

## 2. Funcionalidades

### 2.1. Entrada de Dados
Parâmetros recebidos:
- **idCliente**: Ex.: `"123.456.789-00"`.
- **valorEmprestimo**: Ex.: `10000.00`.
- **quantidadeParcelas**: Opcional, múltiplo de 12, mínimo 24 (ex.: `24`, `36`, etc.). Se omitido, retorna opções.
- **dataInicioPagamento**: Futura (ex.: `01/04/2025`).
- **contratarSeguro**: `true` ou `false`.

**Opcional**:
- **dataSolicitacao**: Ex.: `22/02/2025`.

### 2.2. Verificação Inicial
1. **Consulta ao banco de dados**:
   - Verificar consignados via `idCliente` (ex.: CPF).
   - Obter **vencimentos líquidos**, **parcelas anteriores** e **idade**.
   - **tipoVinculo**: `"servidor_federal"`, `"servidor_estadual"`, `"servidor_municipal"`, `"aposentado"`.

2. **Cálculo da margem consignável**:
   - Fórmula: `Margem = (Vencimentos líquidos * 0.3) - Parcelas anteriores`
   - Exemplo: Vencimentos `5000.00`, parcelas `800.00` → Margem = `700.00`


### 2.3. Regras de Taxas de Juros e Prazos
Taxas baseiam-se em `tipoVinculo`, `idade` e `contratarSeguro`, com incremento de **0,025 a cada 12 meses** acima de 24, teto 1,80%. Prazos são múltiplos de 12, mínimo 24, idade final ≤ 80:

- **Fórmula da taxa**:
  - `TaxaJurosMensal = TaxaBase + 0.025 * ((QuantidadeParcelas - 24) / 12)`
  - Limite: `min(TaxaCalculada, 1.80%)`

- **Com seguro**:
  - **Servidores federais**: Taxa base 1,3% (0.013), máximo 96 meses (24, 36, ..., 96).
  - **Servidores estaduais**: Taxa base 1,4% (0.014), máximo 84 meses (24, 36, ..., 84).
  - **Servidores municipais**: Taxa base 1,5% (0.015), máximo 72 meses (24, 36, ..., 72).
  - **Aposentados**:
    - Até 66 anos: Taxa base 1,3%, máximo 96 meses.
    - 67 a 70 anos: Taxa base 1,45%, máximo 84 meses.
    - 71 a 74 anos: Taxa base 1,45%, máximo 72 meses.
    - 75 a 78 anos: Taxa base 1,6%, máximo 48 meses (24, 36, 48).
    - 79 anos: Taxa base 1,6%, máximo 24 meses (24).

- **Sem seguro (taxa base + 0,2%, teto 1,80%)**:
  - **Servidores federais**: Taxa base 1,5%, máximo 96 meses.
  - **Servidores estaduais**: Taxa base 1,6%, máximo 84 meses.
  - **Servidores municipais**: Taxa base 1,7%, máximo 72 meses.
  - **Aposentados**:
    - Até 66 anos: Taxa base 1,5%, máximo 96 meses.
    - 67 a 70 anos: Taxa base 1,65%, máximo 84 meses.
    - 71 a 74 anos: Taxa base 1,65%, máximo 72 meses.
    - 75 a 78 anos: Taxa base 1,8%, máximo 48 meses (teto).
    - 79 anos: Taxa base 1,8%, máximo 24 meses (teto).
    - Acima de 79 anos: Não permitido.

### 2.4. Cálculos Realizados

### 2.4. Cálculos Realizados

1. **Consulta de dados**:
   - Obter `vencimentos líquidos`, `parcelas anteriores`, `idade`.

2. **Determinação de taxa e prazo**:
   - `TaxaJurosMensal = TaxaBase + 0,025 * ((QuantidadeParcelas - 24) / 12)`, limitada a 1,80%.
   - Taxa base depende de `tipoVinculo`, `idade` e `contratarSeguro` (ver seção 2.3).

3. **Custo do seguro (se contratado)**:
   - Fórmula: `CustoSeguro = [0,04 + (0,001 * idade)] * ValorEmprestimo`
   - Exemplo: Idade 75, Valor 10.000 → `CustoSeguro = 1.150,00`

4. **Cálculo do IOF**:
   - `IOF_Fixo = 0,0038 * ValorEmprestimo`
   - `IOF_Variavel = 0,00008219 * ValorEmprestimo * NúmeroDeDias`
   - `IOF_Total = min(IOF_Fixo + IOF_Variavel, 0,03 * ValorEmprestimo)`

5. **Ajuste com carência (juros compostos)**:
   - `ValorInicial = ValorEmprestimo + IOF_Total + CustoSeguro`
   - `ValorTotalFinanciado = ValorInicial * (1 + TaxaJurosMensal / 30) ^ Carência`

6. **Cálculo da parcela (Price)**:
   - `Parcela = [ValorTotalFinanciado * TaxaJurosMensal] / [1 - (1 + TaxaJurosMensal)^(-QuantidadeParcelas)]`

7. **Validação da margem**:
   - Compara `Parcela` com `Margem`. Se exceder, retorna erro.

8. **Retorno de opções (se `quantidadeParcelas` omitida)**:
   - Gera lista de parcelamentos possíveis (24 até o máximo permitido), com taxa, parcela e custo total.

### 2.5. Validações
- Verificar `idCliente`, `dataInicioPagamento` futura (`22/02/2025`).
- Validar `valorEmprestimo` positivo; se `quantidadeParcelas` fornecida, deve ser múltiplo de 12 e ≥ 24.
- Ajustar ao máximo permitido, respeitando idade final ≤ 80.
- Verificar margem.

### 2.6. Saídas Geradas
- **Com `quantidadeParcelas` fornecida**:
  - Exemplo (com seguro):
    ```json
    {
      "idCliente": "123.456.789-00",
      "valorEmprestimo": 10000.00,
      "parcela": 350.13,
      "quantidadeParcelas": 48,
      "dataInicioPagamento": "01/04/2025",
      "dataFimContrato": "01/04/2029",
      "taxaJurosMensal": 0.0165,
      "contratarSeguro": true,
      "custoSeguro": 1150.00,
      "iof": 157.99,
      "carencia": 30,
      "valorTotalFinanciado": 11496.87,
      "margemUtilizada": 350.13,
      "margemRestante": 349.87,
      "prazoMaximoPermitido": 48
    }
    
   Exemplo (parcela excede margem):
   ```json
      {
        "erro": "Parcela solicitada (496.69) excede a margem consignável disponível (700.00)"
      }
   ```
   
   
   Exemplo (prazo inválido):
   ```json
      {
        "erro": "Quantidade de parcelas (60) excede o prazo máximo permitido (48) para aposentado de 75 anos (idade final não pode ultrapassar 80 anos)"
      }
   ```
   
   
   Exemplo (prazo inválido):
   ```json
      {
        "erro": "Quantidade de parcelas (30) deve ser múltiplo de 12, começando por 24"
      }
   ```
   
   
   Exemplo (prazo inválido):
   ```json
      {
        "erro": "Empréstimo não permitido para cliente com 80 anos ou mais (idade final ultrapassaria 80 anos)"
      }
   ```

   Sem quantidadeParcelas:
      Retorna opções possíveis de parcelamento.


3. Estrutura dos Cálculos

## 3.1. Fórmulas

- **Margem**:  
  `Margem = (Vencimentos líquidos * 0.3) - Parcelas anteriores`

- **Taxa**:  
  `TaxaJurosMensal = TaxaBase + 0.025 * ((QuantidadeParcelas - 24) / 12)`, com teto de 1,80%.

- **Seguro**:  
  `CustoSeguro = [0,04 + (0,001 * idade)] * ValorEmprestimo`

- **Parcela**:  
  `Parcela = [ValorTotalFinanciado * TaxaJurosMensal] / [1 - (1 + TaxaJurosMensal)^(-QuantidadeParcelas)]`

## 3.2. Exemplo Prático

### Entrada:
- **idCliente**: "123.456.789-00"  
- **valorEmprestimo**: 10.000,00  
- **quantidadeParcelas**: OMITIDA  
- **contratarSeguro**: true

### Coletados do banco após envio da solicitação:
- **tipoVinculo**: "aposentado"
- **idade**: 75  
- **Vencimentos**: 5.000,00  
- **Parcelas anteriores**: 800,00  

### Cálculos:
- **Margem**:  
  `Margem = (5.000,00 * 0.3) - 800,00 = 700,00`

- **Regra**:  
  Aposentado de 75 anos, com seguro → Taxa base = 1,6%, máximo 48 meses.  
  **Opções**: 24, 36, 48 meses.
  
   ```json
   {
     "idCliente": "123.456.789-00",
     "valorEmprestimo": 10000.00,
     "dataInicioPagamento": "01/04/2025",
     "tipoVinculo": "aposentado",
     "contratarSeguro": true,
     "prazoMaximoPermitido": 48,
     "opcoesParcelamento": [
       {
         "quantidadeParcelas": 24,
         "taxaJurosMensal": 0.016,
         "parcela": 588.92,
         "iof": 98.00,
         "custoSeguro": 1150.00,
         "valorTotalFinanciado": 11435.27,
         "margemUtilizada": 588.92,
         "margemRestante": 111.08,
         "dataFimContrato": "01/04/2027"
       },
       {
         "quantidadeParcelas": 36,
         "taxaJurosMensal": 0.01625,
         "parcela": 415.06,
         "iof": 128.00,
         "custoSeguro": 1150.00,
         "valorTotalFinanciado": 11466.77,
         "margemUtilizada": 415.06,
         "margemRestante": 284.94,
         "dataFimContrato": "01/04/2028"
       },
       {
         "quantidadeParcelas": 48,
         "taxaJurosMensal": 0.0165,
         "parcela": 350.13,
         "iof": 157.99,
         "custoSeguro": 1150.00,
         "valorTotalFinanciado": 11496.87,
         "margemUtilizada": 350.13,
         "margemRestante": 349.87,
         "dataFimContrato": "01/04/2029"
       }
     ]
   }
   ```

## 4. Observações

- Sem `quantidadeParcelas`, retorna opções viáveis até o prazo máximo, respeitando margem e idade.
- Taxas aumentam 0,025% a cada 12 meses, com teto de 1,80%.
- Seguro é estimado como média; amortização detalha valores exatos.

   ### Explicações
      1. **Nova Funcionalidade**: Se `quantidadeParcelas` é omitida, o backend gera `opcoesParcelamento` com todas as opções de 24 até o máximo permitido, incluindo taxa ajustada, parcela, custo do seguro (se aplicável) e impacto na margem.
      2. **Regras Mantidas**: Todas as validações (idade ≤ 80, margem, múltiplos de 12) são aplicadas às opções retornadas.
      3. **Exemplo**: Para um aposentado de 75 anos, retorna 24, 36 e 48 meses, com taxas crescentes e detalhes completos.
      
      Se precisar ajustar o formato do retorno ou testar outros casos, é só pedir!



