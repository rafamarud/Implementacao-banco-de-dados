# 📚 Anotações de Aula — SQL Server

---

## 1. Fundamentos e Cláusulas Básicas (`SELECT`, `WHERE`, Agregações)

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

---

## 2. Junções de Tabelas (`JOINs`) e Operadores de Conjunto

### 🔹 Tipos de JOIN
```sql
-- INNER JOIN: Selecionar primeiro nome, último nome e nome do departamento para funcionários do departamento de "Pesquisa"
SELECT F.Pnome, F.Unome, D.Dnome 
FROM Funcionario AS F
INNER JOIN Departamento AS D 
    ON D.Dnumero = F.Dnr 
WHERE D.Dnome = 'Pesquisa';

-- INNER JOIN: Listar nome do funcionário e nome do projeto para quem trabalha no "ProdutoX"
SELECT F.Pnome, P.Projnome
FROM Funcionario AS F
INNER JOIN TRABALHA_EM AS T 
    ON F.Cpf = T.Fcpf
INNER JOIN PROJETO AS P 
    ON T.Pnr = P.Projnumero
WHERE P.Projnome = 'ProdutoX';

-- INNER JOIN: Para projetos em "Mauá", listar número do projeto, número do departamento, e sobrenome, endereço e data de nascimento do gerente
SELECT Projnumero, Dnum, Unome, Endereco, Datanasc
FROM PROJETO 
INNER JOIN DEPARTAMENTO ON Dnum = Dnumero 
INNER JOIN FUNCIONARIO ON Cpf_gerente = Cpf
WHERE Projlocal = 'Mauá';

-- LEFT JOIN: Encontrar funcionários mesmo aqueles que não possuem departamento vinculado
SELECT *
FROM FUNCIONARIO
LEFT JOIN DEPARTAMENTO 
    ON FUNCIONARIO.Dnr = DEPARTAMENTO.Dnumero;

-- LEFT JOIN: Encontrar departamentos que não possuem nenhum funcionário alocado
SELECT 
    DEPARTAMENTO.Dnome,
    DEPARTAMENTO.Dnumero
FROM DEPARTAMENTO
LEFT JOIN FUNCIONARIO 
    ON DEPARTAMENTO.Dnumero = FUNCIONARIO.Dnr
WHERE FUNCIONARIO.Dnr IS NULL;

-- Alternativa 1 (NOT IN): Encontrar departamentos sem funcionários
SELECT Dnome, Dnumero
FROM DEPARTAMENTO
WHERE Dnumero NOT IN (SELECT Dnr FROM FUNCIONARIO WHERE Dnr IS NOT NULL);

-- Alternativa 2 (NOT EXISTS): Encontrar departamentos sem funcionários
SELECT Dnome, Dnumero
FROM DEPARTAMENTO d
WHERE NOT EXISTS (
    SELECT 1 
    FROM FUNCIONARIO f 
    WHERE f.Dnr = d.Dnumero
);

-- RIGHT JOIN: Retorna todos os departamentos, vinculando os funcionários quando houver
SELECT *   
FROM FUNCIONARIO
RIGHT JOIN DEPARTAMENTO 
    ON FUNCIONARIO.Dnr = DEPARTAMENTO.Dnumero;

-- FULL JOIN: Retorna todos os funcionários e todos os departamentos, associados ou não
SELECT *
FROM FUNCIONARIO
FULL JOIN DEPARTAMENTO 
    ON FUNCIONARIO.Dnr = DEPARTAMENTO.Dnumero;

-- SELF JOIN: Relacionar a tabela FUNCIONARIO com ela mesma para exibir o nome do funcionário e o nome do seu supervisor
SELECT
    F.Pnome AS Nome_Funcionario,
    F.Unome AS Sobrenome_Funcionario,
    S.Pnome AS Nome_Supervisor,
    S.Unome AS Sobrenome_Supervisor
FROM FUNCIONARIO AS F
JOIN FUNCIONARIO AS S 
    ON F.Cpf_supervisor = S.Cpf;
```

