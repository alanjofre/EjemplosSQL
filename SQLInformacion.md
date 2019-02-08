# __Investigación/Practica SQL__ 

* ~~Tipos de datos en SQL~~
* ~~variable tipo tabla~~
* ~~columna~~
* ~~vistas~~
* vistas indexadas 
* ~~merge~~ 
* ~~StoreProcedures~~ 
* ~~Partition By~~
* indices
* ~~planes de ejecucion~~
* ~~Seguridad~~ sin encriptar 
* SQL Profiles 
* Documentacion Documentacion Microsoft. 

-----------------------------------------------------------------------------------------
## __Tipo de Datos en SQL__ 

#### __Creación de tipo de datos:__ 
    CREATE TYPE nombretipodedato FROM tipodedato(x) NOT NULL 
#### __Listar tipos de datos creados por el usuario:__  
    SELECT * 
    FROM   sys.types 
    WHERE  is_user_defined = 1 
#### __Ver si existe el tipo de dato:__  
    SELECT * 
    FROM   sys.types 
    WHERE  NAME = 'NombreTipoDato' 
#### __Crear Tabla con tipo de dato definido por el usuario:__ 
    CREATE nombretabla (columnaa nombretipodedato)go
#### __Eliminar tipo de dato:__ 
    DROP type nombretipodedato 
_(Para poder eliminarlo ninguna tabla tiene que usar el tipo de dato a elimminar.)_ 

## __Variales Tipo Tabla__

### __Características:__
* Las Variables Tipo Tabla son tipos de datos que generalmente son utilizados en un lote T-SQL,  procedimiento almacenado o función definida por el usuario.
* Las variables tipo tabla se crea y definen igual a las tablas con la diferencia que tienen una alcance de vida definido.
* Use las variables tipo tabla con conjunto de datos pequeños.

#### _Beneficios:_ 
* Viven únicamente durante la ejecución del lote, función o procedimiento almacenado.
* Tiempo de bloqueos mas cortos.
* Cuando se usan en procedimientos almacenados realizan menos re compilaciones.

#### _Consideración:_ 
* Para que el rendimiento de la variable tipo Tabla sea optimo, el resultado no debe ser demasiado grande. 

#### _Sintaxis_
    Declare @NombreVariableTipoTabla Table (Campo1 TipoDatos, Campo2 TipoDatos)

Ejemplo:

    DECLARE @NombreVariableTipoTabla1 TABLE 
    ( 
     codigo      NCHAR(3) PRIMARY KEY, 
     descripcion NVARCHAR(50) 
    ) 

    INSERT INTO @NombreVariableTipoTabla1 
    SELECT productid, 
        productname 
    FROM   products 
    WHERE  categoryid = 1 

    SELECT * 
    FROM  @NombreVariableTipoTabla1 

    go 

## __Vistas__
Una vista es una tabla virtual cuyo contenido está definido por una consulta, ésta puede provenir de una o de varias tablas, o bien de otras vistas de la base de datos actual u otras bases de datos. 

#### __Creación de vistas__
    USE BaseDeDatos; 

    go 

    CREATE VIEW v_pruebavista 
    AS 
    SELECT a.columna1, 
            b.columna2, 
            b.columna3 
    FROM   tablaa a 
            JOIN tablab b 
            ON a.columna2 = b.columna1 

    go 

    -- Query the view    
    SELECT columna1, 
        columna2 
    FROM   v_pruebavista 
    ORDER  BY columna1 


## __Merge__

#### __Ejemplo__
Estructura: Codigo INT PRIMARY KEY,
Nombre VARCHAR(100),
Puntos INT

(1,'Juan Perez',10),
(2,'Marco Salgado',5),
(3,'Carlos Soto',9),
(4,'Alberto Ruiz',12),
(5,'Alejandro Castro',5) -Usuarios

