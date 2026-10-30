# 📚 Anotações de Aula — SQL Server

# 1. Fundamentos e Cláusulas Básicas

## 1.1. Consultas, Filtros e Ordenação

A instrução `SELECT` permite consultar dados armazenados nas tabelas. Combinada com cláusulas como `WHERE`, `ORDER BY` e operadores lógicos, possibilita filtrar e organizar os resultados.

```sql
-- Selecionar salários únicos (sem duplicidade)
SELECT DISTINCT F.Salario
FROM FUNCIONARIO AS F;

-- Filtrar funcionário pelo primeiro nome
SELECT *
FROM FUNCIONARIO AS F
WHERE F.PNome = 'Carlos';

-- Funcionários do sexo masculino com salário maior ou igual a 30.000
SELECT *
FROM FUNCIONARIO AS F
WHERE F.Sexo = 'M' AND F.Salario >= 30000;

-- Funcionários cujo endereço contenha 'São Paulo' OU 'Curitiba'
SELECT *
FROM FUNCIONARIO AS F
WHERE F.Endereco LIKE '%São Paulo%'
   OR F.Endereco LIKE '%Curitiba%';

-- Funcionários cujo endereço NÃO contenha 'São Paulo'
SELECT *
FROM FUNCIONARIO AS F
WHERE NOT F.Endereco LIKE '%São Paulo%';

-- Calcular o custo anual e ordenar do maior para o menor
SELECT
    F.Pnome AS 'Nome',
    F.Minicial AS 'Sobrenome',
    F.Salario,
    F.Salario * 12 AS 'CustoAnual'
FROM FUNCIONARIO AS F
ORDER BY CustoAnual DESC;

-- Funcionários que não possuem supervisor
SELECT *
FROM FUNCIONARIO AS F
WHERE F.Cpf_supervisor IS NULL;

-- Funcionários que possuem supervisor
SELECT *
FROM FUNCIONARIO AS F
WHERE F.Cpf_supervisor IS NOT NULL;

-- Retornar os 3 funcionários com os maiores salários
SELECT TOP 3 *
FROM FUNCIONARIO AS F
ORDER BY F.Salario DESC;

-- Funcionários que nasceram no ano de 1972
SELECT *
FROM FUNCIONARIO AS F
WHERE F.Datanasc LIKE '1972%';
```

### Principais operadores e cláusulas

| Operador / Cláusula | Finalidade |
|---|---|
| `DISTINCT` | Elimina resultados duplicados. |
| `WHERE` | Filtra registros. |
| `AND` | Todas as condições devem ser verdadeiras. |
| `OR` | Pelo menos uma condição deve ser verdadeira. |
| `NOT` | Inverte uma condição lógica. |
| `LIKE` | Pesquisa padrões de texto. |
| `%` | Representa qualquer sequência de caracteres. |
| `IS NULL` | Verifica valores nulos. |
| `IS NOT NULL` | Verifica valores não nulos. |
| `ORDER BY` | Ordena os resultados. |
| `ASC` | Ordem crescente. |
| `DESC` | Ordem decrescente. |
| `TOP` | Limita a quantidade de registros retornados. |

## 1.2. Funções de Agregação

As funções de agregação permitem realizar cálculos sobre conjuntos de registros.

| Função | Finalidade |
|---|---|
| `MIN()` | Retorna o menor valor. |
| `MAX()` | Retorna o maior valor. |
| `AVG()` | Calcula a média. |
| `SUM()` | Calcula a soma. |
| `COUNT()` | Conta os registros. |

```sql
-- Retornar o menor salário cadastrado
SELECT MIN(Salario)
FROM FUNCIONARIO;

-- Buscar os dados do funcionário que recebe o menor salário
SELECT *
FROM FUNCIONARIO AS F
WHERE F.Salario = (
    SELECT MIN(Salario)
    FROM FUNCIONARIO
);

-- Armazenar o menor salário em uma variável
DECLARE @salario_min DECIMAL(10,2);

SET @salario_min = (
    SELECT MIN(Salario)
    FROM FUNCIONARIO
);

-- Consultar os funcionários que recebem o menor salário
SELECT *
FROM FUNCIONARIO AS F
WHERE F.Salario = @salario_min;

-- Contar o número total de funcionários
SELECT COUNT(F.Cpf)
FROM FUNCIONARIO AS F;

-- Contar o número total de dependentes
SELECT COUNT(D.Nome_dependente)
FROM DEPENDENTE AS D;

-- Somar o total de funcionários e dependentes
SELECT
    (SELECT COUNT(F.Cpf) FROM FUNCIONARIO AS F) +
    (SELECT COUNT(D.Nome_dependente) FROM DEPENDENTE AS D)
    AS 'QtdPessoas';

-- Calcular a média salarial dos funcionários
SELECT AVG(F.Salario) AS MediaSalarial
FROM FUNCIONARIO AS F;

-- Funcionários que recebem abaixo da média salarial
SELECT *
FROM FUNCIONARIO AS F
WHERE F.Salario < (
    SELECT AVG(Salario)
    FROM FUNCIONARIO
)
ORDER BY F.Salario ASC;

-- Calcular o custo anual total de salários da empresa
SELECT SUM(F.Salario) * 12 AS 'CustoAnual'
FROM FUNCIONARIO AS F;
```

