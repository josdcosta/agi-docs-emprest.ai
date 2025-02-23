# Documentação - Empréstimo Pessoal

## 1. Objetivo

Este sistema de backend foi desenvolvido para gerenciar consultas e atualizações de informações relacionadas a empréstimos pessoais em um banco de dados. Ele abrange o acompanhamento de datas de vencimento, registro de pagamentos, cálculo de multas e juros de mora por atraso, além de fornecer relatórios detalhados para clientes e administradores.

## 2. Funcionalidades

### 2.1. Consulta de Dados

Permite consultar o estado atual de um empréstimo pessoal, incluindo parcelas pagas, vencidas e a vencer, com detalhes de multas e juros aplicáveis.

#### 2.1.1. Entrada de Dados

* `idCliente`: Ex.: "123.456.789-00" (CPF do cliente).
* `idEmprestimo`: Opcional, Ex.: "EMP-00123" (identificador único do empréstimo). Se omitido, retorna todos os empréstimos do cliente.
* `dataConsulta`: Opcional, Ex.: "22/02/2025". Padrão: data atual (ex.: 22/02/2025).

#### 2.1.2. Processo de Consulta

* **Validação:**
    * Verificar se `idCliente` existe no banco de dados.
    * Se `idEmprestimo` for fornecido, validar sua existência associada ao `idCliente`.
* **Busca no banco de dados:**
    * Recuperar:
        * Dados gerais do empréstimo: `valorEmprestimo`, `quantidadeParcelas`, `taxaJurosMensal`, `dataInicioPagamento`, etc.
        * Tabela de parcelas: `numeroParcela`, `dataVencimento`, `dataPagamento`, `valorParcelaOriginal`, `multaAtraso`, `jurosMora`, `valorPago`, `status` (paga, vencida, a vencer).
* **Cálculo de atrasos:**
    * Para parcelas com `status = vencida`:
        * Multa por atraso: Fixa em 2% sobre `valorParcelaOriginal`.
        * Juros de mora: 1% ao mês (0,0333% ao dia) sobre `valorParcelaOriginal`, proporcional aos dias de atraso (`dataConsulta` - `dataVencimento`).
        * Fórmula:
            * `Multa = valorParcelaOriginal * 0.02`
            * `JurosMora = valorParcelaOriginal * 0.000333 * diasAtraso`
            * `ValorTotalDevido = valorParcelaOriginal + Multa + JurosMora`
        * Exemplo de cálculo:
            * Valor da parcela original: R$ 1.500,00
            * Dias de atraso: 20
            * Multa: R$ 1.500,00 \* 0.02 = R$ 30,00
            * Juros de mora: R$ 1.500,00 \* 0.000333 \* 20 = R$ 9,99
            * Valor total devido: R$ 1.500,00 + R$ 30,00 + R$ 9,99 = R$ 1.539,99

#### 2.1.3. Saídas Geradas

* Exemplo (empréstimo com parcelas em dia e atrasadas):

    ```json
    {
        "idCliente": "987.654.321-00",
        "idEmprestimo": "EMP-00789",
        "valorEmprestimo": 25000.00,
        "quantidadeParcelas": 24,
        "taxaJurosMensal": 0.025,
        "dataInicioPagamento": "10/02/2025",
        "dataConsulta": "05/05/2025",
        "parcelas": [
            {
                "numeroParcela": 1,
                "dataVencimento": "10/03/2025",
                "dataPagamento": "08/03/2025",
                "valorParcelaOriginal": 1326.00,
                "multaAtraso": 0.00,
                "jurosMora": 0.00,
                "valorPago": 1326.00,
                "status": "paga"
            },
            {
                "numeroParcela": 2,
                "dataVencimento": "10/04/2025",
                "dataPagamento": null,
                "valorParcelaOriginal": 1326.00,
                "multaAtraso": 0.00,
                "jurosMora": 0.00,
                "valorPago": 0.00,
                "status": "a vencer"
            },
            {
                "numeroParcela": 3,
                "dataVencimento": "10/05/2025",
                "dataPagamento": null,
                "valorParcelaOriginal": 1326.00,
                "multaAtraso": 0.00,
                "jurosMora": 0.00,
                "valorPago": 0.00,
                "status": "a vencer"
            }
        ],
        "totalPago": 1326.00,
        "totalDevido": 0.00,
        "proximaParcela": {
            "numeroParcela": 2,
            "dataVencimento": "10/04/2025",
            "valorParcelaOriginal": 1326.00
        }
    }
    ```