(1,'Juan Perez',12),
(2,'Marco Salgado',11),
(4,'Alberto Ruiz Castro',4),
(5,'Alejandro Castro',5),
(6,'Pablo Ramos',8)-UsuarioActual

    --Sincronizar la tabla TARGET con 
    --los datos actuales de la tabla SOURCE 
    MERGE usuarios AS TARGET 
    using usuariosactual AS SOURCE 
    ON ( TARGET.codigo = source.codigo ) 

    --Cuandos los registros concuerdan con por la llave 
    --se actualizan los registros si tienen alguna variación 
    WHEN matched AND TARGET.nombre <> SOURCE.nombre OR TARGET.puntos <> 
    SOURCE.puntos THEN 
    UPDATE SET TARGET.nombre = SOURCE.nombre, 
                TARGET.puntos = SOURCE.puntos 

    --Cuando los registros no concuerdan por la llave 
    --indica que es un dato nuevo, se inserta el registro 
    --en la tabla TARGET proveniente de la tabla SOURCE 
    WHEN NOT matched BY target THEN 
    INSERT (codigo, 
            nombre, 
            puntos) 
    VALUES (SOURCE.codigo, 
            SOURCE.nombre, 
            SOURCE.puntos) 

    --Cuando el registro existe en TARGET y no existe en SOURCE 
    --se borra el registro en TARGET 
    WHEN NOT matched BY source THEN 
    DELETE 

## __Funciones__

 ### __Funciones agregadas SQL, devuelven un sólo valor, calculado con los valores de una columna.__

* _AVG() - La media de los valores_
* _COUNT() - El número de filas_
* _MAX() - El valor más grande_
* _MIN() - El valor más pequeño_
* _SUM() - La suma de los valores_
* _GROUP BY - Es una sentencia que va muy ligada a las funciones agregadas_

### __Funciones escalares SQL, devuelven un sólo valor basándose en el valor de entrada.__
* __Funciones con valores de tabla y múltiples instrucciones:__ 
* _UCASE() - Convierte un campo a mayúsculas_
* _LCASE() - Convierte un campo a minúsculas_
* _MID() - Extrae caracteres de un campo de texto_
* _LEN() - Devuelve la longitud de un campo de texto_
* _NOW() - Devuelve la hora y fecha actuales del sistema_
* _FORMAT() - Da formato a un formato para mostrarlo_

#### __Funciones definidas por el usuario.__

* __Tipos de funciones:__
    * __Funciones escalares__: _Las funciones escalares son aquellas que reciben parámetros de entrada para ser procesados y al final retornar en un valor con un tipo de dato sencillo._

    * __Funciones con valores de tabla en línea:__ _Este tipo de función tiene la misma sintaxis que una función escalar, la única diferencia es que devuelve tipo de dato TABLE (una tabla compuesta de registros)._

#### __Ejemplos__

* __Función escalar__:

        CREATE FUNCTION [DBO].[Nombre_Funcion] 
        (
            @ID AS VARCHAR(7),
            @nombre AS VARCHAR (70),
            @tipo AS CHAR (2)
         )
        RETURNS  VARCHAR(6)
        AS
        BEGIN
        --Aquí puede ir cualquier código de SQL Server
            DECLARE @Valor_Retorno AS VARCHAR(6)
            DECLARE @Valor_Intermedio AS VARCHAR(6)

            @Valor_Intermedio = 
                SELECT Valor FROM DBO.tbTabla  
                WHERE strCod = @ID AND strNombre = @nombre 
                    AND strTipo = @tipo
            
            SET @Valor_Retorno =  @Valor_Intermedio
            
            RETURN @Valor_Retorno
        END
        
        GO

