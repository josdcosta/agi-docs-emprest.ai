# Documentação Emprest.AI

## Autores
- [@dalleth](https://github.com/dalleth-martinss)
- [@josdcosta](https://github.com/josdcosta)

## Referência
- [Planilha de cálculo](https://docs.google.com/spreadsheets/d/1Y_vrP424Qpyh_nWdp_xtSSbsdswpp4XKPIOVeIV9B4E/edit?usp=sharing)

---

# Documentação do Backend - Empréstimo Consignado

## 1. Objetivo do Backend
O backend será responsável por gerenciar solicitações de empréstimos consignados, verificando a existência de consignados anteriores no banco de dados, calculando a margem consignável (30% dos vencimentos líquidos menos parcelas existentes) e processando a concessão do empréstimo com base nos parâmetros fornecidos, incluindo cálculo de parcelas e validação da margem.

---

## 2. Funcionalidades do Backend

### 2.1. Verificação Inicial
Antes de processar a solicitação de empréstimo:
1. **Consulta ao banco de dados**:
   - Verificar se o usuário (identificado pelo `idCliente`, que pode ser o CPF) já possui consignados registrados.
   - Obter os **vencimentos líquidos** do cliente (renda mensal após descontos).
   - Obter o valor total das **parcelas de consignados anteriores**, se existirem.

2. **Cálculo da margem consignável**:
   - Fórmula: `Margem = (Vencimentos líquidos * 0.3) - Soma das parcelas de consignados anteriores`
   - Exemplo:
     - Vencimentos líquidos: `5000.00`
     - Parcelas anteriores: `800.00`
     - Margem: `(5000.00 * 0.3) - 800.00 = 1500.00 - 800.00 = 700.00`

### 2.2. Entrada de Dados para Concessão
Os seguintes parâmetros serão recebidos na solicitação de concessão:
- **idCliente**: Identificador único do cliente (ex.: CPF, formato `string`, como `"123.456.789-00"`).
- **valorEmprestimo**: Valor solicitado do empréstimo (ex.: `10000.00`).
- **quantidadeParcelas**: Número de parcelas (ex.: `24`).
- **dataInicioPagamento**: Data do início dos pagamentos, sempre futura (ex.: `01/04/2025`).
- **taxaJurosMensal**: Taxa de juros nominal mensal em decimal (ex.: `0.015` para 1,5% ao mês).

**Parâmetros adicionais sugeridos**:
- **dataSolicitacao**: Data da solicitação do empréstimo (ex.: `22/02/2025`), para cálculos de carência.
- **seguros**: Valor opcional de seguros, se aplicável (ex.: `500.00`, padrão `0.00`).
- **tributos**: Valor opcional de tributos, se aplicável (ex.: `200.00`, padrão `0.00`).

### 2.3. Cálculos Realizados
1. **Consulta de dados do cliente**:
   - Obter `vencimentos líquidos` e `parcelas anteriores` do banco de dados com base no `idCliente`.

2. **Cálculo da parcela do consignado**:
   - Usando o método Price (parcelas fixas):
     - Fórmula: `Parcela = [ValorEmprestimo * TaxaJurosMensal] / [1 - (1 + TaxaJurosMensal)^(-QuantidadeParcelas)]`
     - Exemplo:
       - Valor: `10000.00`
       - Taxa: `0.015`
       - Parcelas: `24`
       - Parcela: `[10000 * 0.015] / [1 - (1 + 0.015)^(-24)] = 150 / 0.302 = 496.69`

3. **Ajuste com carência (opcional)**:
   - Se `dataSolicitacao` e `dataInicioPagamento` forem fornecidas:
     - Carência (em dias): `dataInicioPagamento - dataSolicitacao`
     - Taxa diária: `TaxaJurosMensal / 30`
     - Valor ajustado: `ValorEmprestimo * (1 + TaxaDiária * Carência)`

4. **Validação da margem**:
   - Comparar o valor da `Parcela` calculada com a `Margem`:
     - Se `Parcela <= Margem`: Prosseguir com a concessão.
     - Se `Parcela > Margem`: Retornar erro com mensagem.

### 2.4. Validações
- Verificar se o `idCliente` existe no banco de dados.
- Garantir que `dataInicioPagamento` seja futura em relação à data atual (`22/02/2025`).
- Validar que `valorEmprestimo`, `quantidadeParcelas`, e `taxaJurosMensal` sejam positivos.
- Verificar se a parcela calculada cabe na margem consignável.

### 2.5. Saídas Geradas
- **Sucesso**:
  - Objeto JSON com:
    - `idCliente`: Identificador do cliente.
    - `valorEmprestimo`: Valor concedido.
    - `parcela`: Valor da parcela fixa.
    - `quantidadeParcelas`: Total de parcelas.
    - `dataInicioPagamento`: Data do primeiro pagamento.
    - `margemUtilizada`: Valor da margem utilizada (`parcela`).
    - `margemRestante`: Margem restante após o consignado (`Margem - Parcela`).
  - Exemplo:
    ```json
    {
      "idCliente": "123.456.789-00",
      "valorEmprestimo": 10000.00,
      "parcela": 496.69,
      "quantidadeParcelas": 24,
      "dataInicioPagamento": "01/04/2025",
      "margemUtilizada": 496.69,
      "margemRestante": 203.31
    }