* Exemplo (parcela vencida):

    ```json
    {
        "idCliente": "987.654.321-00",
        "idEmprestimo": "EMP-00789",
        "dataConsulta": "30/05/2025",
        "parcelas": [
            {
                "numeroParcela": 3,
                "dataVencimento": "10/05/2025",
                "dataPagamento": null,
                "valorParcelaOriginal": 1326.00,
                "multaAtraso": 26.52,
                "jurosMora": 8.83,
                "valorPago": 0.00,
                "valorTotalDevido": 1361.35,
                "status": "vencida"
            }
        ],
        "totalDevido": 1361.35
    }
    ```

### 2.2. Atualização de Dados

Permite registrar pagamentos de parcelas, calculando multas e juros de mora quando aplicável.

#### 2.2.1. Entrada de Dados

* `idCliente`: Ex.: "987.654.321-00" (CPF do cliente).
* `idEmprestimo`: Ex.: "EMP-00789" (identificador único do empréstimo).
* `numeroParcela`: Número da parcela paga.
* `dataPagamento`: Data do pagamento, Ex.: "30/05/2025".
* `valorPago`: Valor pago pelo cliente.

#### 2.2.2. Processo de Atualização

* **Validação:**
    * Verificar se `idCliente`, `idEmprestimo` e `numeroParcela` existem e estão associados.
    * Confirmar que a parcela não está com `status = paga`.
    * Validar que `dataPagamento` é igual ou posterior a `dataVencimento`.
* **Cálculo de atraso (se aplicável):**
    * Se `dataPagamento > dataVencimento`:
        * Multa: `valorParcelaOriginal * 0.02`.
        * Juros de mora: `valorParcelaOriginal * 0.000333 * (dataPagamento - dataVencimento)`.
        * Valor total devido: `valorParcelaOriginal + Multa + JurosMora`.
* **Atualização no banco:**
    * Atualizar:
        * `dataPagamento`: Data informada.
        * `multaAtraso`: Valor calculado (0,00 se pago em dia).
        * `jurosMora`: Valor calculado (0,00 se pago em dia).
        * `valorPago`: Valor informado pelo usuário.
        * `status`: Alterar para "paga".
* **Verificação de pagamento insuficiente:**
    * Se `valorPago < valorTotalDevido`, registrar o pagamento parcial e manter `status = vencida`, ajustando o `valorParcelaOriginal` restante.

#### 2.2.3. Saídas Geradas

* Exemplo (pagamento em atraso):

    ```json
    {
        "idCliente": "987.654.321-00",
        "idEmprestimo": "EMP-00789",
        "numeroParcela": 3,
        "dataVencimento": "10/05/2025",
        "dataPagamento": "30/05/2025",
        "valorParcelaOriginal": 1326.00,
        "multaAtraso": 26.52,
        "jurosMora": 8.83,
        "valorTotalDevido": 1361.35,
        "valorPago": 1361.35,
        "status": "paga",
        "mensagem": "Parcela 3 atualizada com sucesso. Pagamento registrado com multa e juros por 20 dias de atraso."
    }
    ```