### 🔹 Operadores de Conjunto (`UNION`, `EXCEPT`, `INTERSECT`)
```sql
-- UNION: Listar todos os nomes únicos de projetos e departamentos (sem duplicatas)
SELECT Dnome AS Nome FROM DEPARTAMENTO
UNION
SELECT Projnome AS Nome FROM PROJETO;

-- UNION: Listar todas as cidades únicas onde a empresa possui atividade (departamento ou projeto)
SELECT Dlocal AS Cidade FROM LOCALIZACAO_DEP
UNION 
SELECT Projlocal AS Cidade FROM PROJETO;

-- UNION ALL: Listar todas as cidades com presença de departamento ou projeto (mantendo duplicatas)
SELECT Dlocal AS Cidade FROM LOCALIZACAO_DEP
UNION ALL
SELECT Projlocal AS Cidade FROM PROJETO;

-- EXCEPT: Listar CPFs de funcionários que NÃO são gerentes de nenhum departamento
SELECT Cpf FROM FUNCIONARIO
EXCEPT
SELECT Cpf_gerente FROM DEPARTAMENTO;

-- EXCEPT: Encontrar Funcionários que NÃO são Supervisores
SELECT Pnome, Unome
FROM FUNCIONARIO
WHERE Cpf IN (
    SELECT Cpf FROM FUNCIONARIO
    EXCEPT 
    SELECT Cpf_supervisor FROM FUNCIONARIO WHERE Cpf_supervisor IS NOT NULL
);

-- INTERSECT: Encontrar CPF de funcionários que TAMBÉM são gerentes de departamento
SELECT Cpf FROM FUNCIONARIO
INTERSECT
SELECT Cpf_gerente FROM DEPARTAMENTO;

-- INTERSECT: Encontrar Funcionários que SÃO Supervisores
SELECT Pnome, Unome
FROM FUNCIONARIO
WHERE Cpf IN (
    SELECT Cpf FROM FUNCIONARIO
    INTERSECT 
    SELECT Cpf_supervisor FROM FUNCIONARIO
);
```

---

## 3. Agrupamento e Subqueries (`GROUP BY`, `HAVING`, `EXISTS`, `ANY`, `ALL`)

