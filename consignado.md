# Documentações Emprest.AI

> **📄 Documentação 1**
## Autores
- [@Dalleth Martins](https://github.com/dalleth-martinss)
- [@Josué Davi da Costa](https://github.com/josdcosta)
- [@Carollina Guedes](https://github.com/CarollinaGuedes)
- [@Victor Augusto Ferreira](https://github.com/Victor-augusto-ferreira)

## Referência
- [Planilha de cálculo](https://docs.google.com/spreadsheets/d/1Y_vrP424Qpyh_nWdp_xtSSbsdswpp4XKPIOVeIV9B4E/edit?usp=sharing)

# Documentação - Empréstimo Consignado

## 1. Objetivo
O backend gerencia solicitações de empréstimos consignados, verificando consignados anteriores, calculando a margem consignável (35% da remuneração líquida menos parcelas de outros empréstimos existentes) e processando a concessão. Taxas de juros variam por vínculo e idade, aumentando 0,0025 a cada 12 meses acima de 24, com teto de 2,14% conforme regulamentação do Banco Central. Prazos são múltiplos de 12 a partir de 24, limitando a idade final a 80 anos. Um custo fixo de seguro pode ser incluído opcionalmente. Se `quantidadeParcelas` não for fornecida, retorna os possíveis parcelamentos com valores. As regras seguem a Lei 10.820/2003 e regulamentações do Banco Central e INSS.

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

#### 2.2. Verificação Inicial
1. **Consulta ao banco de dados**:
   - Verificar consignados via `idCliente` (ex.: CPF).
   - Obter **remuneração líquida**, **parcelas anteriores de empréstimos** e **idade**.
   - **tipoVinculo**: `"servidor_federal"`, `"servidor_estadual"`, `"servidor_municipal"`, `"aposentado"`.
   - *Para refinanciamento e portabilidade*: Consultar `idEmprestimoOriginal` para obter saldo devedor atual, status de pagamento e detalhes do contrato original (taxa de juros, prazo restante).

2. **Cálculo da margem consignável**:
   - Fórmula: `Margem = (Remuneração líquida * 0.35) - Parcelas anteriores de empréstimos`
   - Exemplo: Remuneração líquida `5000.00`, parcelas de empréstimos `800.00` → Margem = `(5000.00 * 0.35) - 800.00 = 950.00`
#### 2.3. Regras de Taxas de Juros e Prazos
Taxas baseiam-se em `tipoVinculo`, `idade` e `contratarSeguro`, com incremento de **0,0025 a cada 12 meses** acima de 24, teto 2,14% (limite do Banco Central para consignados). Prazos são múltiplos de 12, mínimo 24, idade final ≤ 80:

- **Fórmula da taxa**:
  - `TaxaJurosMensal = TaxaBase + 0.0025 * ((QuantidadeParcelas - 24) / 12)`
  - Limite: `min(TaxaCalculada, 2.14%)`

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

- **Sem seguro (taxa base + 0,2%, teto 2,14%)**:
  - **Servidores federais**: Taxa base 1,5%, máximo 96 meses.
  - **Servidores estaduais**: Taxa base 1,6%, máximo 84 meses.
  - **Servidores municipais**: Taxa base 1,7%, máximo 72 meses.
  - **Aposentados**:
    - Até 66 anos: Taxa base 1,5%, máximo 96 meses.
    - 67 a 70 anos: Taxa base 1,65%, máximo 84 meses.
    - 71 a 74 anos: Taxa base 1,65%, máximo 72 meses.
    - 75 a 78 anos: Taxa base 1,8%, máximo 48 meses.
    - 79 anos: Taxa base 1,8%, máximo 24 meses.
    - Acima de 79 anos: Não permitido.

- **Refinanciamento**:
  - A taxa de juros pode ser ajustada com base nas condições atuais do cliente (`tipoVinculo`, `idade`), respeitando o teto de 2,14%. Usa a mesma fórmula de incremento de 0,0025 por cada 12 meses acima de 24.

- **Portabilidade**:
  - A taxa de juros é definida pelo banco de destino, informada na entrada ou baseada em uma tabela específica do banco receptor, respeitando o teto de 2,14%.

#### 2.4. Cálculos Realizados
1. **Consulta de dados**:
   - Obter `vencimentos líquidos`, `parcelas anteriores`, `idade`.
   - *Para refinanciamento e portabilidade*: Obter `saldoDevedor` e status de pagamento via `idEmprestimoOriginal`.

2. **Determinação de taxa e prazo**:
   - `TaxaJurosMensal = TaxaBase + 0,0025 * ((QuantidadeParcelas - 24) / 12)`, limitada a 2,14%.
   - Taxa base depende de `tipoVinculo`, `idade` e `contratarSeguro` (ver seção 2.3).

3. **Custo do seguro (se contratado)**:
   - Fórmula: `CustoSeguro = [0,04 + (0,001 * idade)] * ValorEmprestimo`
   - Exemplo: Idade 75, Valor 10.000 → `CustoSeguro = 1.150,00`

4. **Cálculo do IOF**:
   - `IOF_Fixo = 0,0038 * ValorEmprestimo`
   - `IOF_Variavel = 0,000082 * ValorEmprestimo * min(NúmeroDeDias, 365)`
   - `IOF_Total = IOF_Fixo + IOF_Variavel`
   - Nota: `NúmeroDeDias` é a diferença em dias entre `dataInicioPagamento` e `dataFimContrato`.

5. **Ajuste com carência (juros compostos)**:
   - `ValorInicial = ValorEmprestimo + IOF_Total + CustoSeguro`
   - `ValorTotalFinanciado = ValorInicial * (1 + TaxaJurosMensal / 30) ^ Carência`

6. **Cálculo da parcela (Price)**:
   - `Parcela = [ValorTotalFinanciado * TaxaJurosMensal] / [1 - (1 + TaxaJurosMensal)^(-QuantidadeParcelas)]`

7. **Cálculo da taxa efetiva mensal (CET)**:
   - Fórmula: Resolve numericamente a equação do fluxo de caixa para encontrar a taxa que iguala o valor presente líquido:
     - `ValorEmprestimo = Parcela * [(1 - (1 + TaxaEfetivaMensal)^(-QuantidadeParcelas)) / TaxaEfetivaMensal]`
   - Inclui IOF e seguro no `ValorTotalFinanciado`.

8. **Refinanciamento**:
   - `SaldoDevedor = ValorOriginal - AmortizaçõesPagas`
   - `ValorTotalFinanciado = SaldoDevedor + novoValorEmprestimo (se fornecido) + CustoSeguro (se aplicável) + IOF`
   - Nova parcela calculada via fórmula de Price (item 6).

9. **Portabilidade**:
   - Obter `SaldoDevedor` do banco original.
   - `ValorTotalFinanciado = SaldoDevedor + CustoSeguro (se aplicável) + IOF`
   - Nova parcela calculada com a taxa do banco de destino via fórmula de Price (item 6).

10. **Validação da margem**:
    - Compara `Parcela` com `Margem`. Se exceder, retorna erro.

11. **Retorno de opções (se `quantidadeParcelas` omitida)**:
    - Gera lista de parcelamentos possíveis (24 até o máximo permitido), com taxa, parcela, custo total e taxa efetiva mensal.
     
#### 2.5. Validações
- Verificar `idCliente`, `dataInicioPagamento` futura (ex.: `22/02/2025`).
- Validar `valorEmprestimo` positivo; se `quantidadeParcelas` fornecida, deve ser múltiplo de 12 e ≥ 24.
- Ajustar ao máximo permitido, respeitando idade final ≤ 80.
- Verificar margem.
- **Refinanciamento**:
  - Verificar se pelo menos 20% das parcelas do empréstimo original foram pagas.
  - Validar idade final ≤ 80 anos com `novaQuantidadeParcelas`.
- **Portabilidade**:
  - Verificar se as parcelas do empréstimo original estão em dia.
  - Confirmar que o banco de destino suporta a operação de portabilidade.
  - 
### 2.6. Saídas Geradas
- **Com `quantidadeParcelas` fornecida**:
   - Exemplo (sem seguro):
     ```json
     {
       "idCliente": "123.456.789-00",
       "valorEmprestimo": 10000.00,
       "parcela": 319.81,
       "quantidadeParcelas": 48,
       "dataInicioPagamento": "01/04/2025",
       "dataFimContrato": "01/04/2029",
       "taxaJurosMensal": 0.018,
       "taxaEfetivaMensal": 0.0185,  // Novo campo: calculado com IOF
       "contratarSeguro": false,
       "custoSeguro": 0.00,
       "iof": 157.99,
       "carencia": 30,
       "valorTotalFinanciado": 10318.84,
       "margemUtilizada": 319.81,
       "margemRestante": 380.19,
       "prazoMaximoPermitido": 48
     }

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
        "taxaEfetivaMensal": 0.0192,  // Novo campo: inclui IOF e seguro
        "contratarSeguro": true,
        "custoSeguro": 1150.00,
        "iof": 157.99,
        "carencia": 30,
        "valorTotalFinanciado": 11496.87,
        "margemUtilizada": 350.13,
        "margemRestante": 349.87,
        "prazoMaximoPermitido": 48
      }
    ```

- **Cálculo da Amortização (Price)**:
  - Juros = Saldo Devedor Anterior * TaxaJurosMensal
  - Amortização = Parcela - Juros
  - Saldo Devedor = Saldo Devedor Anterior - Amortização

| Parcela | Saldo Devedor Anterior | Juros       | Amortização | Parcela  | Saldo Devedor Restante |
|---------|-----------------------|-------------|-------------|----------|------------------------|
| 1       | 11.496,87             | 189,70      | 160,43      | 350,13   | 11.336,44              |
| 2       | 11.336,44             | 187,05      | 163,08      | 350,13   | 11.173,36              |
| 3       | 11.173,36             | 184,36      | 165,77      | 350,13   | 11.007,59              |
| 4       | 11.007,59             | 181,63      | 168,50      | 350,13   | 10.839,09              |
| 5       | 10.839,09             | 178,85      | 171,28      | 350,13   | 10.667,81              |
| ...     | ...                   | ...         | ...         | ...      | ...                    |
| 48      | 347,45                | 5,73        | 344,40      | 350,13   | 0,00                   |

   
    
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
      
#### 2.7. Refinanciamento de Empréstimo Consignado
O refinanciamento permite renegociar um empréstimo consignado existente no mesmo banco, possibilitando melhores condições de taxa de juros ou prazo.

**Entrada de Dados Adicionais**:
- `idEmprestimoOriginal`: Identificador único do empréstimo a ser refinanciado (ex.: `"EMP123456"`).
- `novoValorEmprestimo`: Valor adicional solicitado (opcional, ex.: `5000.00`).
- `novaQuantidadeParcelas`: Novo prazo desejado (múltiplo de 12, mínimo 24, ex.: `36`).

**Processo**:
1. Consultar o banco de dados com `idEmprestimoOriginal` para obter:
   - Saldo devedor atual.
   - Parcelas já pagas.
   - Taxa de juros original e prazo restante.
2. Verificar elegibilidade:
   - Mínimo de 20% das parcelas originais pagas.
   - Idade final ≤ 80 anos com o novo prazo.
3. Calcular o novo empréstimo:
   - `ValorTotalFinanciado = SaldoDevedor + novoValorEmprestimo (se fornecido) + CustoSeguro (se aplicável) + IOF`.
   - Nova taxa de juros ajustada conforme regras de `tipoVinculo` e `idade` (seção 2.3).
   - Nova parcela via fórmula de Price (seção 3.1).
4. Validar a margem consignável com a nova parcela.


### 2.8. Portabilidade de Empréstimo Consignado (Nova Seção)

#### 2.8. Portabilidade de Empréstimo Consignado
A portabilidade permite transferir um empréstimo consignado para outro banco, buscando melhores condições.

**Entrada de Dados Adicionais**:
- `idEmprestimoOriginal`: Identificador do empréstimo a ser portado (ex.: `"EMP123456"`).
- `bancoDestino`: Identificador do banco receptor (ex.: `"BANCOXYZ"`).
- `novaQuantidadeParcelas`: Novo prazo desejado no banco de destino (múltiplo de 12, ex.: `48`).

**Processo**:
1. Consultar o banco de dados com `idEmprestimoOriginal` para obter:
   - Saldo devedor atual.
   - Status de pagamento (deve estar em dia).
2. Verificar elegibilidade:
   - Parcelas em dia.
   - Banco de destino aceita portabilidade.
3. No banco de destino:
   - Calcular novo empréstimo com base no saldo devedor, nova taxa de juros (definida pelo banco de destino) e `novaQuantidadeParcelas`.
   - Incluir `CustoSeguro` (se aplicável) e `IOF`.
   - Calcular nova parcela via fórmula de Price.
4. Validar a margem consignável com a nova parcela.

**Saída**:
- JSON com detalhes do novo empréstimo no banco de destino:
  ```json
  {
    "idCliente": "123.456.789-00",
    "idEmprestimoOriginal": "EMP123456",
    "bancoDestino": "BANCOXYZ",
    "saldoDevedorOriginal": 8000.00,
    "parcela": 360.20,
    "novaQuantidadeParcelas": 48,
    "taxaJurosMensal": 0.0155,
    "taxaEfetivaMensal": 0.0180,
    "contratarSeguro": false,
    "custoSeguro": 0.00,
    "iof": 120.00,
    "valorTotalFinanciado": 8120.00,
    "margemUtilizada": 360.20,
    "margemRestante": 339.80
  }

**Saída**:
- JSON com detalhes do novo empréstimo:
  ```json
  {
    "idCliente": "123.456.789-00",
    "idEmprestimoOriginal": "EMP123456",
    "saldoDevedorOriginal": 8000.00,
    "novoValorEmprestimo": 2000.00,
    "parcela": 375.50,
    "novaQuantidadeParcelas": 48,
    "taxaJurosMensal": 0.0165,
    "taxaEfetivaMensal": 0.0190,
    "contratarSeguro": true,
    "custoSeguro": 920.00,
    "iof": 125.60,
    "valorTotalFinanciado": 11045.60,
    "margemUtilizada": 375.50,
    "margemRestante": 324.50
  }

3. Estrutura dos Cálculos

## 3.1. Fórmulas

- **Margem**:  
  `Margem = (Remuneração líquida * 0.35) - Parcelas anteriores de empréstimos` (35% destinada a empréstimos consignados, conforme Lei 10.820/2003)

- **Taxa**:  
  `TaxaJurosMensal = TaxaBase + 0.0025 * ((QuantidadeParcelas - 24) / 12)`, com teto de 2,14%.

- **Seguro**:  
  `CustoSeguro = [0,04 + (0,001 * idade)] * ValorEmprestimo`

- **Parcela**:  
  `Parcela = [ValorTotalFinanciado * TaxaJurosMensal] / [1 - (1 + TaxaJurosMensal)^(-QuantidadeParcelas)]`

- **Taxa Efetiva Mensal (CET)** *(novo)*:  
  - Resolve: `ValorEmprestimo = Parcela * [(1 - (1 + TaxaEfetivaMensal)^(-QuantidadeParcelas)) / TaxaEfetivaMensal]`  
  - Considera `ValorTotalFinanciado` (com IOF e seguro) para refletir o custo total ao cliente.

#### 3.2. Exemplo Prático

##### Exemplo de Empréstimo Novo
- **Entrada**:  
  - `"idCliente": "123.456.789-00"`  
  - `"valorEmprestimo": 10000.00"`  
  - `"quantidadeParcelas": OMITIDA`  
  - `"contratarSeguro": true`  
- **Dados Coletados**:  
  - `tipoVinculo`: `"aposentado"`, `idade`: 75, `vencimentos`: 5000.00, `parcelas anteriores`: 800.00  
- **Saída**: Veja exemplo na seção original.

##### Exemplo de Refinanciamento
- **Entrada**:  
  - `"idEmprestimoOriginal": "EMP123456"`  
  - `"novoValorEmprestimo": 2000.00"`  
  - `"novaQuantidadeParcelas": 48"`  
- **Dados do Empréstimo Original**:  
  - Saldo devedor `8000.00`, 12 de 36 parcelas pagas, `tipoVinculo`: `"aposentado"`, `idade`: 75, `margem`: 950.00  
- **Cálculos**:  
  - `ValorTotalFinanciado = 8000.00 + 2000.00 + 920.00 (seguro) + 125.60 (IOF) = 11045.60`  
  - `Parcela = 375.50` (via Price, taxa 0.0165)  
- **Saída**: Veja JSON na seção 2.7.

##### Exemplo de Portabilidade
- **Entrada**:  
  - `"idEmprestimoOriginal": "EMP123456"`  
  - `"bancoDestino": "BANCOXYZ"`  
  - `"novaQuantidadeParcelas": 48"`  
- **Dados do Empréstimo Original**:  
  - Saldo devedor `8000.00`, parcelas em dia, `margem`: 950.00  
- **Cálculos**:  
  - Taxa do banco destino: 0.0155  
  - `ValorTotalFinanciado = 8000.00 + 120.00 (IOF) = 8120.00`  
  - `Parcela = 360.20` (via Price)  
- **Saída**: Veja JSON na seção 2.8.

   Para o exemplo com `ValorTotalFinanciado = 11.496,87`, `TaxaJurosMensal = 0,0165`, e `Parcela = 350,13`:
   
   | Parcela | Saldo Devedor Anterior | Juros  | Amortização | Parcela | Saldo Devedor Restante |
   |---------|-----------------------|--------|-------------|---------|------------------------|
   | 1       | 11.496,87             | 189,70 | 160,43      | 350,13  | 11.336,44              |
   | 2       | 11.336,44             | 187,05 | 163,08      | 350,13  | 11.173,36              |
   | 3       | 11.173,36             | 184,36 | 165,77      | 350,13  | 11.007,59              |
   | 4       | 11.007,59             | 181,63 | 168,50      | 350,13  | 10.839,09              |
   | 5       | 10.839,09             | 178,85 | 171,28      | 350,13  | 10.667,81              |
   | ...     | ...                   | ...    | ...         | ...     | ...                    |
   | 48      | 347,45                | 5,73   | 344,40      | 350,13  | 0,00                   |

## 4. Observações
- Sem `quantidadeParcelas`, retorna opções viáveis até o prazo máximo, respeitando margem e idade.
- Taxas aumentam 0,0025 a cada 12 meses, com teto de 1,80%.
- Seguro é um custo fixo incluído no `ValorTotalFinanciado`; amortização detalha valores exatos.
- A **taxa efetiva mensal (CET)** é exibida para transparência, refletindo o custo total do empréstimo (juros, IOF, seguro), mas não substitui a `taxaJurosMensal` nos cálculos financeiros.

   ### Explicações
      1. Se `quantidadeParcelas` é omitida, o backend gera `opcoesParcelamento` com todas as opções de 24 até o máximo permitido, incluindo taxa ajustada, parcela, custo do seguro (se aplicável) e impacto na margem.
      2.  Todas as validações (idade ≤ 80, margem, múltiplos de 12) são aplicadas às opções retornadas.

  ## 5. Referências Legais
- **Lei 10.820/2003**: Dispõe sobre a autorização para desconto de prestações em folha de pagamento, definindo a base para consignados. [Leia aqui](https://www.jusbrasil.com.br/legislacao/98043/lei-10820-03).
- **Lei 14.509/2022**: Aumenta a margem consignável para servidores federais, detalhando uso de 35% para empréstimos. [Leia aqui](https://www.gov.br/servidor/pt-br/assuntos/noticias/2023/maio/entenda-servidores-que-estao-com-margem-consignada-totalmente-comprometida-nao-serao-prejudicados-por-mudanca-na-lei-no-14-509-2022).
- **Regulamentação INSS**: Estabelece 35% da margem para empréstimos consignados de aposentados. [Leia aqui](https://www.gov.br/inss/pt-br/noticias/margem-do-emprestimo-consignado-esta-atualizada).

---
---
---

> **📄 Documentação 2**  
# Documentação - Emprest.AI (Consulta e Atualização de Dados)

## 1. Objetivo
Esta parte do backend gerencia a **consulta** e **atualização** de informações relacionadas às parcelas de empréstimos consignados no banco de dados. Inclui o acompanhamento das datas de vencimento, registro de pagamentos, cálculo de multas e juros de mora por atraso, além de oferecer relatórios detalhados para clientes e administradores.

## 2. Funcionalidades

### 2.1. Consulta de Dados
Permite consultar o estado atual de um empréstimo consignado, incluindo parcelas pagas, vencidas e a vencer, com detalhes de multas e juros aplicáveis, além de informações sobre refinanciamento ou portabilidade, se aplicável.

#### 2.1.1. Entrada de Dados
Parâmetros recebidos:
- **idCliente**: Ex.: `"123.456.789-00"` (CPF do cliente).
- **idEmprestimo**: Opcional, Ex.: `"EMP-00123"` (identificador único do empréstimo). Se omitido, retorna todos os empréstimos do cliente.
- **dataConsulta**: Opcional, Ex.: `"22/02/2025"`. Padrão: data atual (ex.: `22/02/2025`).

#### 2.1.2. Processo de Consulta
1. **Validação**:
   - Verificar se `idCliente` existe no banco de dados.
   - Se `idEmprestimo` for fornecido, validar sua existência associada ao `idCliente`.

2. **Busca no banco de dados**:
   - Recuperar:
     - Dados gerais do empréstimo: `valorEmprestimo`, `quantidadeParcelas`, `taxaJurosMensal`, `dataInicioPagamento`, `contratarSeguro`, etc.
     - Tabela de parcelas: `numeroParcela`, `dataVencimento`, `dataPagamento`, `valorParcelaOriginal`, `multaAtraso`, `jurosMora`, `valorPago`, `status` (`paga`, `vencida`, `a vencer`).
     - *Para refinanciamento/portabilidade*: `statusContrato` (`ativo`, `refinanciado`, `portado`), `idEmprestimoNovo` (se aplicável), `saldoDevedorOriginal`, `bancoDestino` (para portabilidade).

3. **Cálculo de atrasos**:
   - Para parcelas com `status = vencida`:
     - **Multa por atraso**: Fixa em 2% sobre `valorParcelaOriginal`.
     - **Juros de mora**: 1% ao mês (0,0333% ao dia) sobre `valorParcelaOriginal`, proporcional aos dias de atraso (`dataConsulta - dataVencimento`).
     - Fórmula:  
       `Multa = valorParcelaOriginal * 0.02`  
       `JurosMora = valorParcelaOriginal * 0.000333 * diasAtraso`  
       `ValorTotalDevido = valorParcelaOriginal + Multa + JurosMora`

#### 2.1.3. Saídas Geradas
- **Exemplo (empréstimo ativo)**:
    ```json
    {
      "idCliente": "123.456.789-00",
      "idEmprestimo": "EMP-00123",
      "valorEmprestimo": 10000.00,
      "quantidadeParcelas": 48,
      "taxaJurosMensal": 0.0165,
      "dataInicioPagamento": "01/04/2025",
      "dataConsulta": "22/02/2025",
      "statusContrato": "ativo",
      "parcelas": [
        {
          "numeroParcela": 1,
          "dataVencimento": "01/05/2025",
          "dataPagamento": "30/04/2025",
          "valorParcelaOriginal": 350.13,
          "multaAtraso": 0.00,
          "jurosMora": 0.00,
          "valorPago": 350.13,
          "status": "paga"
        },
        {
          "numeroParcela": 2,
          "dataVencimento": "01/06/2025",
          "dataPagamento": null,
          "valorParcelaOriginal": 350.13,
          "multaAtraso": 0.00,
          "jurosMora": 0.00,
          "valorPago": 0.00,
          "status": "a vencer"
        }
      ],
      "totalPago": 350.13,
      "totalDevido": 0.00,
      "proximaParcela": {
        "numeroParcela": 2,
        "dataVencimento": "01/06/2025",
        "valorParcelaOriginal": 350.13
      }
    }
  ```
- **Exemplo (empréstimo refinanciado**:
    ```json
   {
     "idCliente": "123.456.789-00",
     "idEmprestimo": "EMP-00123",
     "valorEmprestimo": 10000.00,
     "quantidadeParcelas": 48,
     "taxaJurosMensal": 0.0165,
     "dataInicioPagamento": "01/04/2025",
     "dataConsulta": "22/02/2025",
     "statusContrato": "refinanciado",
     "idEmprestimoNovo": "EMP-00124",
     "saldoDevedorOriginal": 8000.00,
     "parcelas": [
       {
         "numeroParcela": 1,
         "dataVencimento": "01/05/2025",
         "dataPagamento": "30/04/2025",
         "valorParcelaOriginal": 350.13,
         "multaAtraso": 0.00,
         "jurosMora": 0.00,
         "valorPago": 350.13,
         "status": "paga"
       }
     ],
     "totalPago": 350.13,
     "totalDevido": 0.00,
     "mensagem": "Empréstimo refinanciado. Consultar novo contrato EMP-00124 para detalhes."
   }
    ```

### 2.2. Atualização de Dados
Permite registrar pagamentos, atualizar datas, recalcular valores de multas e juros de mora, e gerenciar o encerramento de contratos devido a refinanciamento ou portabilidade.

#### 2.2.1. Entrada de Dados
Parâmetros recebidos:  
- **idCliente**: Ex.: `"123.456.789-00"`.  
- **idEmprestimo**: Ex.: `"EMP-00123"`.  
- **numeroParcela**: Ex.: `3` (parcela específica a ser atualizada).  
- **dataPagamento**: Ex.: `"15/07/2025"` (data em que o pagamento foi realizado).  
- **valorPago**: Ex.: `358.76` (valor efetivamente pago).  
- *Opcional (para refinanciamento/portabilidade)*:  
  - **acao**: `"refinanciar"` ou `"portar"`.  
  - **idEmprestimoNovo**: Ex.: `"EMP-00124"` (novo contrato gerado).  
  - **bancoDestino**: Ex.: `"BANCOXYZ"` (obrigatório para portabilidade).

#### 2.2.2. Processo de Atualização
1. **Validação**:  
   - Verificar se `idCliente`, `idEmprestimo` e `numeroParcela` existem e estão associados.  
   - Confirmar que a parcela não está com `status = paga` (exceto para ações de refinanciamento/portabilidade).  
   - Validar que `dataPagamento` é igual ou posterior a `dataVencimento`.  
   - *Para refinanciamento/portabilidade*: Confirmar que `idEmprestimoNovo` é válido e que o saldo devedor foi quitado ou transferido.

2. **Cálculo de atraso (se aplicável)**:  
   - Se `dataPagamento > dataVencimento`:  
     - **Multa**: `valorParcelaOriginal * 0.02`.  
     - **Juros de mora**: `valorParcelaOriginal * 0.000333 * (dataPagamento - dataVencimento)`.  
     - **Valor total devido**: `valorParcelaOriginal + Multa + JurosMora`.  

3. **Atualização no banco**:  
   - **Pagamento normal**:  
     - Atualizar:  
       - `dataPagamento`: Data informada.  
       - `multaAtraso`: Valor calculado (0,00 se pago em dia).  
       - `jurosMora`: Valor calculado (0,00 se pago em dia).  
       - `valorPago`: Valor informado pelo usuário.  
       - `status`: Alterar para `"paga"`.  
   - **Refinanciamento**:  
     - Atualizar `statusContrato` para `"refinanciado"`.  
     - Registrar `idEmprestimoNovo`.  
     - Encerrar o contrato original (saldo devedor quitado pelo novo contrato).  
   - **Portabilidade**:  
     - Atualizar `statusContrato` para `"portado"`.  
     - Registrar `bancoDestino`.  
     - Encerrar o contrato original (saldo devedor transferido ao banco de destino).

4. **Verificação de pagamento insuficiente**:  
   - Se `valorPago < valorTotalDevido`, registrar o pagamento parcial e manter `status = vencida`, ajustando o `valorParcelaOriginal` restante.

#### 2.2.3. Saídas Geradas
- **Exemplo (pagamento em atraso)**:  
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
      "valorPago": 358.76,
      "status": "paga",
      "mensagem": "Parcela 3 atualizada com sucesso. Pagamento registrado com multa e juros por 14 dias de atraso."
    }
  ```
  
  - **Exemplo (refinanciamento)**:  
  ```json
  {
    "idCliente": "123.456.789-00",
    "idEmprestimo": "EMP-00123",
    "acao": "refinanciar",
    "idEmprestimoNovo": "EMP-00124",
    "saldoDevedorOriginal": 8000.00,
    "statusContrato": "refinanciado",
    "mensagem": "Contrato EMP-00123 refinanciado com sucesso. Novo contrato: EMP-00124."
  }
  ```
 - **Exemplo (portabilidade)**:  
  ```json
  {
    "idCliente": "123.456.789-00",
    "idEmprestimo": "EMP-00123",
    "acao": "portar",
    "bancoDestino": "BANCOXYZ",
    "saldoDevedorOriginal": 8000.00,
    "statusContrato": "portado",
    "mensagem": "Contrato EMP-00123 portado para BANCOXYZ com sucesso."
  }
  ```

### 3. Estrutura dos Cálculos

#### 3.1. Fórmulas
- **Multa por atraso**:  
  `Multa = valorParcelaOriginal * 0.02`  

- **Juros de mora**:  
  `JurosMora = valorParcelaOriginal * 0.000333 * diasAtraso`  

- **Valor total devido**:  
  `ValorTotalDevido = valorParcelaOriginal + Multa + JurosMora`  

- **Dias de atraso**:  
  `diasAtraso = dataConsulta (ou dataPagamento) - dataVencimento`

#### 3.2. Exemplo Prático

##### Exemplo de Pagamento Normal
- **Entrada**:  
  - `"idCliente": "123.456.789-00"`  
  - `"idEmprestimo": "EMP-00123"`  
  - `"numeroParcela": 3`  
  - `"dataPagamento": "15/07/2025"`  
  - `"valorPago": 358.76"`  
- **Dados do banco**:  
  - `dataVencimento`: `"01/07/2025"`  
  - `valorParcelaOriginal`: `350.13`  
  - `status`: `"vencida"`  
- **Cálculos**:  
  - `Dias de atraso`: `15/07/2025 - 01/07/2025 = 14 dias`  
  - `Multa`: `350.13 * 0.02 = 7.00`  
  - `Juros de mora`: `350.13 * 0.000333 * 14 = 1.63`  
  - `Valor total devido`: `350.13 + 7.00 + 1.63 = 358.76`  
- **Resultado**: Veja saída na seção 2.2.3 (pagamento em atraso).

##### Exemplo de Refinanciamento
- **Entrada**:  
  - `"idCliente": "123.456.789-00"`  
  - `"idEmprestimo": "EMP-00123"`  
  - `"acao": "refinanciar"`  
  - `"idEmprestimoNovo": "EMP-00124"`  
- **Dados do banco**:  
  - `saldoDevedorOriginal`: `8000.00`  
  - 12 de 48 parcelas pagas  
- **Resultado**:  
  - `statusContrato` atualizado para `"refinanciado"`.  
  - Registro de `idEmprestimoNovo`.  
  - Veja saída na seção 2.2.3 (refinanciamento).

##### Exemplo de Portabilidade
- **Entrada**:  
  - `"idCliente": "123.456.789-00"`  
  - `"idEmprestimo": "EMP-00123"`  
  - `"acao": "portar"`  
  - `"bancoDestino": "BANCOXYZ"`  
- **Dados do banco**:  
  - `saldoDevedorOriginal`: `8000.00`  
  - Parcelas em dia  
- **Resultado**:  
  - `statusContrato` atualizado para `"portado"`.  
  - Registro de `bancoDestino`.  
  - Veja saída na seção 2.2.3 (portabilidade).

### 4. Observações
- **Multa e juros**: Aplicados apenas a parcelas vencidas, com valores fixos (multa 2%) e variáveis (juros 1% ao mês).  
- **Pagamentos parciais**: Permitem quitar parte da dívida, ajustando o saldo devedor restante.  
- **Histórico**: Todas as atualizações mantêm um log no banco para auditoria.  
- **Data padrão**: Se `dataConsulta` não for fornecida, usa a data atual (`22/02/2025`).  
- **Refinanciamento e Portabilidade**: Contratos refinanciados ou portados são marcados como encerrados (`statusContrato = refinanciado` ou `portado`), e o cliente deve consultar o novo contrato ou banco de destino para acompanhamento futuro.

- **Banco Central do Brasil**: Define teto de 2,14% ao mês para taxas de juros de consignados. [Leia mais](https://pt.wikipedia.org/wiki/Cr%C3%A9dito_consignado).
      


