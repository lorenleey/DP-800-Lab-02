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

El resultado que aparece al ejecutar ese código son tres tablas
![Verificación de AdventureWorksLTS2025](img/verificacion-adventure-works.png) 

## 2. Creación de una View

Se creó la vista `SalesLT.vCustomerOrders` para combinar información de clientes y pedidos.

```sql
CREATE OR ALTER VIEW SalesLT.vCustomerOrders AS
SELECT 
    c.CustomerID,
    CONCAT(c.FirstName, ' ', c.LastName) AS CustomerName,
    h.SalesOrderID,
    h.OrderDate
FROM SalesLT.Customer c
INNER JOIN SalesLT.SalesOrderHeader h 
    ON c.CustomerID = h.CustomerID;
```

Se muestran los cinco pedidos más recientes utilizando la
vista recién creada.

```sql
SELECT TOP (5) * 
FROM SalesLT.vCustomerOrders 
ORDER BY OrderDate DESC;
```

El resultado al consultar la vista `SalesLT.vCustomerOrders`
![Vista verificación](img/verificacion-vistas.png) 

## 3. Creación de un Stored Procedure

Se creó el procedimiento almacenado: `dbo.AddOrderLineItem`

Su objetivo es insertar una nueva línea de producto en un pedido existente y actualizar posteriormente el subtotal del pedido.

``` sql
CREATE OR ALTER PROCEDURE dbo.AddOrderLineItem
    @SalesOrderID INT,
    @ProductID    INT,
    @Quantity     INT
AS
BEGIN
    SET NOCOUNT ON;

    BEGIN TRANSACTION;

    -- Obtener el precio de lista del producto
    DECLARE @UnitPrice DECIMAL(18,2);

    SELECT @UnitPrice = CAST(ListPrice AS DECIMAL(18,2))
    FROM SalesLT.Product
    WHERE ProductID = @ProductID;

    -- Comprobar que el producto existe
    IF @UnitPrice IS NULL
    BEGIN
        ROLLBACK TRANSACTION;
        THROW 50010, 'El ProductID especificado no es válido.', 1;
    END;

    -- Comprobar que el pedido existe
    IF NOT EXISTS
    (
        SELECT 1
        FROM SalesLT.SalesOrderHeader
        WHERE SalesOrderID = @SalesOrderID
    )
    BEGIN
        ROLLBACK TRANSACTION;
        THROW 50011, 'El SalesOrderID especificado no es válido.', 1;
    END;

    -- Insertar una nueva línea de pedido sin descuento
    INSERT INTO SalesLT.SalesOrderDetail
    (
        SalesOrderID,
        OrderQty,
        ProductID,
        UnitPrice,
        UnitPriceDiscount
    )
    VALUES
    (
        @SalesOrderID,
        @Quantity,
        @ProductID,
        @UnitPrice,
        0
    );

    -- Recalcular el subtotal del pedido
    UPDATE h
    SET
        SubTotal = d.SumLineTotal,
        ModifiedDate = SYSUTCDATETIME()
    FROM SalesLT.SalesOrderHeader AS h
    INNER JOIN
    (
        SELECT
            SalesOrderID,
            SUM(LineTotal) AS SumLineTotal
        FROM SalesLT.SalesOrderDetail
        WHERE SalesOrderID = @SalesOrderID
        GROUP BY SalesOrderID
    ) AS d
        ON d.SalesOrderID = h.SalesOrderID;

    COMMIT TRANSACTION;
END;
GO
````

Probar el procedimiento almacenado

``` sql
DECLARE @SalesOrderID INT =
(
    SELECT TOP (1) SalesOrderID
    FROM SalesLT.SalesOrderHeader
    ORDER BY SalesOrderID DESC
);

EXEC dbo.AddOrderLineItem
    @SalesOrderID = @SalesOrderID,
    @ProductID = 680,
    @Quantity = 1;

-- Comprobar las líneas del pedido
SELECT TOP (5) *
FROM SalesLT.SalesOrderDetail
WHERE SalesOrderID = @SalesOrderID
ORDER BY SalesOrderDetailID DESC;

-- Comprobar el subtotal actualizado
SELECT
    SalesOrderID,
    SubTotal,
    TaxAmt,
    Freight,
    TotalDue
FROM SalesLT.SalesOrderHeader
WHERE SalesOrderID = @SalesOrderID;
GO
```
Resultado de la ejecución 

![Procedimiento almacenado](img/procedimiento-almacenado.png) 

## 4. Creación de una Scalar Function

Se creó la función: `dbo.fnOrderTotal`

Esta función recibe el identificador de un pedido y devuelve su importe total.

```sql
CREATE OR ALTER FUNCTION dbo.fnOrderTotal (@OrderID INT)
RETURNS DECIMAL(18,2)
AS
BEGIN
    DECLARE @Total DECIMAL(18,2);

    SELECT @Total = SUM(LineTotal)
    FROM SalesLT.SalesOrderDetail
    WHERE SalesOrderID = @OrderID;

    RETURN ISNULL(@Total, 0.00);
END;
```

  Probar la función escalar

```sql
SELECT 
    d.SalesOrderID,
    dbo.fnOrderTotal(d.SalesOrderID) AS OrderTotal
FROM SalesLT.SalesOrderDetail d
GROUP BY d.SalesOrderID
ORDER BY d.SalesOrderID DESC;
```

Una `Scalar Function` devuelve un único valor y puede reutilizarse dentro de otras consultas.
Resultado de la ejecución 

![Función Escalar ejecución](img/funcion-escalar.png) 