## 1.3. Subconsultas (Subqueries)

Uma subconsulta é uma consulta SQL inserida dentro de outra consulta.

Pode ser utilizada para comparar valores, realizar filtros e obter resultados que dependem de outra consulta.

Exemplo:

```sql
-- Buscar funcionários que recebem o menor salário
SELECT *
FROM FUNCIONARIO AS F
WHERE F.Salario = (
    SELECT MIN(Salario)
    FROM FUNCIONARIO
);
```

Nesse exemplo, a consulta interna encontra o menor salário e a consulta externa retorna os funcionários que recebem esse valor.

---

# 2. Controle de Acesso e Permissões

O SQL Server permite controlar quais usuários podem acessar o banco de dados e quais operações podem executar.

## 2.1. Criação de LOGIN e USER

- `LOGIN`: cria uma credencial de acesso ao servidor SQL Server.
- `USER`: cria um usuário dentro de um banco de dados, associado a um LOGIN.

```sql
-- Acessar o banco de dados master
USE [master];
GO

-- Criar um LOGIN
CREATE LOGIN [Juca]
WITH PASSWORD = N'12345',
MUST_CHANGE,
DEFAULT_DATABASE = [LOJA];
GO

-- Acessar o banco de dados LOJA
USE [LOJA];
GO

-- Criar um USER associado ao LOGIN
CREATE USER [Juca] FOR LOGIN [Juca];
GO
```

## 2.2. Conceder Permissões — GRANT

O comando `GRANT` concede permissões para um usuário executar determinadas operações.

```sql
-- Permitir SELECT e INSERT na tabela CATEGORIA
GRANT SELECT, INSERT ON [dbo].[CATEGORIA] TO Juca;

-- Permitir SELECT e INSERT na tabela FORNECEDOR
GRANT SELECT, INSERT ON [dbo].[FORNECEDOR] TO Juca;

-- Permitir SELECT e INSERT na tabela PRODUTO
GRANT SELECT, INSERT ON [dbo].[PRODUTO] TO Juca;
```

## 2.3. Negar Permissões — DENY

O comando `DENY` nega explicitamente uma permissão ao usuário.

```sql
-- Impedir que o usuário insira registros na tabela CATEGORIA
DENY INSERT ON [dbo].[CATEGORIA] TO Juca;
```

## 2.4. Revogar Permissões — REVOKE

O comando `REVOKE` remove uma permissão anteriormente concedida ou negada.

```sql
-- Remover as permissões SELECT e INSERT da tabela PRODUTO
REVOKE SELECT, INSERT ON [dbo].[PRODUTO] TO Juca;
```

### Diferenças entre os comandos

| Comando | Finalidade |
|---|---|
| `GRANT` | Concede uma permissão. |
| `DENY` | Nega explicitamente uma permissão. |
| `REVOKE` | Remove uma concessão ou negação explícita. |

---

# 3. Views (Exibições)

## 3.1. Conceito

**Views** são tabelas virtuais baseadas em consultas `SELECT`.

Elas permitem encapsular consultas complexas, facilitando o acesso aos dados e a reutilização de consultas.

Uma View não precisa armazenar os dados propriamente ditos: normalmente, ela apresenta os resultados da consulta definida em sua criação.

## 3.2. Criar ou Alterar uma View

O comando `CREATE OR ALTER VIEW` permite criar uma View caso ela não exista ou alterar sua definição caso já exista.