```sql
-- GROUP BY: Contar o número de funcionários por departamento
SELECT 
    DEPARTAMENTO.Dnome, 
    COUNT(FUNCIONARIO.Cpf) AS NumeroFuncionarios
FROM FUNCIONARIO
JOIN DEPARTAMENTO ON FUNCIONARIO.Dnr = DEPARTAMENTO.Dnumero
GROUP BY DEPARTAMENTO.Dnome;

-- GROUP BY: Somar o total dos salários por departamento
SELECT 
    DEPARTAMENTO.Dnome, 
    SUM(FUNCIONARIO.Salario) AS SalarioTotal
FROM FUNCIONARIO
JOIN DEPARTAMENTO ON FUNCIONARIO.Dnr = DEPARTAMENTO.Dnumero
GROUP BY DEPARTAMENTO.Dnome;

-- GROUP BY: Calcular a média de horas trabalhadas por projeto
SELECT 
    PROJETO.Projnome, 
    AVG(TRABALHA_EM.Horas) AS MediaHorasTrabalhadas
FROM TRABALHA_EM
JOIN PROJETO ON TRABALHA_EM.Pnr = PROJETO.Projnumero
GROUP BY PROJETO.Projnome;

-- GROUP BY: Quantidade de funcionários agrupados por sexo
SELECT Sexo, COUNT(*) AS NumeroFuncionarios
FROM FUNCIONARIO
GROUP BY Sexo;

-- GROUP BY: Retornar o maior salário de cada departamento
SELECT 
    DEPARTAMENTO.Dnome, 
    MAX(FUNCIONARIO.Salario) AS MaiorSalario
FROM FUNCIONARIO
JOIN DEPARTAMENTO ON FUNCIONARIO.Dnr = DEPARTAMENTO.Dnumero
GROUP BY DEPARTAMENTO.Dnome;

-- GROUP BY: Número de projetos localizados em cada cidade
SELECT Projlocal, COUNT(*) AS NumeroProjetos
FROM PROJETO
GROUP BY Projlocal;

-- HAVING: Filtrar departamentos que possuem mais de 3 funcionários
SELECT 
    DEPARTAMENTO.Dnome, 
    COUNT(FUNCIONARIO.Cpf) AS NumeroFuncionarios
FROM FUNCIONARIO
JOIN DEPARTAMENTO ON FUNCIONARIO.Dnr = DEPARTAMENTO.Dnumero
GROUP BY DEPARTAMENTO.Dnome
HAVING COUNT(FUNCIONARIO.Cpf) > 3;

-- HAVING: Listar projetos com total de horas trabalhadas maior ou igual a 50
SELECT 
    PROJETO.Projnome, 
    SUM(TRABALHA_EM.Horas) AS TotalHoras
FROM TRABALHA_EM
JOIN PROJETO ON TRABALHA_EM.Pnr = PROJETO.Projnumero
GROUP BY PROJETO.Projnome
HAVING SUM(TRABALHA_EM.Horas) >= 50;

-- EXISTS: Listar funcionários que exercem o papel de gerente de algum departamento
SELECT Pnome, Unome, Cpf
FROM FUNCIONARIO
WHERE EXISTS (
    SELECT 1 
    FROM DEPARTAMENTO 
    WHERE DEPARTAMENTO.Cpf_gerente = FUNCIONARIO.Cpf
);

-- EXISTS: Listar departamentos que possuem pelo menos um projeto associado
SELECT Dnome, Dnumero
FROM DEPARTAMENTO
WHERE EXISTS (
    SELECT 1 
    FROM PROJETO 
    WHERE PROJETO.Dnum = DEPARTAMENTO.Dnumero
);

-- ANY: Encontrar funcionários com salário maior do que qualquer funcionário da Administração
SELECT Pnome, Unome, Salario
FROM FUNCIONARIO
WHERE Salario > ANY (
    SELECT Salario 
    FROM FUNCIONARIO 
    JOIN DEPARTAMENTO ON FUNCIONARIO.Dnr = DEPARTAMENTO.Dnumero
    WHERE DEPARTAMENTO.Dnome = 'Administração'
);

-- ALL: Encontrar projetos cujo total de horas é maior do que o total de todos os projetos de São Paulo
SELECT Projnome, SUM(TRABALHA_EM.Horas) AS TotalHoras
FROM PROJETO
JOIN TRABALHA_EM ON PROJETO.Projnumero = TRABALHA_EM.Pnr
GROUP BY Projnome
HAVING SUM(TRABALHA_EM.Horas) > ALL (
    SELECT SUM(TRABALHA_EM.Horas) 
    FROM PROJETO
    JOIN TRABALHA_EM ON PROJETO.Projnumero = TRABALHA_EM.Pnr
    WHERE PROJETO.Projlocal = 'São Paulo'
    GROUP BY PROJETO.Projnumero
);
```

---

## 4. Programação T-SQL (Funções, Procedures e Controle de Fluxo)

### 🔹 Sintaxe e Controle de Fluxo (Declarações)
```sql
-- DECLARE & SET/SELECT: Criação e atribuição de variáveis
DECLARE @Nome VARCHAR(100), @Numero INT;

SET @Numero = 10;

SELECT @Nome = P.Nome
FROM PESSOAS AS P
WHERE ID = 10;

-- IF / ELSE IF / ELSE: Estrutura condicional
IF (Condição)
BEGIN
    -- Bloco de código caso verdadeiro
END
ELSE IF (Condição)
BEGIN
    -- Bloco código alternativo
END
ELSE
BEGIN
    -- Bloco padrão
END;

-- IIF: Função condicional embutida (pode ser usada em cláusulas SELECT)
SELECT IIF(Salario > 5000, 'Alto', 'Padrão') AS CategoriaSalario
FROM FUNCIONARIO;

-- CASE: Estrutura condicional de seleção
SELECT 
    Pnome,
    CASE 
        WHEN Salario > 5000 THEN 'Sênior'
        WHEN Salario BETWEEN 3000 AND 5000 THEN 'Pleno'
        ELSE 'Júnior'
    END AS Nivel
FROM FUNCIONARIO;

-- WHILE: Laço de repetição
WHILE (Condição)
BEGIN
    -- Executar bloco repetidamente enquanto a condição for verdadeira
END;
```

### 🔹 User-Defined Functions (UDFs)