* Exemplo (pagamento parcial):

    ```json
    {
        "idCliente": "987.654.321-00",
        "idEmprestimo": "EMP-00789",
        "numeroParcela": 3,
        "dataVencimento": "10/05/2025",
        "dataPagamento": "20/05/2025",
        "valorParcelaOriginal": 1361.35,
        "multaAtraso": 26.52,
        "jurosMora": 4.41,
        "valorTotalDevido": 1332.28,
        "valorPago": 1000.00,
        "status": "vencida",
        "mensagem": "Pagamento parcial registrado. Valor restante da parcela: 361.35."
    }
    ```

## 3. Estrutura dos Cálculos

### 3.1. Fórmulas

* **Multa por atraso:**
    * `Multa = valorParcelaOriginal * 0.02`
* **Juros de mora:**
    * `JurosMora = valorParcelaOriginal * 0.000333 * diasAtraso`
* **Valor total devido:**
    * `ValorTotalDevido = valorParcelaOriginal + Multa + JurosMora`
* **Dias de atraso:**
    * `diasAtraso = dataConsulta (ou dataPagamento) - dataVencimento`

### 3.2. Exemplo Prático

**Entrada:**

* `idCliente`: "987.654.321-00"
* `idEmprestimo`: "EMP-00789"
* `numeroParcela`: 3
* `dataPagamento`: "30/05/2025"
* `valorPago`: 1361.35

**Dados do banco:**

* `dataVencimento`: "10/05/2025"
* `valorParcelaOriginal`: 1326.00
* `status`: "vencida"

**Cálculos:**

* Dias de atraso: 30/05/2025 - 10/05/2025 = 20 dias
* Multa: 1326.00 * 0.02 = 26.52
* Juros de mora: 1326.00 * 0.000333 * 20 = 8.83
* Valor total devido: 1326.00 + 26.52 + 8.83 = 1361.35

**Resultado:**

* Registro atualizado com `status = paga`, `dataPagamento = 30/05/2025`, `valorPago = 1361.35`.

## 4. Relatórios

* **Relatório de Empréstimos Concedidos:**
    * Campos: `idCliente`, nome do cliente, `valorEmprestimo`, taxa de juros, prazo, data de concessão, `status` do empréstimo.
    * Filtros: período, tipo de empréstimo, `status` do empréstimo.
    * Ordenação: data de concessão, `valorEmprestimo`.
* **Relatório de Taxas de Inadimplência:**
    * Campos: período, total de empréstimos, total de inadimplência, taxa de inadimplência.
    * Filtros: período, tipo de empréstimo.
    * Ordenação: período.

## 5. Requisitos Não Funcionais

* **Disponibilidade:** 99,9%
* **Atualização dos dados:** em tempo real.
* **Feedback ao cliente:** notificações automáticas sobre pagamentos, vencimentos e atualizações no `status` do empréstimo.
* **Segurança:** criptografia de dados sensíveis, autenticação de usuários, autorização de acesso.

## 6. Regras de Negócio Adicionais

### 6.1. Cálculo da Capacidade de Pagamento

* **Fórmula:**
    Capacidade de Pagamento = Renda Líquida \* Percentual Máximo de Comprometimento da Renda
* **Detalhes:**
    * O percentual máximo de comprometimento varia de acordo com o score de crédito do cliente.
    * Outras dívidas e compromissos financeiros são considerados no cálculo.
* **Exemplo:**
    * Renda líquida mensal: R$ 3.000,00
    * Percentual máximo de comprometimento (score médio): 30%
    * Cálculo:
        * Capacidade de pagamento: R$ 3.000,00 \* 0.30 = R$ 900,00
    * Resultado: A capacidade de pagamento do cliente é de R$ 900,00 por mês.

### 6.2. Taxas de Juros e Prazos

* **Taxas de Juros:**
    * Variam de acordo com o score de crédito do cliente, o valor do empréstimo e o prazo.
    * Faixas de taxas são definidas para cada perfil de risco.
* **Prazos:**
    * Prazos mínimos e máximos são definidos com base na capacidade de pagamento do cliente.
    * Prazos diferenciados podem ser aplicados por faixa etária.