```sql
-- Criar ou alterar uma View para o Departamento de Pessoal (DP)
CREATE OR ALTER VIEW vw_rh_funcionario_dp
AS
SELECT
    CONCAT(F.Pnome, ' ', F.Minicial, '. ', F.Unome) AS 'Nome',
    DATEDIFF(YEAR, F.Datanasc, GETDATE()) AS 'Idade',
    F.Endereco AS 'Endereço',
    F.Data_Admissao AS 'Admissão',
    D.Dnome AS 'Departamento'
FROM FUNCIONARIO AS F
JOIN DEPARTAMENTO AS D
    ON F.Dnr = D.Dnumero;
GO
```

### Funções utilizadas no exemplo

| Função | Finalidade |
|---|---|
| `CONCAT()` | Concatena textos. |
| `DATEDIFF()` | Calcula a diferença entre datas. |
| `GETDATE()` | Retorna a data e hora atuais. |
| `JOIN` | Combina registros de tabelas relacionadas. |

## 3.3. Consultar uma View

Depois de criada, a View pode ser consultada como uma tabela.

```sql
-- Consultar todos os registros da View
SELECT *
FROM vw_rh_funcionario_dp;

-- Consultar filtrando pelo departamento
SELECT *
FROM vw_rh_funcionario_dp
WHERE Departamento = 'Matriz';

-- Consultar ordenando pelo departamento
SELECT *
FROM vw_rh_funcionario_dp
ORDER BY Departamento;
```

---

# 4. Triggers (Gatilhos)

## 4.1. Conceito

**Triggers** são blocos de código executados automaticamente em resposta a eventos de manipulação de dados.

Os principais eventos que podem disparar um Trigger são:

- `INSERT`: inserção de registros.
- `UPDATE`: atualização de registros.
- `DELETE`: exclusão de registros.

São utilizados principalmente para implementar regras de negócio, validações e auditoria de alterações.

## 4.2. Tabelas INSERTED e DELETED

Dentro dos Triggers, o SQL Server disponibiliza duas tabelas especiais:

| Tabela | Finalidade |
|---|---|
| `inserted` | Armazena os novos registros inseridos ou os novos valores após um UPDATE. |
| `deleted` | Armazena os registros excluídos ou os valores anteriores a um UPDATE. |

Comportamento conforme a operação:

| Operação | INSERTED | DELETED |
|---|---|---|
| INSERT | Novos registros | Vazia |
| UPDATE | Novos valores | Valores antigos |
| DELETE | Vazia | Registros excluídos |

## 4.3. Triggers AFTER

O Trigger `AFTER` é executado depois que a operação na tabela é concluída com sucesso.

O modificador `FOR` é sinônimo de `AFTER`.

### Criar tabela de LOG

Uma tabela de LOG permite registrar operações realizadas no banco de dados.

```sql
-- Criação da tabela de auditoria
CREATE TABLE LOG_FUNCIONARIO (
    log_id INT IDENTITY(100,1) PRIMARY KEY,
    cpf CHAR(11),
    operacao VARCHAR(10),
    data_hora DATETIME DEFAULT GETDATE()
);
GO
```

### Trigger para INSERT

```sql
-- Registrar inserções de funcionários
CREATE OR ALTER TRIGGER trg_for_insert_funcionario
ON FUNCIONARIO
AFTER INSERT
AS
BEGIN
    INSERT INTO LOG_FUNCIONARIO (cpf, operacao)
    SELECT I.Cpf, 'INSERT'
    FROM inserted AS I;
END;
GO
```

### Trigger para UPDATE

```sql
-- Registrar atualizações de funcionários
CREATE OR ALTER TRIGGER trg_for_update_funcionario
ON FUNCIONARIO
AFTER UPDATE
AS
BEGIN
    INSERT INTO LOG_FUNCIONARIO (cpf, operacao)
    SELECT I.Cpf, 'UPDATE'
    FROM inserted AS I;
END;
GO
```

### Trigger para DELETE

```sql
-- Registrar exclusões de funcionários
CREATE OR ALTER TRIGGER trg_for_delete_funcionario
ON FUNCIONARIO
AFTER DELETE
AS
BEGIN
    INSERT INTO LOG_FUNCIONARIO (cpf, operacao)
    SELECT D.Cpf, 'DELETE'
    FROM deleted AS D;
END;
GO
```

## 4.4. Trigger Unificado para INSERT, DELETE e UPDATE (IDU)

É possível criar um único Trigger para controlar as três operações.

Para diferenciar `INSERT` de `UPDATE`, verificamos a existência de registros nas tabelas `inserted` e `deleted`.

