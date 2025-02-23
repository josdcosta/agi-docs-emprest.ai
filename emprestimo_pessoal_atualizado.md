# Documentação - Empréstimo Pessoal

[Planilha de Cálculos](https://docs.google.com/spreadsheets/d/1Y_vrP424Qpyh_nWdp_xtSSbsdswpp4XKPIOVeIV9B4E/edit?gid=1897659843#gid=1897659843)

## 1. Objetivo

Este sistema tem como objetivo principal gerenciar o processo completo de empréstimo pessoal, desde a análise de crédito e aprovação até a gestão das parcelas e o acompanhamento do histórico do cliente. O software visa otimizar a análise de risco, definir limites de crédito personalizados, calcular taxas de juros adequadas e oferecer funcionalidades como simulação de parcelas, antecipação de pagamentos e renovação de empréstimos, tudo isso com o objetivo de proporcionar uma experiência eficiente e transparente para clientes e administradores.

## 2. Funcionalidades

### 2.1. Consulta de Dados

Permite consultar o estado atual de um empréstimo pessoal, incluindo parcelas pagas, vencidas e a vencer, com detalhes de multas e juros aplicáveis.

### 2.1.1. Entrada de Dados

Parâmetros recebidos:

* `idCliente`: Ex.: "123.456.789-00" (CPF do cliente).
* `idEmprestimo`: Opcional, Ex.: "EMP-00123" (identificador único do empréstimo). Se omitido, retorna todos os empréstimos do cliente.
* `dataConsulta`: Opcional, Ex.: "22/02/2025". Padrão: data atual (ex.: 22/02/2025).

---

### 2.1.2. Processo de Consulta

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

### 2.1.3. Saídas Geradas

Exemplo (empréstimo com parcelas em dia e atrasadas):

````json
{
    "idCliente": "987.654.321-09",
    "idEmprestimo": "EMP-00456",
    "valorEmprestimo": 15000.00,
    "quantidadeParcelas": 36,
    "taxaJurosMensal": 0.0180,
    "dataInicioPagamento": "15/03/2025",
    "dataConsulta": "28/02/2025",
    "parcelas": [
        {
            "numeroParcela": 1,
            "dataVencimento": "15/04/2025",
            "dataPagamento": "10/04/2025",
            "valorParcelaOriginal": 525.50,
            "multaAtraso": 0.00,
            "jurosMora": 0.00,
            "valorPago": 525.50,
            "status": "paga"
        },
        {
            "numeroParcela": 2,
            "dataVencimento": "15/05/2025",
            "dataPagamento": null,
            "valorParcelaOriginal": 525.50,
            "multaAtraso": 0.00,
            "jurosMora": 0.00,
            "valorPago": 0.00,
            "status": "a vencer"
        },
        {
            "numeroParcela": 3,
            "dataVencimento": "15/06/2025",
            "dataPagamento": null,
            "valorParcelaOriginal": 525.50,
            "multaAtraso": 0.00,
            "jurosMora": 0.00,
            "valorPago": 0.00,
            "status": "a vencer"
        }
    ],
    "totalPago": 525.50,
    "totalDevido": 0.00,
    "proximaParcela": {
        "numeroParcela": 2,
        "dataVencimento": "15/05/2025",
        "valorParcelaOriginal": 525.50
    }
}

Exemplo: Parcela vencida

{
    "idCliente": "987.654.321-09",
    "idEmprestimo": "EMP-00456",
    "dataConsulta": "20/06/2025",
    "parcelas": [
        {
            "numeroParcela": 3,
            "dataVencimento": "15/06/2025",
            "dataPagamento": null,
            "valorParcelaOriginal": 525.50,
            "multaAtraso": 10.51,
            "jurosMora": 0.88,
            "valorPago": 0.00,
            "valorTotalDevido": 536.89,
            "status": "vencida"
        }
    ],
    "totalDevido": 536.89
}

{
    "idCliente": "987.654.321-09",
    "idEmprestimo": "EMP-00456",
    "numeroParcela": 3,
    "dataVencimento": "15/06/2025",
    "dataPagamento": "20/06/2025",
    "valorParcelaOriginal": 525.50,
    "multaAtraso": 10.51,
    "jurosMora": 0.88,
    "valorTotalDevido": 536.89,
    "valorPago": 536.89,
    "status": "paga",
    "mensagem": "Parcela 3 atualizada com sucesso. Pagamento registrado com multa e juros por 5 dias de atraso."
}

Exemplo:Pagamento parcial
{
    "idCliente": "987.654.321-09",
    "idEmprestimo": "EMP-00456",
    "numeroParcela": 3,
    "dataVencimento": "15/06/2025",
    "dataPagamento": "20/06/2025",
    "valorParcelaOriginal": 525.50,
    "multaAtraso": 10.51,
    "jurosMora": 0.88,
    "valorTotalDevido": 536.89,
    "valorPago": 400.00,
    "valorRestante": 136.89,
    "status": "vencida",
    "mensagem": "Pagamento parcial registrado. Resta 136.89 para quitar a parcela 3."
}
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

* `idCliente`: "456.789.012-34"
* `idEmprestimo`: "EMP-00789"
* `numeroParcela`: 5
* `dataPagamento`: "25/08/2025"
* `valorPago`: 492.26

**Dados do banco:**

* `dataVencimento`: "10/08/2025"
* `valorParcelaOriginal`: 480.25
* `status`: "vencida"

**Cálculos:**

* Dias de atraso:
    * 25/08/2025 - 10/08/2025 = 15 dias
* Multa:
    * 480.25 * 0.02 = 9.61
* Juros de mora:
    * 480.25 * 0.000333 * 15 = 2.40
* Valor total devido:
    * 480.25 + 9.61 + 2.40 = 492.26

**Resultado:**

Registro atualizado com `status = paga`, `dataPagamento = 25/08/2025`, `valorPago = 492.26`.

## 4. Observações

* **Multa e juros:** Aplicados apenas a parcelas vencidas, com valores fixos (multa 2%) e variáveis (juros 1% ao mês).
* **Pagamentos parciais:** Permitem quitar parte da dívida, ajustando o saldo devedor restante.
* **Histórico:** Todas as atualizações mantêm um log no banco para auditoria.
* **Data padrão:** Se `dataConsulta` não for fornecida, usa a data atual (22/02/2025).
* **Taxa de Abertura de Crédito (TAC):**
    * Cobrada no momento da contratação do empréstimo.
    * Cobre custos administrativos de análise de crédito e elaboração do contrato.
    * Pode ser um valor fixo ou um percentual do valor do empréstimo.
    * Exemplo de fórmula (se percentual): `TAC = valorEmprestimo * percentualTAC`
* **Imposto sobre Operações Financeiras (IOF):**
    * Imposto federal que incide sobre o valor total do empréstimo.
    * Pode ser pago à vista ou diluído nas parcelas.
    * Cálculo: O cálculo do IOF é complexo e varia de acordo com o prazo do empréstimo e o tipo de cliente. Geralmente, é um percentual sobre o valor do empréstimo, com uma alíquota diária adicional.
    * Recomenda-se consultar a tabela oficial do IOF para valores exatos.
* **Taxa de Cadastro:**
    * Cobrada para realizar o cadastro do cliente, especialmente se não possuir conta corrente no banco.
    * Valor fixo.
* **Taxa de Avaliação de Bens:**
    * Cobrada se o empréstimo exigir garantia de bens (veículos, imóveis).
    * Cobre os custos de avaliação dos bens.
    * Valor varia conforme o tipo de bem.
* **Taxa de Seguro (Seguro Prestamista):**
    * Garante o pagamento do empréstimo em caso de morte ou invalidez do cliente.
    * Opcional, mas pode ser exigido em alguns casos.
    * Valor varia conforme o perfil do cliente e o valor do empréstimo.
* **Taxa de Liquidação Antecipada:**
    * Cobrada se o cliente quitar o empréstimo antes do prazo previsto.
    * Regulamentada por lei, com limites máximos.
    * O cálculo depende do saldo devedor e do tempo restante para o término do contrato.
* **Custo Efetivo Total (CET):**
    * Indicador do custo total do empréstimo, incluindo todas as taxas e encargos.
    * Essencial para comparar diferentes ofertas de empréstimo.
    * O cálculo do CET é complexo e envolve diversas variáveis. Recomenda-se utilizar simuladores online ou consultar a instituição financeira.
* **Observação:** Todas as taxas e encargos devem ser informados de forma clara e transparente no contrato de empréstimo.
* **Observação:** O cliente tem o direito de solicitar o detalhamento de todas as taxas e esclarecer dúvidas antes de assinar o contrato.
* **Observação:** A legislação brasileira protege o consumidor de práticas abusivas.

## 5. Especificação das Faixas de Taxas de Juros

* **Faixas de taxas de juros:**
    * Baixo risco: 1,0% a 1,5% ao mês
    * Risco médio: 1,6% a 2,5% ao mês
    * Alto risco: 2,6% a 4,0% ao mês
* **Associação com valores e prazos:**
    * Valores de empréstimo mais altos e prazos mais longos podem resultar em taxas de juros mais altas.
    * Exemplo:
        * Cliente de baixo risco, empréstimo de R$ 10.000,00 em 24 meses: 1,2% ao mês.
        * Cliente de risco médio, empréstimo de R$ 20.000,00 em 36 meses: 2,0% ao mês.
* **Variação com o score:**
    * Clientes com score de crédito mais alto podem obter taxas de juros mais baixas.
    * Tabela Pontuação Score x Taxa de Juros:

| Score de Crédito | Faixa de Taxa de Juros |
| :--------------- | :---------------------- |
| 700-1000        | 1,0% - 1,5%             |
| 500-699         | 1,6% - 2,5%             |
| 0-499           | 2,6% - 4.0%             |

## 6. Aprofundamento das Regras de Antecipação de Parcelas

* **Fórmulas de cálculo dos descontos:**
    * Desconto = Valor da parcela / (1 + Taxa de juros mensal) ^ Número de meses de antecipação
* **Opções de pagamento:**
    * Pagamento único: Quitação de todas as parcelas restantes com desconto.
    * Pagamento da primeira e última parcela: Quitação das duas parcelas com desconto.
    * Pagamento parcial: pagamento de parcelas aleatórias, com o recálculo do saldo devedor.
* **Exemplos de cálculo:**
    * Parcela de R$ 1.000,00, taxa de juros de 2% ao mês, antecipação de 3 parcelas:
        * Desconto = 1000 / (1 + 0.02) ^ 3 = R$ 942.32
* **Recálculo do saldo devedor:**
    * Após a antecipação, o saldo devedor é recalculado, subtraindo o valor presente das parcelas antecipadas.
    * O cliente pode solicitar um novo cronograma de pagamento com o saldo devedor atualizado.
* **Fórmulas para recálculo do saldo devedor:**
    * Valor presente da parcela antecipada = Valor da parcela / (1 + Taxa de juros mensal) ^ Número de meses de antecipação
    * Saldo devedor restante = Saldo devedor anterior - Valor presente da parcela antecipada
* **Exemplo de recálculo do saldo devedor:**
    * Saldo devedor anterior: R$ 5.000,00
    * Parcela antecipada: R$ 1.000,00
    * Taxa de juros mensal: 2%
    * Meses de antecipação: 3
    * Valor presente da parcela antecipada: 1000 / (1 + 0.02) ^ 3 = R$ 942.32
    * Saldo devedor restante: 5000 - 942.32 = R$ 4.057,68

## 7. Exemplos Práticos de Cálculos

* **Capacidade de pagamento:**
    * Renda líquida: R$ 5.000,00
    * Percentual máximo de comprometimento: 30%
    * Capacidade de pagamento: 5000 \* 0,30 = R$ 1.500,00
* **Cálculo de taxas de juros:**
    * Score de crédito: 750 (baixo risco)
    * Valor do empréstimo: R$ 20.000,00
    * Prazo: 36 meses
    * Taxa de juros: 1,3% ao mês
* **Cálculo de multas e juros:**
    * Parcela de R$ 1.000,00, 10 dias de atraso:
        * Multa: 1000 \* 0,02 = R$ 20,00
        * Juros: 1000 \* 0,000333 \* 10 = R$ 3,33
        * Valor total devido: R$ 1.023,33
* **Cálculo de amortização:**
    * Para o cálculo de amortização, podemos utilizar o sistema Price, com a seguinte fórmula:
        * PMT = PV * [i * (1 + i)^n] / [(1 + i)^n - 1]
    * Onde:
        * PMT = Valor da parcela
        * PV = Valor presente (valor do empréstimo)
        * i = Taxa de juros por período
        * n = Número total de períodos

## 8. Regras de negócio por idade

- **Idade mínima:** 18 anos.
- **Idade máxima:** 75 anos (com condições especiais acima de 65 anos).
- **Variação de condições:**

  - Clientes com mais de 65 anos podem ter prazos de pagamento mais curtos e taxas de juros mais altas.
  - Clientes com mais de 70 anos podem precisar de garantias adicionais.

- Tabela Faixa Etária X Prazos:

| Faixa Etária | Prazos Máximos | Taxas de Juros | Garantias Adicionais |
| :----------- | :------------- | :------------- | :------------------- |
| 18-65 anos   | Até 60 meses   | Taxas padrão   | Não aplicável        |
| 66-70 anos   | Até 48 meses   | +0,5%          | Opcional             |
| 71-75 anos   | Até 36 meses   | +1,0%          | Obrigatório          |

## 10. Informações adicionais

- **Cadastro do cliente:**
  - Campos obrigatórios: Nome completo, CPF, data de nascimento, endereço, telefone, e-mail, renda.
  - Campos opcionais: Informações adicionais de contato, referências.
  - Validações de segurança: Validação de CPF, validação de email, confirmação de telefone, criptografia de dados sensíveis.
  - Processo de cadastro:
    1.  Coleta de dados pessoais e financeiros do cliente.
    2.  Validação dos dados fornecidos.
    3.  Criação de um perfil de cliente no sistema.
    4.  Armazenamento seguro dos dados do cliente.
- **Comunicação com o cliente:**
  - Canais: SMS, WhatsApp, e-mail, notificações no aplicativo.
  - Tipos de mensagens:
    - Confirmação de solicitação de empréstimo.
    - Atualização do status da solicitação.
    - Lembrete de vencimento de parcelas.
    - Confirmação de pagamento de parcelas.
    - Ofertas de novos produtos e serviços.

    ## Glossário

**Amortização**
: Processo de pagamento de uma dívida em parcelas periódicas, que incluem tanto o valor principal quanto os juros.

**CET (Custo Efetivo Total)**
: Indicador que representa o custo total de um empréstimo, incluindo todas as taxas e encargos.

**IOF (Imposto sobre Operações Financeiras)**
: Imposto federal que incide sobre operações de crédito, câmbio, seguros e títulos.

**Score de crédito**
: Pontuação que indica o risco de inadimplência de um cliente, com base em seu histórico de crédito.

**TAC (Taxa de Abertura de Crédito)**
: Taxa cobrada no momento da contratação do empréstimo, para cobrir os custos administrativos da instituição financeira.

**Taxa de Avaliação de Bens**
: Taxa cobrada quando o empréstimo exige garantia de bens, para cobrir os custos de avaliação desses bens.

**Taxa de Cadastro**
: Taxa cobrada para realizar o cadastro do cliente, especialmente se ele não possuir conta corrente no banco.

**Taxa de Liquidação Antecipada**
: Taxa cobrada quando o cliente quita o empréstimo antes do prazo previsto.

**Taxa de Seguro (Seguro Prestamista)**
: Taxa cobrada para garantir o pagamento do empréstimo em caso de morte ou invalidez do cliente.
