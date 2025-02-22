

Documentação Emprest.AI


## Autores

- [@dalleth](https://github.com/dalleth-martinss)
- [@josdcosta](https://github.com/josdcosta)


# Documentação do Backend - Empréstimo Consignado

## 1. Objetivo do Backend
O backend será responsável por gerenciar e calcular os detalhes de um **empréstimo consignado**, incluindo valores de parcelas, amortização do principal, juros, saldo devedor e valor presente das parcelas, com base em entradas como valor recebido, taxa de juros, carência, seguros, tributos e número de parcelas.

---

## 2. Funcionalidades do Backend

### 2.1. Entrada de Dados
O backend receberá os seguintes dados como entrada:
- **Valor recebido**: Valor líquido liberado ao cliente (ex.: `26000.00`).
- **Data de liberação do crédito**: Data em que o crédito foi liberado (ex.: `07/11/2022`).
- **Data da primeira parcela**: Data do primeiro pagamento (ex.: `02/01/2023`).
- **Data do último vencimento**: Data do vencimento da última parcela (ex.: `31/03/2028`).
- **Taxa de juros nominal mensal**: Taxa de juros mensal em decimal (ex.: `0.0155` para 1,55%).
- **Número de parcelas**: Quantidade total de parcelas (ex.: `64`).
- **Seguros**: Valor total dos seguros (ex.: `1888.43`).
- **Tributos**: Valor total dos tributos (ex.: `940.68`).
- **Alíquota fixa**: Alíquota para cálculo de tributos (ex.: `0.0038`).

### 2.2. Cálculos Realizados
O backend executará os seguintes cálculos:

1. **Taxa de juros nominal diária**:
   - Fórmula: `Taxa diária = Taxa mensal / 30`
   - Exemplo: `0.0155 / 30 = 0.000513`.

2. **Período de carência (em dias)**:
   - Fórmula: `Carência = Data da primeira parcela - Data de liberação do crédito`
   - Exemplo: `02/01/2023 - 07/11/2022 = 56 dias`.

3. **Empréstimo + Seguros**:
   - Fórmula: `Empréstimo + Seguros = Valor recebido + Seguros`
   - Exemplo: `26000 + 1888.43 = 27888.43`.

4. **Empréstimo + Seguros + Tributos**:
   - Fórmula: `Empréstimo + Seguros + Tributos = Empréstimo + Seguros + Tributos`
   - Exemplo: `27888.43 + 940.68 = 28829.11`.

5. **Empréstimo + Seguros + Tributos + Carência**:
   - Fórmula: `Total = (Empréstimo + Seguros + Tributos) * (1 + Taxa diária * Carência)`
   - Exemplo: `28829.11 * (1 + 0.000513 * 56) = 29668.83`.

6. **Cálculo da parcela fixa (método Price)**:
   - Fórmula: `Parcela = [Total * Taxa mensal] / [1 - (1 + Taxa mensal)^(-Número de parcelas)]`
   - Exemplo: `[29668.83 * 0.0155] / [1 - (1 + 0.0155)^(-64)] = 734.22`.

7. **Tabela de amortização**:
   - Para cada parcela, calcular:
     - **Juros**: `Juros = Saldo devedor anterior * Taxa mensal`
     - **Principal**: `Principal = Parcela - Juros`
     - **Saldo devedor**: `Saldo devedor = Saldo devedor anterior - Principal`
     - **Valor presente da parcela**: `VP = Parcela / (1 + Taxa mensal)^(n)`, onde `n` é o número da parcela (contado a partir de 1).
   - Exemplo (Parcela 1):
     - Saldo devedor inicial: `29668.83`
     - Juros: `29668.83 * 0.0155 = 459.87`
     - Principal: `734.22 - 459.87 = 274.35`
     - Saldo devedor: `29668.83 - 274.35 = 29394.48`
     - Valor presente: `734.22 / (1 + 0.0155)^1 = 734.22`.

8. **Cálculo de tributos**:
   - Fórmula:
     - Se o contrato ultrapassa 1 ano: `Tributos = Alíquota fixa * 365 * Valor recebido`
     - Se não ultrapassa 1 ano: `Tributos = Alíquota fixa * (Data fim - Data início) * Valor recebido`
   - Exemplo: `0.0038 * 365 * 26000 = 940.68`.

### 2.3. Saídas Geradas
O backend retornará:
- **Resumo inicial**:
  - Valor recebido
  - Empréstimo + Seguros + Tributos + Carência
  - Parcela fixa
  - Taxa de juros diária
  - Período de carência
- **Tabela de amortização**:
  - Para cada parcela:
    - Número da parcela
    - Data de vencimento
    - Valor da parcela
    - Principal
    - Juros
    - Saldo devedor
    - Valor presente da parcela

### 2.4. Validações
O backend deve:
- Verificar se as datas são válidas e seguem a ordem cronológica (`Data de liberação < Data da primeira parcela < Data do último vencimento`).
- Garantir que a taxa de juros, número de parcelas, valor recebido, seguros e tributos sejam valores positivos.
- Validar que o cálculo da parcela resulta em um valor finito e positivo.

---

## 3. Estrutura dos Cálculos

### 3.1. Fórmulas Detalhadas
1. **Taxa de Juros Diária**:
   - `Taxa diária = Taxa nominal mensal / 30`
   - Objetivo: Converter a taxa mensal para uso em cálculos de carência.

2. **Cálculo do Total com Carência**:
   - `Total = (Valor recebido + Seguros + Tributos) * (1 + Taxa diária * Carência)`
   - Objetivo: Ajustar o montante inicial pelo período de carência.

3. **Parcela Fixa (Método Price)**:
   - `Parcela = [Total * Taxa mensal] / [1 - (1 + Taxa mensal)^(-Número de parcelas)]`
   - Objetivo: Determinar o valor fixo das parcelas.

4. **Amortização (por parcela)**:
   - `Juros = Saldo devedor anterior * Taxa mensal`
   - `Principal = Parcela - Juros`
   - `Novo saldo devedor = Saldo devedor anterior - Principal`
   - Objetivo: Detalhar a evolução do saldo devedor ao longo das parcelas.

5. **Valor Presente**:
   - `VP = Parcela / (1 + Taxa mensal)^(n)`
   - Objetivo: Calcular o valor presente de cada parcela em relação à data inicial.

### 3.2. Exemplo Prático
- **Entrada**:
  - Valor recebido: `26000.00`
  - Taxa mensal: `0.0155`
  - Parcelas: `64`
  - Carência: `56 dias`
  - Seguros: `1888.43`
  - Tributos: `940.68`
- **Saída**:
  - Parcela fixa: `734.22`
  - Total com carência: `29668.83`
  - Tabela de amortização (primeira linha):
    - Parcela 1: `{data: "02/01/2023", valor: 734.22, principal: 274.35, juros: 459.87, saldo: 29394.48, vp: 734.22}`

---

## 4. Observações
- Os cálculos seguem o método de amortização **Price** (parcelas fixas), resultando em valores consistentes (ex.: `734.22`).
- As datas devem ser tratadas como objetos ou valores numéricos (ex.: diferença em dias) para facilitar os cálculos.
- O backend deve suportar arredondamentos consistentes (ex.: 2 casas decimais) para valores monetários.

---

Essa documentação descreve as funcionalidades e cálculos necessários para o backend. Está pronta para guiar a implementação!