```sql
CREATE OR ALTER TRIGGER trg_for_IDU_funcionario
ON FUNCIONARIO
AFTER INSERT, DELETE, UPDATE
AS
BEGIN
    -- Verificar se existem registros em INSERTED
    IF EXISTS (SELECT 1 FROM inserted)
    BEGIN
        -- Se também existem em DELETED, é um UPDATE
        IF EXISTS (SELECT 1 FROM deleted)
        BEGIN
            INSERT INTO LOG_FUNCIONARIO (cpf, operacao)
            SELECT I.Cpf, 'UPDATE'
            FROM inserted AS I;
        END
        ELSE
        BEGIN
            -- Se só existem em INSERTED, é um INSERT
            INSERT INTO LOG_FUNCIONARIO (cpf, operacao)
            SELECT I.Cpf, 'INSERT'
            FROM inserted AS I;
        END
    END
    ELSE
    BEGIN
        -- Se não existem registros em INSERTED, é um DELETE
        INSERT INTO LOG_FUNCIONARIO (cpf, operacao)
        SELECT D.Cpf, 'DELETE'
        FROM deleted AS D;
    END
END;
GO
```

## 4.5. Controle de Fluxo e Validação de Colunas — UPDATE()

A função `UPDATE()` permite verificar se uma determinada coluna foi incluída em uma instrução `UPDATE`.

Ela pode ser utilizada para executar ações específicas dependendo da coluna envolvida na atualização.

```sql
-- Verificar se a coluna Dnome foi alterada
CREATE OR ALTER TRIGGER trg_for_update_nome
ON DEPARTAMENTO
AFTER UPDATE
AS
BEGIN
    DECLARE @nome_antigo VARCHAR(40),
            @nome_novo VARCHAR(40);

    SELECT @nome_antigo = D.Dnome
    FROM deleted AS D;

    SELECT @nome_novo = I.Dnome
    FROM inserted AS I;

    IF UPDATE(Dnome)
    BEGIN
        PRINT 'O nome era: ' + ISNULL(@nome_antigo, 'Nulo');
        PRINT 'O novo nome é: ' + ISNULL(@nome_novo, 'Nulo');
    END
    ELSE
    BEGIN
        PRINT 'O nome não foi alterado';
    END
END;
GO
```

### Funções utilizadas

| Função | Finalidade |
|---|---|
| `UPDATE(coluna)` | Verifica se uma coluna foi incluída na instrução UPDATE. |
| `ISNULL()` | Substitui um valor NULL por outro valor definido. |
| `PRINT` | Exibe uma mensagem. |

**Atenção:** `UPDATE(coluna)` verifica se a coluna participou da instrução, não necessariamente se seu valor foi efetivamente modificado.

## 4.6. Triggers INSTEAD OF

O Trigger `INSTEAD OF` intercepta uma operação e executa um bloco de código no lugar da instrução original.

Pode ser utilizado para implementar regras de negócio antes de permitir uma operação.

### Validar duplicidade antes de permitir INSERT

```sql
-- Validar duplicidade antes de permitir a inserção
CREATE OR ALTER TRIGGER trg_insteadof_inserir_dpt
ON DEPARTAMENTO
INSTEAD OF INSERT
AS
BEGIN
    DECLARE @nome VARCHAR(40),
            @numero INT;

    SELECT
        @nome = I.Dnome,
        @numero = I.Dnumero
    FROM inserted AS I;

    IF EXISTS (
        SELECT 1
        FROM DEPARTAMENTO AS D
        WHERE D.Dnome = @nome
    )
    BEGIN
        RAISERROR(
            'Já existe um departamento com esse nome',
            16,
            1
        );
    END
    ELSE
    BEGIN
        INSERT INTO DEPARTAMENTO (
            Dnumero,
            Dnome,
            Cpf_gerente,
            Data_inicio_gerente
        )
        SELECT
            I.Dnumero,
            I.Dnome,
            I.Cpf_gerente,
            I.Data_inicio_gerente
        FROM inserted AS I;
    END
END;
GO
```

### Bloquear exclusão de registros

```sql
-- Impedir a exclusão de departamentos
CREATE OR ALTER TRIGGER trg_insteadof_delete_dpt
ON DEPARTAMENTO
INSTEAD OF DELETE
AS
BEGIN
    PRINT 'NÃO DELETEI o registro - Operação cancelada pela regra de negócio.';
END;
GO
```

