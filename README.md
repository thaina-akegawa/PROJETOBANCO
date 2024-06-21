# PROJETO-3

Claro, posso ajudar com a documentação do seu projeto de análise de dados. Vamos organizar a documentação de forma clara e completa. Aqui está um esboço da estrutura que você pode seguir:

### Documentação do Projeto 3

---

#### 1. **Introdução**

Este projeto tem como objetivo a análise de dados financeiros para identificar padrões e insights valiosos que possam auxiliar na tomada de decisões de crédito. Os dados foram obtidos de várias fontes e tratados para criar uma base consolidada e limpa.

#### 2. **Descrição dos Dados**

- **Tabelas Originais:**
  - `projeto-3-423920.projeto3.default`: Contém informações sobre inadimplência dos clientes.
  - `projeto-3-423920.projeto3.default_salary`: Contém informações sobre o salário do último mês e inadimplência.
  - `projeto-3-423920.projeto3.loans_detail`: Contém detalhes dos empréstimos dos clientes.
  - `projeto-3-423920.projeto3.loans_outstanding`: Contém informações sobre os empréstimos pendentes.
  - `projeto-3-423920.projeto3.user_info`: Contém informações gerais dos clientes.

#### 3. **Tratamento de Dados**

##### 3.1 **Identificação e Tratamento de Valores Nulos**

Identificamos que a variável `lastmonthsalary` tinha 7199 registros nulos, constituindo aproximadamente 20% dos dados. Para tratá-los, realizamos uma análise para verificar se esses registros estavam associados a clientes inadimplentes. Utilizamos o comando SQL:

```sql
SELECT
    COUNT(*) AS total_registros,
    SUM(CASE WHEN last_month_salary IS NULL THEN 1 ELSE 0 END) AS registros_nulos,
    SUM(CASE WHEN last_month_salary IS NOT NULL THEN 1 ELSE 0 END) AS registros_nao_nulos
FROM
    `projeto-3-423920.projeto3.user_info`;
```

Decidimos manter esses registros, atribuindo a eles o valor "sem_registro":

```sql
UPDATE `projeto-3-423920.projeto3.user_info`
SET last_month_salary = 'sem_registro'
WHERE last_month_salary IS NULL;
```

##### 3.2 **Identificação e Tratamento de Valores Duplicados**

Para identificar e remover valores duplicados nas tabelas, utilizamos a função `ROW_NUMBER`:

```sql
CREATE OR REPLACE TABLE `projeto-3-423920.projeto3.loans_outstanding_deduped` AS
SELECT * EXCEPT(row_num)
FROM (
    SELECT *, ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY user_id) AS row_num
    FROM `projeto-3-423920.projeto3.loans_outstanding`
)
WHERE row_num = 1
ORDER BY user_id ASC;
```

Repetimos o processo para todas as tabelas relevantes.

##### 3.3 **Combinação de Tabelas**

Criamos uma tabela combinada para consolidar todos os dados:

```sql
CREATE OR REPLACE TABLE `projeto-3-423920.projeto3.combined_data` AS
SELECT 
    COALESCE(ui.user_id, ds.user_id, ld.user_id, lo.user_id, df.user_id) AS user_id,
    ui.age,
    ui.sex,
    ui.number_dependents,
    COALESCE(ds.last_month_salary, 'sem_registro') AS last_month_salary,
    lo.loan_type,
    ld.debt_ratio,
    CASE
        WHEN ds.default_flag = 1 THEN 'inadimplentes'
        ELSE 'sem_inadimplência'
    END AS default_flag,
    CASE
        WHEN ld.more_90_days_overdue IS NOT NULL AND ld.more_90_days_overdue > 0 THEN '+90'
        WHEN ld.number_times_delayed_payment_loan_60_89_days IS NOT NULL AND ld.number_times_delayed_payment_loan_60_89_days > 0 THEN '60-89'
        WHEN ld.number_times_delayed_payment_loan_30_59_days IS NOT NULL AND ld.number_times_delayed_payment_loan_30_59_days > 0 THEN '30-59'
        ELSE 'No Delays'
    END AS dias_devedor,
    ld.using_lines_not_secured_personal_assets
FROM 
    `projeto-3-423920.projeto3.user_info_2` ui
LEFT JOIN 
    `projeto-3-423920.projeto3.default_salary` ds ON ui.user_id = ds.user_id
LEFT JOIN 
    `projeto-3-423920.projeto3.loans_detail` ld ON ui.user_id = ld.user_id
LEFT JOIN 
    `projeto-3-423920.projeto3.loans_outstanding2` lo ON ui.user_id = lo.user_id
LEFT JOIN 
    `projeto-3-423920.projeto3.default` df ON ui.user_id = df.user_id
ORDER BY user_id ASC;
```

#### 4. **Análise dos Dados**

Após o tratamento dos dados, realizamos várias análises para identificar padrões, tendências e insights:

- **Análise de Inadimplência:** Verificamos a proporção de clientes inadimplentes e não inadimplentes.
- **Análise de Atrasos de Pagamento:** Identificamos os diferentes níveis de atraso de pagamento (30-59 dias, 60-89 dias, mais de 90 dias).
- **Análise de Perfil de Cliente:** Analisamos o perfil dos clientes com base em idade, sexo, número de dependentes e salário do último mês.

#### 5. **Visualização dos Dados**

Para facilitar a compreensão dos dados, utilizamos o Looker Studio para criar visualizações interativas. Adicionamos todas as tabelas tratadas e combinadas para gerar gráficos e dashboards que mostram os principais insights.

#### 6. **Conclusão**

Este projeto demonstrou a importância do tratamento de dados e da combinação de várias fontes para gerar insights valiosos. Com uma base de dados consolidada e limpa, conseguimos identificar padrões de inadimplência e perfis de clientes que podem ajudar na tomada de decisões de crédito mais informadas.


