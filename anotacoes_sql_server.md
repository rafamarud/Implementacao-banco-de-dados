# 📚 Anotações de Aula — SQL Server

## 1. Views (Exibições)

**Views** funcionam como tabelas virtuais baseadas em uma consulta `SELECT`. Elas encapsulam queries complexas para simplificar o acesso aos dados.

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
JOIN DEPARTAMENTO AS D ON F.Dnr = D.Dnumero;
GO

-- Consultar dados filtrando pela View
SELECT *
FROM vw_rh_funcionario_dp 
WHERE Departamento = 'Matriz';

-- Consultar dados ordenando pela View
SELECT * 
FROM vw_rh_funcionario_dp 
ORDER BY Departamento;
```

---

## 2. Triggers (Gatilhos)

Triggers são blocos de código executados automaticamente em resposta a eventos de manipulação de dados (`INSERT`, `UPDATE`, `DELETE`).

### 🔹 Estrutura de Log com Triggers Individuais (`AFTER`)
O modificador `FOR` é sinônimo de `AFTER`. O gatilho dispara **depois** que a ação foi concluída com sucesso na tabela.

```sql
-- Criação da tabela de auditoria/log
CREATE TABLE LOG_FUNCIONARIO (
    log_id INT IDENTITY(100,1) PRIMARY KEY,
    cpf CHAR(11),
    operacao VARCHAR(10),
    data_hora DATETIME DEFAULT GETDATE()
);
GO

-- TRIGGER de log para INSERT
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

-- TRIGGER de log para UPDATE
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

-- TRIGGER de log para DELETE
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

### 🔹 Trigger Unificado para Múltiplas Operações (I/D/U)
*(Correção lógica aplicada: Para diferenciar `INSERT` de `UPDATE`, validamos se existem registros simultâneos em `inserted` e `deleted`)*.

```sql
CREATE OR ALTER TRIGGER trg_for_IDU_funcionario
ON FUNCIONARIO
AFTER INSERT, DELETE, UPDATE
AS
BEGIN
    -- Se existem registros em INSERTED
    IF EXISTS (SELECT 1 FROM inserted)
    BEGIN
        -- Se também existem em DELETED, é um UPDATE
        IF EXISTS (SELECT 1 FROM deleted)
            INSERT INTO LOG_FUNCIONARIO (cpf, operacao)
            SELECT I.Cpf, 'UPDATE' FROM inserted AS I;
        ELSE
            -- Se só existem em INSERTED, é um INSERT
            INSERT INTO LOG_FUNCIONARIO (cpf, operacao) 
            SELECT I.Cpf, 'INSERT' FROM inserted AS I;
    END     
    ELSE
    -- Se não existem registros em INSERTED, mas o gatilho rodou, é um DELETE
    BEGIN
        INSERT INTO LOG_FUNCIONARIO (cpf, operacao)
        SELECT D.Cpf, 'DELETE' FROM deleted AS D;
    END
END;
GO
```

### 🔹 Controle de Fluxo e Validação de Colunas (`UPDATE()`)

```sql
-- Verificar se uma coluna específica (Dnome) foi alterada
CREATE OR ALTER TRIGGER trg_for_update_nome
ON DEPARTAMENTO
AFTER UPDATE
AS 
BEGIN
    DECLARE @nome_antigo VARCHAR(40),
            @nome_novo VARCHAR(40);
            
    SELECT @nome_antigo = D.Dnome FROM deleted AS D;
    SELECT @nome_novo = I.Dnome FROM inserted AS I;
    
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

### 🔹 Gatilhos `INSTEAD OF` e Regras de Negócio
O gatilho `INSTEAD OF` intercepta a ação do usuário e executa o bloco de código **no lugar** da instrução original.

```sql
-- Validar duplicidade antes de permitir o INSERT
CREATE OR ALTER TRIGGER trg_insteadof_inserir_dpt
ON DEPARTAMENTO
INSTEAD OF INSERT
AS
BEGIN
    DECLARE @nome VARCHAR(40), @numero INT;
    SELECT @nome = I.Dnome, @numero = I.Dnumero FROM inserted AS I;
    
    IF EXISTS(SELECT 1 FROM DEPARTAMENTO AS D WHERE D.Dnome = @nome)
    BEGIN
        RAISERROR('Já existe um departamento com esse nome', 16, 1);
    END
    ELSE
    BEGIN
        INSERT INTO DEPARTAMENTO (Dnumero, Dnome, Cpf_gerente, Data_inicio_gerente)
        SELECT I.Dnumero, I.Dnome, I.Cpf_gerente, I.Data_inicio_gerente 
        FROM inserted AS I;
    END
END;
GO

-- Bloquear a exclusão de registros de forma descritiva
CREATE OR ALTER TRIGGER trg_insteadof_delete_dpt
ON DEPARTAMENTO
INSTEAD OF DELETE
AS
BEGIN
    PRINT 'NÃO DELETEI o registro - Operação cancelada pela regra de negócio.';