* __Función con valores de tabla:__

        CREATE FUNCTION Salidas_libro(@idlibro VARCHAR(12)) 
        returns TABLE 
        AS 
            RETURN 
            (SELECT Invb.nombre_libro, 
                    Usu.nombre_usuario, 
                    MovB.fecha_salida, 
                    mobb.fecha_devolucion 
            FROM   tbinventario_biblioteca AS Invb 
                    JOIN tbmovimientos_biblioteca AS Movb 
                        ON Invb.idcliente = MovB.idcliente 
                    JOIN tbusuarios AS Usu 
                        ON Usu.idusuario = MovB.idusuario 
            WHERE  Invb.idlibro = @idLibro) 
    * Ejecución: SELECT Count(Invb.Nombre_Libro) FROM Salidas_libro ('ISBN00000000')

## Stored Procedures

#### __Creación__
        USE BaseDeDatos; 
        go 
        CREATE PROCEDURE NombreStoredProcedure 
            @ParametroA NVARCHAR(50), @ParametroB NVARCHAR(50) 
        AS 
            SELECT Campo1, 
                Campo2, 
                Campo3 
            FROM   NombreTabla_Vista 
            WHERE  Campo1 = @ParametroA 
                AND Campo2 = @ParametroB               
        go
#### __Ejecución__  
        Execute NombreStoredProcedure ParametroA,ParametroB;

#### __Ejecución Automática__ 
El siguiente ejemplo establece un procedimiento para la ejecución automática.

        EXEC sp_procoption @ProcName = '<procedure name>'   
            , @OptionName =  'startup'   
            , @OptionValue = 'on';   
    
El siguiente ejemplo detiene la ejecución automática de un procedimiento.

        EXEC sp_procoption @ProcName = '<procedure name>'   
            , @OptionValue = 'off';  


## __Partition by__

https://www.youtube.com/watch?v=7yp_QYBCEP4
  
        SELECT id, 
            valor1, 
            Sum(valor1) 
                OVER( 
                partition BY id) AS SumaPartitionBy 
        FROM   tabla1 
        ORDER  BY id, 
                valor1 

## __Planes de Ejecución__

https://www.sqlshack.com/sql-server-query-execution-plans-basics/


## __Seguridad__

### __Autenticación en SQL Server__
Describe los inicios de sesión y autenticación en SQL Server y proporciona vínculos a recursos adicionales.
* Existen dos tipos de autenticación: 
    * Autenticación con usuario de Windows.
    * Autenticación Mixta.
    
### __Roles de servidor y base de datos en SQL Server__

