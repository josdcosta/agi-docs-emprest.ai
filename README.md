# Documentação Emprest.AI

## Autores
- [@dalleth](https://github.com/dalleth-martinss)
- [@josdcosta](https://github.com/josdcosta)

## Referência
- [Planilha de cálculo](https://docs.google.com/spreadsheets/d/1Y_vrP424Qpyh_nWdp_xtSSbsdswpp4XKPIOVeIV9B4E/edit?usp=sharing)

---

# Empréstimo Consignado

## 1. Objetivo do Backend
O backend será responsável por gerenciar solicitações de empréstimos consignados, verificando consignados anteriores no banco de dados, calculando a margem consignável (30% dos vencimentos líquidos menos parcelas existentes) e processando a concessão com taxas de juros determinadas pelo sistema. Inclui a opção de seguro prestamista (0,2% ao mês sobre o saldo), ajustando as taxas e prazos em múltiplos de 12 meses a partir de 24, com idade final limitada a 80 anos.

---

## 2. Funcionalidades do Backend

### 2.1. Verificação Inicial
Antes de processar a solicitação:
1. **Consulta ao banco de dados**:
   - Verificar se o usuário (`idCliente`, ex.: CPF) tem consignados registrados.
   - Obter os **vencimentos líquidos** (renda mensal após descontos).
   - Obter o valor total das **parcelas de consignados anteriores**.
   - Obter a **idade do cliente** para determinar taxa e prazo.

2. **Cálculo da margem consignável**:
   - Fórmula: `Margem = (Vencimentos líquidos * 0.3) - Soma das parcelas anteriores`
   - Exemplo:
     - Vencimentos: `5000.00`
     - Parcelas anteriores: `800.00`
     - Margem: `(5000 * 0.3) - 800 = 700.00`

### 2.2. Entrada de Dados para Concessão
Parâmetros recebidos:
- **idCliente**: Identificador único (ex.: `"123.456.789-00"`).
- **valorEmprestimo**: Valor solicitado (ex.: `10000.00`).
- **quantidadeParcelas**: Número de parcelas (múltiplo de 12, mínimo 24, ex.: `24`, `36`, etc.).
- **dataInicioPagamento**: Data do primeiro pagamento, futura (ex.: `01/04/2025`).
- **tipoVinculo**: Tipo de vínculo (`"servidor_federal"`, `"servidor_estadual"`, `"servidor_municipal"`, `"aposentado"`).
- **contratarSeguro**: Booleano indicando se o cliente opta pelo seguro prestamista (ex.: `true` ou `false`).

**Parâmetros adicionais sugeridos**:
- **dataSolicitacao**: Data da solicitação (ex.: `22/02/2025`), para carência.

### 2.3. Regras de Taxas de Juros e Prazos
Taxas e prazos são definidos pelo sistema com base em `tipoVinculo`, `idade` e `contratarSeguro`. O seguro prestamista opcional adiciona 0,2% ao mês sobre o saldo devedor. Prazos são múltiplos de 12 meses (mínimo 24), com idade final ≤ 80 anos:

- **Com seguro prestamista (0,2% ao mês sobre o saldo)**:
  - **Servidores federais**: 
    - Taxa: 1,3% (0.013).
    - Prazo máximo: 96 meses.
    - Opções: 24, 36, 48, 60, 72, 84, 96.
  - **Servidores estaduais**: 
    - Taxa: 1,4% (0.014).
    - Prazo máximo: 84 meses.
    - Opções: 24, 36, 48, 60, 72, 84.
  - **Servidores municipais**: 
    - Taxa: 1,5% (0.015).
    - Prazo máximo: 72 meses.
    - Opções: 24, 36, 48, 60, 72.
  - **Aposentados**:
    - Até 66 anos: 
      - Taxa: 1,3% (0.013).
      - Prazo máximo: 96 meses (idade final ≤ 74).
      - Opções: 24, 36, 48, 60, 72, 84, 96.
    - 67 a 70 anos: 
      - Taxa: 1,45% (0.0145).
      - Prazo máximo: 84 meses (idade final ≤ 77).
      - Opções: 24, 36, 48, 60, 72, 84.
    - 71 a 74 anos: 
      - Taxa: 1,45% (0.0145).
      - Prazo máximo: 72 meses (idade final ≤ 80).
      - Opções: 24, 36, 48, 60, 72.
    - 75 a 78 anos: 
      - Taxa: 1,6% (0.016).
      - Prazo máximo: 48 meses (idade final ≤ 79).
      - Opções: 24, 36, 48.
    - 79 anos: 
      - Taxa: 1,6% (0.016).
      - Prazo máximo: 24 meses (idade final = 80).
      - Opção: 24.

- **Sem seguro (taxas aumentam 0,2%, teto de 1,80%)**:
  - **Servidores federais**: 
    - Taxa: 1,5% (0.015).
    - Prazo máximo: 96 meses.
    - Opções: 24, 36, 48, 60, 72, 84, 96.
  - **Servidores estaduais**: 
    - Taxa: 1,6% (0.016).
    - Prazo máximo: 84 meses.
    - Opções: 24, 36, 48, 60, 72, 84.
  - **Servidores municipais**: 
    - Taxa: 1,7% (0.017).
    - Prazo máximo: 72 meses.
    - Opções: 24, 36, 48, 60, 72.
  - **Aposentados**:
    - Até 66 anos: 
      - Taxa: 1,5% (0.015).
      - Prazo máximo: 96 meses (idade final ≤ 74).
      - Opções: 24, 36, 48, 60, 72, 84, 96.
    - 67 a 70 anos: 
      - Taxa: 1,65% (0.0165).
      - Prazo máximo: 84 meses (idade final ≤ 77).
      - Opções: 24, 36, 48, 60, 72, 84.
    - 71 a 74 anos: 
      - Taxa: 1,65% (0.0165).
      - Prazo máximo: 72 meses (idade final ≤ 80).
      - Opções: 24, 36, 48, 60, 72.
    - 75 a 78 anos: 
      - Taxa: 1,8% (0.018, teto).
      - Prazo máximo: 48 meses (idade final ≤ 79).
      - Opções: 24, 36, 48.
    - 79 anos: 
      - Taxa: 1,8% (0.018, teto).
      - Prazo máximo: 24 meses (idade final = 80).
      - Opção: 24.
    - Acima de 79 anos: Não permitido.

- **Validação**:
  - Determina taxa e prazo máximo com base em `tipoVinculo`, `idade` e `contratarSeguro`.
  - Verifica se `quantidadeParcelas` é múltiplo de 12 e ≥ 24.
  - Ajusta ao prazo máximo permitido, garantindo `idadeInicial + (quantidadeParcelas / 12)` ≤ 80.

### 2.4. Cálculos Realizados
1. **Consulta de dados**:
   - Obter `vencimentos líquidos`, `parcelas anteriores` e `idade` via `idCliente`.

2. **Determinação de taxa e prazo**:
   - Aplica regras conforme `tipoVinculo`, `idade` e `contratarSeguro`.
   - Exemplo: Aposentado, 75 anos, com seguro: Taxa = 1,6% (0.016), Prazo máximo = 48 meses.

3. **Cálculo da parcela (método Price)**:
   - Fórmula: `Parcela = [ValorEmprestimo * TaxaJurosMensal] / [1 - (1 + TaxaJurosMensal)^(-QuantidadeParcelas)]`
   - Com seguro: Adiciona 0,2% ao mês sobre o saldo devedor na tabela de amortização (não na parcela fixa inicial).
   - Exemplo (sem seguro):
     - Valor: `10000.00`
     - Taxa: `0.018` (1,8%, aposentado 75 anos, sem seguro)
     - Parcelas: `48`
     - Parcela: `[10000 * 0.018] / [1 - (1 + 0.018)^(-48)] = 297.71`

4. **Ajuste com carência (opcional)**:
   - Carência (dias): `dataInicioPagamento - dataSolicitacao`
   - Taxa diária: `TaxaJurosMensal / 30`
   - Valor ajustado: `ValorEmprestimo * (1 + TaxaDiária * Carência)`

5. **Cálculo do seguro (se contratado)**:
   - Taxa do seguro: 0,2% ao mês (0.002) sobre o saldo devedor.
   - Para cada parcela na amortização:
     - `CustoSeguro = SaldoDevedorAnterior * 0.002`
     - Exemplo (primeira parcela):
       - Saldo inicial: `10000.00`
       - Seguro: `10000 * 0.002 = 20.00`

6. **Validação da margem**:
   - Compara `Parcela` (sem seguro) ou `Parcela + CustoSeguro médio estimado` (com seguro) com a `Margem`.

### 2.5. Validações
- Verificar existência de `idCliente`.
- Garantir `dataInicioPagamento` futura (`22/02/2025` como referência).
- Validar `valorEmprestimo` e `quantidadeParcelas` positivos, este último múltiplo de 12 e ≥ 24.
- Ajustar `quantidadeParcelas` ao máximo permitido, respeitando idade final ≤ 80.
- Verificar se `Parcela` (ou `Parcela + Seguro`) cabe na margem.

### 2.6. Saídas Geradas
- **Sucesso**:
  - Objeto JSON com:
    - `idCliente`
    - `valorEmprestimo`
    - `parcela`: Parcela fixa (sem incluir seguro na base).
    - `quantidadeParcelas`: Ajustado, se necessário.
    - `dataInicioPagamento`
    - `taxaJurosMensal`: Taxa aplicada.
    - `contratarSeguro`: Booleano.
    - `custoSeguroEstimado`: Média estimada do seguro por parcela (se aplicável).
    - `margemUtilizada`: Inclui seguro, se contratado.
    - `margemRestante`
    - `prazoMaximoPermitido`
  - Exemplo (com seguro):
    ```json
    {
      "idCliente": "123.456.789-00",
      "valorEmprestimo": 10000.00,
      "parcela": 287.92,
      "quantidadeParcelas": 48,
      "dataInicioPagamento": "01/04/2025",
      "taxaJurosMensal": 0.016,
      "contratarSeguro": true,
      "custoSeguroEstimado": 12.50,
      "margemUtilizada": 300.42,
      "margemRestante": 399.58,
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
