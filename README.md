# TODO-IPY-U2


# Paso 1, Creacion Web Service

En primer lugar, debemos determinar los web services que haremos, los que se muestran principalmente en el diagrama de despliegue, los pasos a continuacion, son los que seguimos en clases con la profe, para que nos sirva de referencia de lo que debemos ir construyendo. 

#API JAX-WS
Tecnologia Java para construir Clientes y Servicios Web, basados en XML y SOAP. 
Esta api oculta la complejidad de SOAP, resolviendo problemas asociados a creacion manual de WSDL. 

Cliente         <---    SOAP      --->      Web Service
(JAX-WS Runtime)      (Mensaje)             (JAX-WS Runtime)

Requisitos: 
- Netbeans
- OJDBC7 [Permite llegar a base de datos a traves de cualquier app Java, es un conector.]
- Glassfish

#Agregar glassfish: 
Abrir pestana services en netbeans, ir a servers y agregar server, escoger glassfish server, buscar el archivo descomprimido bajado de drive, local domain, siguiente. 
  
#Crear WebService
crear nuevo proyecto en netbeans, ir a java web y escoger web application, poner nombre a web service, en caso ejemplo "servicioGesBod", finish. Se creara un archivo html
   
Para echarlo a andar, se selecciona build y luego run. 
   
#Seleccionar serviciogesbod, seleccionar carpeta web service y seleccionar tipo archivo web service, lo que creara el esqueleto para un servicio soap, ponerle nombre al          servicio, ejemplo "WSgesbod", luego definir package, ejemplo, servicios, eso agregara directorio web service, codigo se creara en source packages: 
   

```java
   import javax.jws.WebService;
   import javax.jws.WebMethod;
   import javax.jws.WebParam;
   
   
   
   @WebService(serviceName = "ServicioGesbod")
   public class WSgesbod {
   
       //Retorna todos los productos
       @WebMethod(operationName ="consultarProductos")
       @WebResult(name="Producto")
       public List<Producto> get_todoProducto(){

           ProductoDao prodDao = new ProductoDao(); 
           List<Producto> lista = prodDao.fun_todoProducto(); 
           return lista; 
       }

       //Retorna un producto
       @WebMethod(operationName="getProducto")
       @WebResult(name="Producto")
       public Producto getProducto(@WebParam(name="codigoProducto") int codigo_producto){

           ProductoDao prodDao = new ProductoDao(); 
           Producto prod = prodDao.fun_producto(codigo_producto); 

           return prod; 

       }

       //actualiza stock
       @WebMethod(operationName="actualizarStock")
       @WebResult(name="resultado")   
       public boolean updateStock( @WebParam(name="codigo_produicto") int codigo_producto, @WebParam(name="cantidadProducto") int stock_producto )
       {
           ProductoDao prodDao = new ProductoDao();
           boolean resultado = prodDao.fun_actualizarStock(codigo_producto, stock_producto); 
           return resultado; 

       }
    }
```

# paso 2, Conexion a base de datos. 

1. Debemos sacar el script de nuestra base de datos, ademas de realizar algunos insert de informacion para las tablas que trabajaremos en primera instancia y que necesiten tener informacion, que sera entregada a los web services. 


Ejemplo clases: 


```sql
CREATE USER bodega IDENTIFIED BY bodega
      DEFAULT TABLESPACE users
      TEMPORARY TABLESPACE temp
      QUOTA UNLIMITED ON users;
      
GRANT CREATE session TO bodega;
GRANT CREATE table TO bodega;
GRANT CREATE view TO bodega;
GRANT CREATE procedure TO bodega;
GRANT CREATE synonym TO bodega;      

/

CREATE TABLE PRODUCTO 
(
  ID_PRODUCTO NUMBER NOT NULL 
, NOMBRE VARCHAR2(300) 
, PRECIO NUMBER 
, STOCK NUMBER 
, AUTOR VARCHAR2(1000) 
, VIGENCIA VARCHAR2(2) 
, CONSTRAINT PRODUCTO_PK PRIMARY KEY 
  (
    ID_PRODUCTO 
  )
  ENABLE 
);


INSERT INTO "PRODUCTO" (ID_PRODUCTO, NOMBRE, PRECIO, STOCK, AUTOR, VIGENCIA) VALUES ('1', 'La caperucita', '2000', '2000', 'Disney', 'S');
INSERT INTO "PRODUCTO" (ID_PRODUCTO, NOMBRE, PRECIO, STOCK, AUTOR, VIGENCIA) VALUES ('2', 'Tierra de oso', '7800', '4500', 'Disney', 'S');
```


2. Crearemos package donde se creara la conexion a la base de datos, nombre Modelo, escoger modelo y crear una nueva clase, con nombre Conexion. 
  - Script entregado por profesora en clases: 
