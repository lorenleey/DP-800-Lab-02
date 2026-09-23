# DP-800-Lab-02 - Implement Programmability Objects with SQ
Este laboratorio forma parte de la preparación para la certificación **Microsoft DP-800: Developing AI-Enabled Database Solutions**.


# Requisitos
  - SQL Server 2019 o superior
- SQL Server Management Studio (SSMS)
- Transact-SQL (T-SQL)
- AdventureWorksLT

 Repositorio oficial de los laboratorios:

`MicrosoftLearning/mslearn-sql-developer`

# Desarrollo del laboratorio

## 1. Verificación de AdventureWorksLT

Primero se verificó que la base de datos **AdventureWorksLT** estuviera correctamente instalada y accesible.

``` sql
 -- Verificar clientes
SELECT TOP (5)
    CustomerID,
    FirstName,
    LastName
FROM SalesLT.Customer;
GO

-- Verificar pedidos
SELECT TOP (5)
    SalesOrderID,
    OrderDate,
    CustomerID
FROM SalesLT.SalesOrderHeader;
GO

-- Verificar productos
SELECT TOP (5)
    ProductID,
    Name,
    ListPrice
FROM SalesLT.Product;
GO
````