## 4.7. Validação Pós-Ação com ROLLBACK

O comando `ROLLBACK TRANSACTION` desfaz uma transação.

Pode ser utilizado em um Trigger `AFTER` para cancelar uma operação que já foi executada, mas que viola uma regra de negócio.

```sql
-- Verificar duplicidade após a inserção e reverter caso necessário
CREATE OR ALTER TRIGGER trg_after_Insere_dpt
ON DEPARTAMENTO
AFTER INSERT
AS
BEGIN
    DECLARE @nome VARCHAR(40);

    SELECT @nome = I.Dnome
    FROM inserted AS I;

    IF (
        SELECT COUNT(Dnome)
        FROM DEPARTAMENTO
        WHERE Dnome = @nome
    ) > 1
    BEGIN
        ROLLBACK TRANSACTION;

        RAISERROR(
            'Já existe um departamento com esse nome. Transação desfeita.',
            16,
            1
        );
    END
END;
GO
```

### Diferença entre AFTER e INSTEAD OF

| AFTER | INSTEAD OF |
|---|---|
| Executa depois da operação original. | Executa no lugar da operação original. |
| A operação já foi realizada. | A operação original é interceptada. |
| Pode desfazer a operação com ROLLBACK. | Pode decidir se a operação será executada. |
| Utilizado em auditoria e validações posteriores. | Utilizado em regras de negócio e validações prévias. |

## 4.8. Gerenciamento de Triggers

O SQL Server disponibiliza comandos para habilitar, desabilitar e consultar Triggers.

```sql
-- Desabilitar um Trigger específico
ALTER TABLE DEPARTAMENTO
DISABLE TRIGGER trg_insteadof_delete_dpt;

-- Habilitar um Trigger específico
ALTER TABLE DEPARTAMENTO
ENABLE TRIGGER trg_insteadof_delete_dpt;

-- Listar os Triggers existentes em uma tabela
EXEC sp_helptrigger DEPARTAMENTO;
```

### Resumo dos comandos

| Comando | Finalidade |
|---|---|
| `CREATE TRIGGER` | Cria um Trigger. |
| `ALTER TRIGGER` | Altera um Trigger. |
| `CREATE OR ALTER TRIGGER` | Cria ou altera um Trigger. |
| `DISABLE TRIGGER` | Desabilita um Trigger. |
| `ENABLE TRIGGER` | Habilita um Trigger. |
| `sp_helptrigger` | Lista os Triggers de uma tabela. |
| `ROLLBACK TRANSACTION` | Desfaz uma transação. |

---

# 5. Resumo Geral para Revisão

| Assunto | Principais comandos e funções |
|---|---|
| Consultas | `SELECT`, `DISTINCT`, `TOP` |
| Filtros | `WHERE`, `AND`, `OR`, `NOT`, `LIKE` |
| Ordenação | `ORDER BY`, `ASC`, `DESC` |
| Valores nulos | `IS NULL`, `IS NOT NULL` |
| Agregações | `MIN`, `MAX`, `AVG`, `SUM`, `COUNT` |
| Variáveis | `DECLARE`, `SET` |
| Controle de acesso | `LOGIN`, `USER`, `GRANT`, `DENY`, `REVOKE` |
| Views | `CREATE OR ALTER VIEW` |
| Triggers | `CREATE OR ALTER TRIGGER` |
| Operações | `INSERT`, `UPDATE`, `DELETE` |
| Tabelas especiais | `inserted`, `deleted` |
| Controle de Trigger | `AFTER`, `INSTEAD OF`, `UPDATE()` |
| Transações | `ROLLBACK TRANSACTION` |
| Gerenciamento | `ENABLE TRIGGER`, `DISABLE TRIGGER`, `sp_helptrigger` |

---

**Observações importantes para a prática:**

- Os exemplos de Triggers que utilizam variáveis escalares para ler `inserted` ou `deleted` consideram um único registro por operação. Em aplicações reais, é importante tratar operações com múltiplas linhas.
- O cálculo de idade com `DATEDIFF(YEAR, ...)` pode retornar um ano a mais caso o aniversário ainda não tenha ocorrido no ano atual.
- O comando `DENY` tem precedência sobre permissões concedidas por `GRANT`, respeitadas as regras de permissões e funções do SQL Server.
- O `REVOKE` não significa necessariamente que o usuário perdeu o acesso: ele remove a concessão ou negação explícita, podendo existir outras permissões herdadas.
