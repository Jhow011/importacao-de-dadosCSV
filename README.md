
## 📄 Documentação do Projeto SQL: Importação Eficiente de Dados CSV no MySQL

### 1. Visão Geral do Projeto

**Nome do Projeto:** `mysql-load-data-infile-eficiente`
**Objetivo:** Demonstrar o uso do comando `LOAD DATA INFILE` como a ferramenta mais rápida e eficaz para transferir listas extensas de dados de arquivos CSV para tabelas MySQL.

**Conceitos Fundamentais:**

*   **CSV (Comma-Separated Values):** Formato de arquivo popular, simples e estruturado, onde os dados tabulares são organizados em linhas (registros) e colunas (campos). A separação dos valores é feita por um delimitador, que pode ser vírgula (`,`), ponto e vírgula (`;`) ou tabulação.
*   **Eficiência:** O `LOAD DATA INFILE` atua como um atalho poderoso, economizando tempo em comparação com inserções linha por linha (INSERT).

### 2. Sintaxe e Cláusulas Chave do `LOAD DATA INFILE`

O comando `LOAD DATA INFILE` é essencial para preencher as tabelas do MySQL com informações em CSV.

**Sintaxe Básica:**

```sql
LOAD DATA INFILE 'caminho/para/arquivo.csv'
INTO TABLE nome_da_tabela
FIELDS TERMINATED BY 'delimitador'
ENCLOSED BY 'caractere_delimitador'
LINES TERMINATED BY 'terminador_de_linha';
```

**Cláusulas Essenciais:**

| Cláusula | Função | Exemplo |
| :--- | :--- | :--- |
| **`LOAD DATA INFILE`** | Indica o caminho completo onde o MySQL deve encontrar o arquivo CSV. | `"C:/.../Categories.csv"` |
| **`INTO TABLE`** | Especifica a tabela de destino dos dados. | `INTO TABLE Categories` |
| **`FIELDS TERMINATED BY`** | Define o caractere usado para separar as colunas. Se o arquivo usar ponto e vírgula (`;`), a cláusula deve ser ajustada. | `FIELDS TERMINATED BY ';'` |
| **`LINES TERMINATED BY`** | Define o caractere que indica o fim de cada linha (o padrão é `\n`). Em Windows, pode ser necessário usar `\r\n`. | `LINES TERMINATED BY '\n'` |
| **`IGNORE 1 ROWS`** | Crucial para ignorar a primeira linha do arquivo CSV, que geralmente contém o cabeçalho (nomes das colunas), evitando que seja inserida como registro de dados. | `IGNORE 1 ROWS` |

**Requisito de Acessibilidade:** A localização do arquivo CSV deve ser acessível pelo MySQL. Os exemplos do projeto utilizam o caminho comum de upload do MySQL: `"C:/ProgramData/MySQL/MySQL Server 8.0/Uploads/"`.

### 3. Estrutura do Banco de Dados (`ERP_DB`)

O projeto demonstra a importação de dados para um esquema de banco de dados ERP, que é criado com o comando `create database ERP_DB;`.

As tabelas e as respectivas chaves primárias (`PRIMARY KEY`) são:

| Tabela | Chave Primária | Importações com `LOAD DATA INFILE` |
| :--- | :--- | :--- |
| `Categories` | `CategoryID` | Sim |
| `Suppliers` | `SupplierID` | Sim |
| `Customers` | `CustomerID` (VARCHAR) | Sim |
| `Employees` | `EmployeeID` | Sim |
| `Shippers` | `ShipperID` | Sim |
| `Products` | `ProductID` | Sim (com transformação de dados) |
| `Orders` | `OrderID` | Sim (com transformação de data) |
| `OrderDetails` | `OrderDetailID` | Sim |

O projeto também inclui a adição de chaves estrangeiras (`FOREIGN KEY`) após a criação das tabelas, como `fkpro` e `fkpri` na tabela `Products`, garantindo a integridade referencial.

### 4. Demonstração de Transformação de Dados em Tempo de Carga

Uma vantagem crucial do `LOAD DATA INFILE` é a capacidade de manipular os dados *durante* a importação, utilizando variáveis (`@nome_coluna`) e a cláusula `SET`.

#### A. Tratamento de Formato Numérico (Preço)

Se o preço no arquivo CSV estiver no formato brasileiro (usando vírgula como separador decimal), é necessário convertê-lo para o formato esperado pelo MySQL (ponto).

**Exemplo (Tabela `Products`)**:
1.  O valor é capturado em uma variável temporária: `@price`.
2.  A função `REPLACE` é usada na cláusula `SET` para trocar a vírgula (`,`) por ponto (`.`) antes de inserir no campo `Price`.

```sql
LOAD DATA INFILE "caminho/products.csv"
...
(ProductID,ProductName,SupplierID,CategoryID,Unit,@price ) -- Captura a string de preço
set price = replace (@price,",","."); -- Transforma a vírgula em ponto
```

#### B. Conversão de Formato de Data

Se as datas no CSV estiverem em um formato de *string* (ex: DD/MM/AAAA), é necessário convertê-las para o tipo `DATE` do MySQL.

**Exemplo (Tabela `Employees` ou `Orders`)**:
1.  A data original é capturada em uma variável temporária: `@BirthDate` ou `@OrderDate`.
2.  A função `STR_TO_DATE` é usada para converter a string, especificando o formato de entrada (`%d/%m/%Y` = Dia/Mês/Ano).

```sql
LOAD DATA INFILE "caminho/Employees.csv"
...
(EmployeeID,LastName,FirstName,@BirthDate,Photo,Notes) -- Captura a data como string
set BirthDate = str_to_date(@BirthDate,"%d/%m/%Y"); -- Converte a string para DATE
```

### 5. Configurações de Delimitador

O projeto enfatiza a necessidade de ajustar o delimitador se ele não for a vírgula padrão.

**Exemplo B: Uso de Ponto e Vírgula (`;`)**
Se o arquivo `Customers.csv` usar ponto e vírgula como delimitador:

```sql
LOAD DATA INFILE 'caminho/para/Customers.csv'
INTO TABLE Customers
FIELDS TERMINATED BY ';' -- Ajuste para PONTO E VÍRGULA
ENCLOSED BY ''
LINES TERMINATED BY '\n'
IGNORE 1 ROWS;
```

A estrutura da tabela deve corresponder à ordem e aos tipos de dados das colunas no arquivo CSV para que a importação seja bem-sucedida.
