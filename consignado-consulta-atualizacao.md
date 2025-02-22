# Documentação - Emprest.AI (Consulta e Atualização de Dados)

## Autores
- [@dalleth](https://github.com/dalleth-martinss)
- [@josdcosta](https://github.com/josdcosta)
- [@CarollinaGuedes](https://github.com/CarollinaGuedes)

## Referência
- [Documentação de Concessão](#) (base para esta extensão)

---

## 1. Objetivo
Esta parte do backend gerencia a **consulta** e **atualização** de informações relacionadas às parcelas de empréstimos consignados no banco de dados. Inclui o acompanhamento das datas de vencimento, registro de pagamentos, cálculo de multas e juros de mora por atraso, além de oferecer relatórios detalhados para clientes e administradores.

---

## 2. Funcionalidades

### 2.1. Consulta de Dados
Permite consultar o estado atual de um empréstimo consignado, incluindo parcelas pagas, vencidas e a vencer, com detalhes de multas e juros aplicáveis.

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

3. **Cálculo de atrasos**:
   - Para parcelas com `status = vencida`:
     - **Multa por atraso**: Fixa em 2% sobre `valorParcelaOriginal`.
     - **Juros de mora**: 1% ao mês (0,0333% ao dia) sobre `valorParcelaOriginal`, proporcional aos dias de atraso (`dataConsulta - dataVencimento`).
     - Fórmula:  
       `Multa = valorParcelaOriginal * 0.02`  
       `JurosMora = valorParcelaOriginal * 0.000333 * diasAtraso`  
       `ValorTotalDevido = valorParcelaOriginal + Multa + JurosMora`

#### 2.1.3. Saídas Geradas
- **Exemplo (empréstimo com parcelas em dia e atrasadas)**:
    ```json
    {
      "idCliente": "123.456.789-00",
      "idEmprestimo": "EMP-00123",
      "valorEmprestimo": 10000.00,
      "quantidadeParcelas": 48,
      "taxaJurosMensal": 0.0165,
      "dataInicioPagamento": "01/04/2025",
      "dataConsulta": "22/02/2025",
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
        },
        {
          "numeroParcela": 3,
          "dataVencimento": "01/07/2025",
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

  Exemplo (parcela vencida):
    ```json
        {
        "idCliente": "123.456.789-00",
        "idEmprestimo": "EMP-00123",
        "dataConsulta": "15/07/2025",
        "parcelas": [
          {
            "numeroParcela": 3,
            "dataVencimento": "01/07/2025",
            "dataPagamento": null,
            "valorParcelaOriginal": 350.13,
            "multaAtraso": 7.00,
            "jurosMora": 1.63,
            "valorPago": 0.00,
            "valorTotalDevido": 358.76,
            "status": "vencida"
          }
        ],
        "totalDevido": 358.76
      }
    ```
### 2.2. Atualização de Dados
Permite registrar pagamentos, atualizar datas e recalcular valores de multas e juros de mora.

#### 2.2.1. Entrada de Dados
Parâmetros recebidos:  
- **idCliente**: Ex.: `"123.456.789-00"`.  
- **idEmprestimo**: Ex.: `"EMP-00123"`.  
- **numeroParcela**: Ex.: `3` (parcela específica a ser atualizada).  
- **dataPagamento**: Ex.: `"15/07/2025"` (data em que o pagamento foi realizado).  
- **valorPago**: Ex.: `358.76` (valor efetivamente pago).

#### 2.2.2. Processo de Atualização
1. **Validação**:  
   - Verificar se `idCliente`, `idEmprestimo` e `numeroParcela` existem e estão associados.  
   - Confirmar que a parcela não está com `status = paga`.  
   - Validar que `dataPagamento` é igual ou posterior a `dataVencimento`.  

2. **Cálculo de atraso (se aplicável)**:  
   - Se `dataPagamento > dataVencimento`:  
     - **Multa**: `valorParcelaOriginal * 0.02`.  
     - **Juros de mora**: `valorParcelaOriginal * 0.000333 * (dataPagamento - dataVencimento)`.  
     - **Valor total devido**: `valorParcelaOriginal + Multa + JurosMora`.  

3. **Atualização no banco**:  
   - Atualizar:  
     - `dataPagamento`: Data informada.  
     - `multaAtraso`: Valor calculado (0,00 se pago em dia).  
     - `jurosMora`: Valor calculado (0,00 se pago em dia).  
     - `valorPago`: Valor informado pelo usuário.  
     - `status`: Alterar para `"paga"`.  

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
Exemplo (pagamento parcial):
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
    "valorPago": 300.00,
    "valorRestante": 58.76,
    "status": "vencida",
    "mensagem": "Pagamento parcial registrado. Resta 58.76 para quitar a parcela 3."
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
##### Entrada:  
- **idCliente**: `"123.456.789-00"`  
- **idEmprestimo**: `"EMP-00123"`  
- **numeroParcela**: `3`  
- **dataPagamento**: `"15/07/2025"`  
- **valorPago**: `358.76`  

##### Dados do banco:  
- `dataVencimento`: `"01/07/2025"`  
- `valorParcelaOriginal`: `350.13`  
- `status`: `"vencida"`

##### Cálculos:  
- **Dias de atraso**:  
  `15/07/2025 - 01/07/2025 = 14 dias`  
- **Multa**:  
  `350.13 * 0.02 = 7.00`  
- **Juros de mora**:  
  `350.13 * 0.000333 * 14 = 1.63`  
- **Valor total devido**:  
  `350.13 + 7.00 + 1.63 = 358.76`  

##### Resultado:  
- Registro atualizado com `status = paga`, `dataPagamento = 15/07/2025`, `valorPago = 358.76`.

### 4. Observações
- **Multa e juros**: Aplicados apenas a parcelas vencidas, com valores fixos (multa 2%) e variáveis (juros 1% ao mês).  
- **Pagamentos parciais**: Permitem quitar parte da dívida, ajustando o saldo devedor restante.  
- **Histórico**: Todas as atualizações mantêm um log no banco para auditoria.  
- **Data padrão**: Se `dataConsulta` não for fornecida, usa a data atual (`22/02/2025`).