```java
/*
 * To change this license header, choose License Headers in Project Properties.
 * To change this template file, choose Tools | Templates
 * and open the template in the editor.
 */
package Modelo;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

/**
 *
 * @author 
 */
public class Conexion {
    private Connection cnn;

    public Conexion() {
        this.conectar();
    }

    public Connection getCnn() {
        return cnn;
    }

    public void setCnn(Connection cnn) {
        this.cnn = cnn;
    }
    
    private void conectar() {
        try{
        Class.forName("oracle.jdbc.OracleDriver");
        String BaseDeDatos = "jdbc:oracle:thin:@localhost:1521:xe";
        cnn= DriverManager.getConnection(BaseDeDatos,"usuario","clave");
            if(cnn==null)
            {
                System.out.println("Conexion fallida");
            }
        }
        catch(ClassNotFoundException | SQLException e)
        {System.out.println("error"+e);}
        
    }
    
    public void desonectarBD() {
        try {
            this.getCnn().close();
        } catch (SQLException ee) {
            System.out.println(ee.getMessage());
        }

    }

    
}
```
Este codigo sirve para conectarse a paginas web o escritorio hacia una base de datos. 

3. Importar el OJDBC que es como llegaremos a la base de datos, en libraries, add jar/folder, buscar conector subido a drive, para netbeans 8.2 el mas estable es el 7, 
se recomienda dejar el ojdbc en la carpeta donde se crea proyecto y escoger path relativo, luego se agrega. 

4. Conectarse a la base de datos desde sql developer y ejecutamos el script de nuestra base de datos. 

# Creacion de modelos

1. en packages, modelo se crearan las clases que utilizaremos, que seran coincidentes con los nombre de las tablas de la base de datos, estas se irian creando a medida que se vayan necesitando en el desarrollo. 

Ejemplo de clase Producto en clases: 
```java
package Modelo;

public class Producto {
    
    private int codigo_producto; 
    private String nombre_producto; 
    private int precio; 
    private int stock; 
    private String autor; 
    private String vigencia; 

    public Producto() {
    }
    
   
    public Producto(int codigo_producto, String nombre_producto, int precio, int stock, String autor, String vigencia) {
        this.codigo_producto = codigo_producto;
        this.nombre_producto = nombre_producto;
        this.precio = precio;
        this.stock = stock;
        this.autor = autor;
        this.vigencia = vigencia;
    }

    public int getCodigo_producto() {
        return codigo_producto;
    }

    public void setCodigo_producto(int codigo_producto) {
        this.codigo_producto = codigo_producto;
    }

    public String getNombre_producto() {
        return nombre_producto;
    }

    public void setNombre_producto(String nombre_producto) {
        this.nombre_producto = nombre_producto;
    }

    public int getPrecio() {
        return precio;
    }

    public void setPrecio(int precio) {
        this.precio = precio;
    }

    public int getStock() {
        return stock;
    }

    public void setStock(int stock) {
        this.stock = stock;
    }

    public String getAutor() {
        return autor;
    }

    public void setAutor(String autor) {
        this.autor = autor;
    }

    public String getVigencia() {
        return vigencia;
    }

    public void setVigencia(String vigencia) {
        this.vigencia = vigencia;
    }

    @Override
    public String toString() {
        return "Producto{" + "codigo_producto=" + codigo_producto + ", nombre_producto=" + nombre_producto + ", precio=" + precio + ", stock=" + stock + ", autor=" + autor + ", vigencia=" + vigencia + '}';
    }
     
}
```

# Creacion de procedimientos almacenados 
1. Se crearan los procedimientos almacenados en la base de datos, para luego llamarlos desde java 

Tip para crear procedimiento almacenado: 
- Escoger tabla desde sql developer, click derecho e ir a tabla, luego escoger generar API de tabla, lo que creara un package con las acciones insertar, actualizar y eliminar, el leer los datos debe ser creado manual. 

Ejemplo Clases: 

```sql

create or replace package PRODUCTO_PKG
is
PROCEDURE proc_mostrarProducto(p_cursor out SYS_REFCURSOR);
PROCEDURE proc_prodDetalle(p_id number, p_cursor out SYS_REFCURSOR); 

procedure upd_descuentaStock (
    p_ID_PRODUCTO in PRODUCTO.ID_PRODUCTO%type
    ,p_STOCK in PRODUCTO.STOCK%type default null 
    ); 
    


create or replace package body PRODUCTO_PKG
is

PROCEDURE proc_mostrarProducto(p_cursor out SYS_REFCURSOR)
is
begin
    open p_cursor for 
        select * from producto;
end;

PROCEDURE proc_prodDetalle(p_id number, p_cursor out SYS_REFCURSOR)
is
begin
    open p_cursor for 
        select * from producto
        where id_producto = p_id; 
end;

procedure upd_descuentaStock (
    p_ID_PRODUCTO in PRODUCTO.ID_PRODUCTO%type
    ,p_STOCK in PRODUCTO.STOCK%type default null 
    ) is
begin
    update PRODUCTO set
    STOCK = STOCK - p_STOCK
    where ID_PRODUCTO = p_ID_PRODUCTO;
end;
```



# Creacion de clase intermedia que permitira hacer las acciones hacia la base de datos y desde la base de datos, en caso dejemplo, sera la clase DAO. 