#### 1. Funções Escalares
```sql
-- Função Escalar: Retorna o dobro do valor passado por parâmetro
CREATE OR ALTER FUNCTION fn_dobro(@Numero DECIMAL(10, 2))
RETURNS DECIMAL(10,2)
AS
BEGIN
    RETURN @Numero * 2;
END;
GO

-- Exemplos de uso da função escalar fn_dobro
SELECT dbo.fn_dobro(5);

SELECT F.Pnome, F.Unome, F.Salario, dbo.fn_dobro(F.Salario) AS 'Dobro'
FROM FUNCIONARIO AS F
WHERE F.Pnome = 'Maria';

SELECT F.Pnome, F.Unome, F.Salario, dbo.fn_dobro(F.Salario) AS 'Dobro'
FROM FUNCIONARIO AS F
WHERE F.Pnome = 'Carlos';

-- Buscar funcionários que ganham mais que o dobro do menor salário cadastrado
DECLARE @menor_salario DECIMAL(10,2);
SELECT @menor_salario = MIN(Salario) FROM FUNCIONARIO;

SELECT Pnome, Unome, F.Salario
FROM FUNCIONARIO AS F
WHERE F.Salario > dbo.fn_dobro(@menor_salario);

-- Função Escalar: Calcula a idade exata com base na data de nascimento
CREATE OR ALTER FUNCTION fn_calcular_idade(@data_nasc DATE)
RETURNS INT
AS
BEGIN
    DECLARE @idade INT;
    SET @idade = DATEDIFF(YEAR, @data_nasc, GETDATE());
    
    -- Ajuste para caso o aniversário ainda não tenha ocorrido no ano atual
    IF (MONTH(@data_nasc) > MONTH(GETDATE())
        OR (MONTH(@data_nasc) = MONTH(GETDATE()) AND DAY(@data_nasc) > DAY(GETDATE())))
        SET @idade = @idade - 1;
        
    RETURN @idade;
END;
GO

-- Exemplo de uso: Calcular idade dos dependentes
SELECT 
    D.Nome_dependente,
    D.Datanasc,
    dbo.fn_calcular_idade(D.Datanasc) AS 'Idade'
FROM DEPENDENTE AS D;

-- Exemplo de uso: Calcular idade dos funcionários e formatar data (DD/MM/YYYY)
SELECT 
    F.Pnome,
    F.Unome,
    CONVERT(VARCHAR, F.Datanasc, 103) AS 'Data de Nascimento',
    dbo.fn_calcular_idade(F.Datanasc) AS 'Idade'
FROM FUNCIONARIO AS F;
```

#### 2. Funções In-Line (Tabela In-line)
```sql
-- Função In-Line: Retorna uma tabela contendo os funcionários de um determinado departamento
CREATE OR ALTER FUNCTION fn_func_dpt(@nome_dpt VARCHAR(50))
RETURNS TABLE
AS
RETURN (
    SELECT F.Pnome, F.Unome
    FROM FUNCIONARIO AS F
    JOIN DEPARTAMENTO AS D ON F.Dnr = D.Dnumero
    WHERE D.Dnome = @nome_dpt
);
GO

-- Executando a função de tabela
SELECT * FROM dbo.fn_func_dpt('Pesquisa');
```

#### 3. Funções Multi-Statement (Múltiplos Comandos)
```sql
-- Função Multi-Statement: Retorna uma tabela com nome completo, salário mensal e salário anual estimado (13 salários + 30% férias)
CREATE OR ALTER FUNCTION fn_salariAnual()
RETURNS @SalAno TABLE (
    nome_comp VARCHAR(100),
    salario DECIMAL(10,2),
    salario_anual DECIMAL(10,2)
)
AS
BEGIN
    INSERT INTO @SalAno
    SELECT
        CONCAT(F.Pnome, ' ', F.Minicial, ' ', F.Unome),
        F.Salario, 
        F.Salario * 13 + (F.Salario * 0.3)
    FROM FUNCIONARIO AS F;
    
    RETURN;
END;  
GO

-- Executando a função
SELECT * FROM dbo.fn_salariAnual();
```

---

### 🔹 Stored Procedures (Procedimentos Armazenados)

