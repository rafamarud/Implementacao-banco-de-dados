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
GRANT SELECT, INSERT ON [dbo].[FORNECEDOR] TO Juca
