# Documentação - Emprest.AI (Consulta e Atualização de Dados)

## Autores
- [@dalleth](https://github.com/dalleth-martinss)
- [@josdcosta](https://github.com/josdcosta)
- [@CarollinaGuedes](https://github.com/CarollinaGuedes)

## Referência

---

## 1. Objetivo
Esta parte do backend gerencia a **consulta** e **atualização** de informações relacionadas às parcelas de empréstimos consignados no banco de dados. Inclui o acompanhamento das datas de vencimento, registro de pagamentos, cálculo de multas e juros de mora por atraso, além de oferecer relatórios detalhados para clientes e administradores.

---

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
