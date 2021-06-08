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
