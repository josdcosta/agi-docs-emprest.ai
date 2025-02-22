# Documentação Emprest.AI

## Autores
- [@dalleth](https://github.com/dalleth-martinss)
- [@josdcosta](https://github.com/josdcosta)

## Referência
- [Planilha de cálculo](https://docs.google.com/spreadsheets/d/1Y_vrP424Qpyh_nWdp_xtSSbsdswpp4XKPIOVeIV9B4E/edit?usp=sharing)

---

# Documentação do Backend - Empréstimo Consignado

## 1. Objetivo do Backend
O backend será responsável por gerenciar solicitações de empréstimos consignados, verificando a existência de consignados anteriores no banco de dados, calculando a margem consignável (30% dos vencimentos líquidos menos parcelas existentes) e processando a concessão do empréstimo com base nos parâmetros fornecidos, incluindo cálculo de parcelas, validação da margem e regras de prazo ajustadas para que a idade final do cliente não exceda 80 anos, com parcelas em múltiplos de 12 meses a partir de 24.

---

## 2. Funcionalidades do Backend

### 2.1. Verificação Inicial
Antes de processar a solicitação de empréstimo:
1. **Consulta ao banco de dados**:
   - Verificar se o usuário (identificado pelo `idCliente`, que pode ser o CPF) já possui consignados registrados.
   - Obter os **vencimentos líquidos** do cliente (renda mensal após descontos).
   - Obter o valor total das **parcelas de consignados anteriores**, se existirem.
   - Obter a **idade do cliente** para aplicação das regras de prazo.

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
- **quantidadeParcelas**: Número de parcelas solicitado (deve ser múltiplo de 12, mínimo 24, ex.: `24`, `36`, `48`, etc.).
- **dataInicioPagamento**: Data do início dos pagamentos, sempre futura (ex.: `01/04/2025`).
- **taxaJurosMensal**: Taxa de juros nominal mensal em decimal (ex.: `0.015` para 1,5% ao mês).
- **tipoVinculo**: Tipo de vínculo do cliente (`"servidor_federal"`, `"servidor_estadual"`, `"servidor_municipal"`, ou `"aposentado"`).

**Parâmetros adicionais sugeridos**:
- **dataSolicitacao**: Data da solicitação do empréstimo (ex.: `22/02/2025`), para cálculos de carência.
- **seguros**: Valor opcional de seguros, se aplicável (ex.: `500.00`, padrão `0.00`).
- **tributos**: Valor opcional de tributos, se aplicável (ex.: `200.00`, padrão `0.00`).

### 2.3. Regras de Prazo Baseadas na Idade Máxima de 80 Anos
Com base na projeção da esperança de vida em 2025 (78 anos) e no limite máximo de 80 anos para quitação, os prazos máximos de parcelamento são definidos em múltiplos de 12 meses, começando por 24, conforme o tipo de vínculo e, para aposentados, ajustados pela idade do cliente:

- **Projeção da esperança de vida**:
  - Entre 2000 e 2019, a esperança de vida aumentou 4,8 anos (média de 0,25 anos/ano).
  - De 2019 a 2025 (6 anos): incremento estimado de `0,25 * 6 = 1,5 anos`.
  - Esperança de vida em 2025: `76,4 + 1,5 = 77,9 anos`, arredondado para **78 anos**.
  - Limite máximo ajustado: **80 anos** como idade final (idade inicial + prazo).

- **Regra final ajustada**:
  - **Servidores federais**: Até 96 meses (8 anos, máximo permitido).
    - Opções: 24, 36, 48, 60, 72, 84, 96 meses.
  - **Servidores estaduais**: Até 84 meses (7 anos).
    - Opções: 24, 36, 48, 60, 72, 84 meses.
  - **Servidores municipais**: Até 72 meses (6 anos).
    - Opções: 24, 36, 48, 60, 72 meses.
  - **Aposentados**:
    - Até 68 anos: 96 meses (8 anos, idade final ≤ 76 anos).
    - 69 a 72 anos: 84 meses (7 anos, idade final ≤ 79 anos).
    - 73 a 74 anos: 72 meses (6 anos, idade final ≤ 80 anos).
    - 75 a 76 anos: 48 meses (4 anos, idade final ≤ 80 anos).
    - 77 a 78 anos: 36 meses (3 anos, idade final ≤ 80 anos).
    - 79 anos: 24 meses (2 anos, idade final = 80 anos).
    - Acima de 79 anos: Empréstimo não permitido (idade final ultrapassaria 80 anos).

- **Validação do prazo**:
  - O backend verifica se `quantidadeParcelas` é múltiplo de 12 e maior ou igual a 24.
  - Calcula a idade final (`idadeInicial + prazo em anos`) e ajusta o prazo ao máximo permitido pelo vínculo e idade, garantindo que não exceda 80 anos.
  - Exemplo: Aposentado de 75 anos solicita 60 meses (não permitido, ajustado para 48 meses).

### 2.4. Cálculos Realizados
1. **Consulta de dados do cliente**:
   - Obter `vencimentos líquidos`, `parcelas anteriores` e `idade` do banco de dados com base no `idCliente`.

2. **Ajuste do prazo máximo**:
   - Aplicar as regras de prazo conforme o `tipoVinculo` e `idade`, limitando a idade final a 80 anos e ajustando para múltiplos de 12 a partir de 24.
   - Exemplo:
     - Aposentado, 75 anos: Prazo máximo = 48 meses (75 + 4 = 79 anos).
     - Solicitado 60 meses: Ajustado para 48 meses.

3. **Cálculo da parcela do consignado**:
   - Usando o método Price (parcelas fixas):
     - Fórmula: `Parcela = [ValorEmprestimo * TaxaJurosMensal] / [1 - (1 + TaxaJurosMensal)^(-QuantidadeParcelas)]`
     - Exemplo:
       - Valor: `10000.00`
       - Taxa: `0.015`
       - Parcelas: `24`
       - Parcela: `[10000 * 0.015] / [1 - (1 + 0.015)^(-24)] = 496.69`

4. **Ajuste com carência (opcional)**:
   - Se `dataSolicitacao` e `dataInicioPagamento` forem fornecidas:
     - Carência (em dias): `dataInicioPagamento - dataSolicitacao`
     - Taxa diária: `TaxaJurosMensal / 30`
     - Valor ajustado: `ValorEmprestimo * (1 + TaxaDiária * Carência)`

5. **Validação da margem**:
   - Comparar o valor da `Parcela` calculada com a `Margem`:
     - Se `Parcela <= Margem`: Prosseguir com a concessão.
     - Se `Parcela > Margem`: Retornar erro com mensagem.

### 2.5. Validações
- Verificar se o `idCliente` existe no banco de dados.
- Garantir que `dataInicioPagamento` seja futura em relação à data atual (`22/02/2025`).
- Validar que `valorEmprestimo`, `quantidadeParcelas`, e `taxaJurosMensal` sejam positivos.
- Verificar se `quantidadeParcelas` é múltiplo de 12 e ≥ 24.
- Ajustar `quantidadeParcelas` ao limite máximo conforme as regras de prazo, garantindo que `idadeInicial + (quantidadeParcelas / 12)` ≤ 80.
- Verificar se a parcela calculada cabe na margem consignável.

### 2.6. Saídas Geradas
- **Sucesso**:
  - Objeto JSON com:
    - `idCliente`: Identificador do cliente.
    - `valorEmprestimo`: Valor concedido.
    - `parcela`: Valor da parcela fixa.
    - `quantidadeParcelas`: Total de parcelas (ajustado, se necessário).
    - `dataInicioPagamento`: Data do primeiro pagamento.
    - `margemUtilizada`: Valor da margem utilizada (`parcela`).
    - `margemRestante`: Margem restante após o consignado (`Margem - Parcela`).
    - `prazoMaximoPermitido`: Prazo máximo conforme vínculo e idade.
   Exemplo:
    ```json
    {
      "idCliente": "123.456.789-00",
      "valorEmprestimo": 10000.00,
      "parcela": 496.69,
      "quantidadeParcelas": 24,
      "dataInicioPagamento": "01/04/2025",
      "margemUtilizada": 496.69,
      "margemRestante": 203.31,
      "prazoMaximoPermitido": 96
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