END;
GO
```

### 🔹 Validação Pós-Ação com `ROLLBACK`

```sql
-- Verificar duplicidade após inserção e reverter caso infrinja a regra
CREATE OR ALTER TRIGGER trg_after_Insere_dpt
ON DEPARTAMENTO
AFTER INSERT
AS
BEGIN
    DECLARE @nome VARCHAR(40);
    SELECT @nome = I.Dnome FROM inserted AS I;

    IF ((SELECT COUNT(Dnome) FROM DEPARTAMENTO WHERE Dnome = @nome) > 1)
    BEGIN
        ROLLBACK TRANSACTION;
        RAISERROR('Já existe um departamento com esse nome. Transação desfeita.', 16, 1);
    END
END;
GO
```

### 🔹 Gerenciamento de Triggers (Comandos Úteis)

```sql
-- Desabilitar um Trigger específico
ALTER TABLE DEPARTAMENTO DISABLE TRIGGER trg_insteadof_delete_dpt;

-- Habilitar um Trigger específico
ALTER TABLE DEPARTAMENTO ENABLE TRIGGER trg_insteadof_delete_dpt;

-- Listar os Triggers existentes em uma tabela específica
EXEC sp_helptrigger DEPARTAMENTO;
```

---

## 3. Fundamentos e Cláusulas Básicas (`SELECT`, `WHERE`, Agregações)

### 🔹 Filtros, Operadores Lógicos e Ordenação

```sql
-- Selecionar salários únicos (sem duplicidade) na tabela de Funcionários
SELECT DISTINCT F.Salario
FROM FUNCIONARIO AS F;

-- Filtrar funcionário pelo primeiro nome exatamente igual a 'Carlos'
SELECT *
FROM FUNCIONARIO AS F
WHERE F.PNome = 'Carlos';

-- Filtrar funcionários do sexo masculino que possuem salário maior ou igual a 30.000
SELECT *
FROM FUNCIONARIO AS F
WHERE F.Sexo = 'M' AND F.Salario >= 30000;

-- Selecionar funcionários cujo endereço contenha 'São Paulo' OU 'Curitiba'
SELECT *
FROM FUNCIONARIO AS F
WHERE F.Endereco LIKE '%São Paulo%' 
   OR F.Endereco LIKE '%Curitiba%';

-- Selecionar funcionários cujo endereço NÃO contenha 'São Paulo'
SELECT *
FROM FUNCIONARIO AS F
WHERE NOT F.Endereco LIKE '%São Paulo%';

-- Calcular o custo anual do funcionário e ordenar do maior para o menor custo
SELECT 
    F.Pnome AS 'Nome', 
    F.Minicial AS 'Sobrenome', 
    F.Salario,
    F.Salario * 12 AS 'CustoAnual'
FROM FUNCIONARIO AS F
ORDER BY CustoAnual DESC;

-- Selecionar funcionários que não possuem supervisor (Cpf_supervisor é Nulo)
SELECT *
FROM FUNCIONARIO AS F
WHERE F.Cpf_supervisor IS NULL;

-- Selecionar funcionários que possuem supervisor
SELECT * 
FROM FUNCIONARIO AS F
WHERE F.Cpf_supervisor IS NOT NULL;

-- Retornar os 3 funcionários com os maiores salários
SELECT TOP 3 *
FROM FUNCIONARIO AS F
ORDER BY F.Salario DESC;

-- Buscar funcionários que nasceram no ano de 1972
SELECT *
FROM FUNCIONARIO AS F
WHERE F.Datanasc LIKE '1972%';
```

### 🔹 Funções de Agregação (`MIN`, `MAX`, `AVG`, `SUM`, `COUNT`)

```sql
-- Retornar o menor salário cadastrado
SELECT MIN(Salario) 
FROM FUNCIONARIO;

-- Buscar os dados do funcionário que ganha o menor salário (usando subquery)
SELECT *
FROM FUNCIONARIO AS F
WHERE F.Salario = (SELECT MIN(Salario) FROM FUNCIONARIO);

-- Armazenar o menor salário em uma variável escalar e consultar os funcionários correspondentes
DECLARE @salario_min DECIMAL(10,2);
SET @salario_min = (SELECT MIN(Salario) FROM FUNCIONARIO);

SELECT * 
FROM FUNCIONARIO AS F
WHERE F.Salario = @salario_min;

-- Contar o número total de funcionários registrados
SELECT COUNT(F.Cpf)
FROM FUNCIONARIO AS F;

-- Contar o número total de dependentes registrados
SELECT COUNT(D.Nome_dependente)
FROM DEPENDENTE AS D;

-- Somar o total de funcionários e dependentes cadastrados no sistema
SELECT
    (SELECT COUNT(F.Cpf) FROM FUNCIONARIO AS F) + 
    (SELECT COUNT(D.Nome_dependente) FROM DEPENDENTE AS D) AS 'QtdPessoas';

-- Calcular a média salarial dos funcionários
SELECT AVG(F.Salario) AS MediaSalarial
FROM FUNCIONARIO AS F;

-- Selecionar funcionários que ganham abaixo da média salarial, ordenados do menor para o maior salário
SELECT *
FROM FUNCIONARIO AS F
WHERE F.Salario < (SELECT AVG(F.Salario) FROM FUNCIONARIO)
ORDER BY F.Salario ASC;

-- Calcular o custo anual total de salários da empresa
SELECT SUM(F.Salario) * 12 AS 'CustoAnual'
FROM FUNCIONARIO AS F;
```