* **Exemplo:**
    * Score de crédito: 750 (faixa de score boa)
    * Valor do empréstimo: R$ 10.000,00
    * Prazo desejado: 24 meses
    * Taxa de juros para score 750 e 24 meses: 1,5% ao mês
    * Cálculo da parcela (Sistema Price):
        * Parcela = \[10000 \* 0.015] / \[1 - (1 + 0.015)^(-24)] = R$ 499,24
    * Resultado: A parcela mensal será de R$ 499,24.

### 6.3. Cálculo de Multas e Juros

* **Multa por atraso:** 2% sobre o valor da parcela original.
* **Juros de mora:** 1% ao mês (0,0333% ao dia) sobre o valor da parcela original, proporcional aos dias de atraso.
* **Isenção:** Casos específicos podem ter isenção de multas e juros, mediante análise.
* **Exemplo:**
    * Valor da parcela original: R$ 500,00
    * Dias de atraso: 15 dias
    * Cálculos:
        * Multa: R$ 500,00 \* 0.02 = R$ 10,00
        * Juros de mora: R$ 500,00 \* 0.000333 \* 15 = R$ 2,50
        * Valor total devido: R$ 500,00 + R$ 10,00 + R$ 2,50 = R$ 512,50
    * Resultado: O valor total devido é de R$ 512,50.

### 6.4. Pagamentos e Antecipações

* **Pagamentos Parciais:**
    * O saldo devedor restante é recalculado após cada pagamento parcial.
    * A alocação do pagamento é feita primeiro para juros, depois multa e, por fim, valor principal.
* **Antecipação de Parcelas:**
    * Descontos são aplicados em caso de antecipação de parcelas, calculados com base na taxa de juros do empréstimo.
    * A antecipação pode ser total ou parcial.
* **Exemplo (Pagamento Parcial):**
    * Valor da parcela original: R$ 600,00
    * Valor pago: R$ 400,00
    * Saldo devedor restante: R$ 600,00 - R$ 400,00 = R$ 200,00
    * Resultado: O saldo devedor restante é de R$ 200,00.
* **Exemplo (Antecipação de Parcelas):**
    * Valor da parcela: R$ 500,00
    * Taxa de juros mensal: 1,5%
    * Parcelas a antecipar: 3
    * Cálculo do desconto:
        * Desconto = 500 / (1 + 0.015)^3 = R$ 477.94
    * Resultado: O valor a ser pago com o desconto é de R$ 477.94.

### 6.5. Regras de Negócio por Idade

* **Idade Mínima:**
    * Clientes com menos de 18 anos não são elegíveis para empréstimos.
* **Idade Máxima:**
    * Clientes com mais de 70 anos podem solicitar empréstimos com prazos máximos de 24 meses e taxas de juros 0,5% superiores às taxas normais.
    * Clientes com mais de 75 anos não são elegíveis para novos empréstimos.
* **Prazos e Condições de Pagamento:**
    * Prazos de pagamento para clientes com mais de 65 anos podem ser reduzidos, dependendo da análise de risco.
    * Condições especiais de pagamento podem ser oferecidas para clientes aposentados ou pensionistas, mediante comprovação de renda.
* **Avaliação de Risco:**
    * A avaliação de risco para clientes com mais de 65 anos incluirá análise de histórico de saúde e fontes de renda estáveis (aposentadoria, pensão).
    * Garantias adicionais podem ser solicitadas para clientes com mais de 70 anos.


## 7. Observações

* **Multa e juros:** Aplicados apenas a parcelas vencidas, com valores fixos (multa 2%) e variáveis (juros 1% ao mês).
* **Pagamentos parciais:** Permitem quitar parte da dívida, ajustando o saldo devedor restante.
* **Histórico:** Todas as atualizações mantêm um log no banco para auditoria.
* **Data padrão:** Se `dataConsulta` não for fornecida, usa a data atual (22/02/2025).