(https://docs.microsoft.com/es-es/sql/relational-databases/security/authentication-access/server-level-roles?view=sql-server-2017)

Describe funciones fijas de bases de datos y servidores, funciones de base de datos personalizadas y cuentas integradas, y proporciona vínculos a recursos adicionales.
* __Tipos de Roles:__
    * __Roles Fijos en Base de Datos:__ 

        __Roles__     | __Descripción__
        :------------ | :---------------------------------------------------------------------------------------------------
        Sysadmin      | Realiza cualquier actividad en el servidor. 
        ServerAdmin   | Puede realizar configuraciones en el servidor y apagarlo.
        SecurityAdmin | Administra los inicios de sesión y permisos a nivel de servidor y base de datos (GRANT,DENY,REVOKE).
        ProcessAdmin  | Puede detener procesos que se ejecutan en una instancia de SQL. 
        SetupAdmin    | Pueden agregar y quitar servidores vinculados mediante instrucciones Transact-SQL.
        BulkAdmin     | Pueden ejecutar instrucciones BULK INSERT.
        DiskAdmin     | Se usa para administrar archivos de disco. 
        DbCrator      | Puede crear, modificar, quitar y resturar caulquier base de datos. 
        Public        | Cada inicio de sesión pertence al rol public hasta que se le conceda o deniegue algún permiso. 


    * __Roles y usuarios de base de datos__

         __Roles__     | __Descripción__
        :------------- | :-----------------------------------------------------------------------------------------------------
        Public         | Esta contenido en todas las bases de datos. No se puede eliminar ni agregar usaurios de ella. 
        DBO            | Tiene permisos implicitos para realizar todas las tareas en la base de datos. 
        Guest          | Esta cuenta en la base de datos evita que los usuarios que se conecten a una instancia de SQL y tenga acceso a todas las bases de datos de uns servidor. 


### __Propiedad y separación de esquemas de usuario en SQL Server__
Describe la propiedad de los objetos y separación entre usuario y esquema, y proporciona vínculos a recursos adicionales.
Trabajar con esquemas permite disponer de más flexibilidad en la administración de los permisos de objeto de base de datos. Un esquema es un contenedor con nombre para objetos de base de datos, que permite agrupar objetos en espacios de nombres independientes.

+ Ejemplo: _Creacion de esquema, tabla y asignacion de permisos._   

        CREATE SCHEMA Esquema1;
        GO
        CREATE TABLE Esquema1.TablaA (ChainID int, width dec(10,2));
        GRANT SELECT ON SCHEMA::Esquema1 TO Usuario1;  
        DENY SELECT ON SCHEMA::Esquema1 TO Usuario2; 

### __Autorización y permisos en SQL Server__
Describe la concesión de permisos a través del principio de los privilegios mínimos y proporciona vínculos a recursos adicionales.

__Tipo de Permiso__ | __Descripción__
:------------------ | :--------------
GRANT               | Concede un permiso. 
REVOKE              | Saca un permiso, es el estado por defecto de cualquier objeto nuevo.
DENY                | Revoca el permiso y no puede ser heredado. Tiene prioridad sobre todos los permisos. No aplica en el propietario del objeto o a los miembros del schema SysAdmin. 

### __Cifrado de datos en SQL Server__
Describe las opciones de cifrado de datos en SQL Server y proporciona vínculos a recursos adicionales.

### __Seguridad de integración de CLR en SQL Server__
Proporciona vínculos a recursos de seguridad de la integración CLR.

## __Indices__

_Si se crean índices, las consultast de la base de datos va a ese índice primero y luego recupera los correspondientes registros de la tabla directamente._

__Tipos de Indices:__ 
* Agrupados (Clustered)
    * Un índice agrupado define el orden en el cual los datos son __físicamente almacenados__ en una tabla. Los datos de las tablas pueden ser ordenados sólo en una forma, por lo tanto, sólo puede haber un índice agrupado por tabla. Cuando se crea una nueva tabla y se define una columna como PrimaryKey, se crea automaticamente un indice clustered teniendo en cuenta el campo PrimaryKey.
    Ejemplo: 

            USE basededatos 
            CREATE CLUSTERED INDEX nombre_indice_agrupado 
            ON tablaa(campo1 ASC, campo3 DESC) 

* No agrupados (Nonclustered)
    * Un índice no agrupado no ordena los datos físicos dentro de la tabla. De hecho, un índice no agrupado es agrupado en un solo lugar y los datos de la tabla son almacenados en otro lugar. Esto permite tener más de un índice no agrupado por tabla. Cuando una consulta es lanzada contra una columna en la cual el índice es creado, la base de datos primero irá al índice y buscará la dirección de la fila correspondiente en la tabla. Luego, irá a esa dirección de fila y obtendrá otros valores de columna. Es debido a este paso adicional que los índices no agrupados son más lentos que los índices agrupados.
    Ejemplo:

            USE basededatos 
            CREATE NONCLUSTERED INDEX nombre_indice_no_agrupado  ON student(campo ASC) 

Conclución: 
* Puede haber sólo un índice agrupado por tabla. De todos modos, puede crear múltiples índices no agrupados en una sola tabla.
* Los índices agrupados sólo ordenan tablas. Por lo tanto, no consumen almacenaje extra. 
* Los índices no agrupados son almacenados en un lugar separado de la tabla real. Reclamando más espacio de almacenamiento.
* Los índices agrupados son más rápidos que los índices no agrupados, ya que no involucran ningún paso extra de búsqueda.