```sql
-- Procedure simples: Imprime uma mensagem no console
CREATE OR ALTER PROCEDURE sp_exibe_meu_nome
AS 
BEGIN
    PRINT 'Rafael Maruyama Dias';
END;
GO

EXEC sp_exibe_meu_nome;
GO

-- Procedure com Validação: Insere funcionário apenas se os dados já não existirem
CREATE OR ALTER PROCEDURE dbo.sp_verifica_nome(
    @Pnome VARCHAR(20),
    @Minicial CHAR(1),
    @Unome VARCHAR(20),
    @Cpf VARCHAR(11)
)
AS
BEGIN
    IF EXISTS (
        SELECT 1 
        FROM FUNCIONARIO AS F 
        WHERE F.Pnome = @Pnome 
          AND F.Unome = @Unome 
          AND F.Minicial = @Minicial 
          AND F.Cpf = @Cpf
    )
    BEGIN
        PRINT 'Já existe alguém com nome: ' + @Pnome + ' ' + @Minicial + ', ' + @Unome;
    END
    ELSE
    BEGIN
        INSERT INTO FUNCIONARIO (Pnome, Minicial, Unome, Cpf)
        VALUES (@Pnome, @Minicial, @Unome, @Cpf);
        
        PRINT 'Cadastro feito com sucesso';
    END;
END;
GO

-- Exemplo de execução da procedure de inserção/validação
EXEC dbo.sp_verifica_nome
    @Pnome = 'aaa',  
    @Minicial = 'b',
    @Unome = 'ccc',
    @Cpf = '12345678901';
GO

-- Procedure com parâmetro: Aplica aumento percentual a TODOS os funcionários
CREATE OR ALTER PROCEDURE sp_aumento_todos(@porcentagem DECIMAL(3,1))
AS
BEGIN
    UPDATE FUNCIONARIO
    SET Salario = Salario * (1 + (@porcentagem / 100));
END;
GO

EXEC dbo.sp_aumento_todos @porcentagem = 5;
SELECT * FROM FUNCIONARIO;

-- Procedure com múltiplos parâmetros: Aplica aumento a um funcionário específico (pelo CPF)
CREATE OR ALTER PROCEDURE sp_aumento_individual(
    @porcentagem DECIMAL(3,1),
    @cpf CHAR(11)
)
AS
BEGIN
    UPDATE FUNCIONARIO
    SET Salario = Salario * (1 + (@porcentagem / 100))
    WHERE Cpf = @cpf;
END;
GO

EXEC dbo.sp_aumento_individual @porcentagem = 50, @cpf = '98765432300';
SELECT * FROM FUNCIONARIO;

-- Procedure Criptografada: Impede a visualização do código-fonte da procedure
CREATE OR ALTER PROCEDURE sp_funcionarios
WITH ENCRYPTION
AS
    SELECT * FROM FUNCIONARIO;
GO

EXEC sp_help sp_funcionarios;

-- Procedure com Validação: Insere departamento e localização apenas se o código/nome não existirem
CREATE OR ALTER PROCEDURE sp_novo_dplc(
    @departamento VARCHAR(50),
    @localidade VARCHAR(50),
    @numero INT
)
AS
BEGIN
    IF EXISTS(SELECT 1 FROM DEPARTAMENTO WHERE Dnome = @departamento AND Dnumero = @numero)
    BEGIN
        PRINT 'Esse departamento já existe';
        RETURN;
    END
    ELSE
    BEGIN
        INSERT INTO DEPARTAMENTO(Dnome, Dnumero)
        VALUES (@departamento, @numero);

        INSERT INTO LOCALIZACAO_DEP(Dlocal, Dnumero)
        VALUES (@localidade, @numero);

        PRINT @departamento + ' inserido com sucesso';
        PRINT @localidade + ' inserido com sucesso';
    END;
END;
GO

EXEC dbo.sp_novo_dplc @departamento = 'TI_Dev', @localidade = 'São Paulo', @numero = 110;

-- Consulta para verificar os departamentos e suas respectivas localizações
SELECT *
FROM DEPARTAMENTO AS D
JOIN LOCALIZACAO_DEP AS L ON D.Dnumero = L.Dnumero;
```

---

## 5. Conceitos Teóricos Relacionais

* **Modelo de Entidade Relacionamento Conceitual (MER):** Utilizado para mapear as regras de negócio de forma abstrata.
* **Mapeamento de Dados:** Abordagem crucial para evitar duplicidade de dados e facilitar a construção de relacionamentos robustos.
* **Convenções de Código:** Utilização prática de padrões visuais (`snake_case` e `CamelCase`) para manter a legibilidade estrutural dos scripts de banco de dados.
* **Ferramenta de Apoio:** Uso do software *brModelo* para a criação e modelagem de diagramas conceituais de banco de dados.
