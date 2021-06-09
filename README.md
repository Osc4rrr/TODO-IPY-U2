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

package servicios;

import Modelo.Producto;
import Modelo.ProductoDao;
import java.util.List;
import javax.jws.WebService;
import javax.jws.WebMethod;
import javax.jws.WebParam;
import javax.jws.WebResult;


@WebService(serviceName = "ServicioGesbod")
public class WSgesbod {
    /**
     * This is a sample web service operation
     */
    @WebMethod(operationName = "saludo")
    @WebResult(name="saludar")
    public String hello(
            @WebParam(name = "nombre") String txt, 
            @WebParam(name = "apellido_paterno") String apellido
    ) {
        return "Hola " + txt + " " + apellido;
    }
    
    @WebMethod(operationName = "sumar")
    @WebResult(name="resultado")
    public int Suma(
            @WebParam(name = "digito1") int numero1, 
            @WebParam(name = "digito2") int numero2
    ) {
        int sum = numero1+numero2; 
        return sum;
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

# Paso 3, Creacion de modelos

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

# Paso 4, Creacion de procedimientos almacenados 
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



# Paso 5, Creacion de clase intermedia DAO 

permitira hacer las acciones hacia la base de datos y desde la base de datos, en caso dejemplo, sera la clase DAO en package modelo, esto se deberia crear en otro package y no necesariamente en el mismo package modelo, asi tener una estructura mas ordenada.

Dao nos permite abstraernos del motor de base de datos, lo que quiere decir que si se cambia motor de base de datos, el patron dao deberia funcionar igual. 

Ejemplo ProductoDao Clases: 
```java

package Modelo;


import java.sql.*; 
import java.util.ArrayList;
import java.util.List;
import java.util.logging.Level;
import java.util.logging.Logger;
import oracle.jdbc.OracleTypes; 

public class ProductoDao {
    Conexion conn; 

    public ProductoDao() {
        conn = new Conexion(); 
    }
    
    public List<Producto> fun_todoProducto(){
    
        Connection acceso = conn.getCnn(); 
        Producto prod = null;
        List<Producto> lista = new ArrayList(); 
        
        try { 
            CallableStatement cs = acceso.prepareCall("{ call PRODUCTO_PKG.proc_mostrarProducto(?) }");
            cs.registerOutParameter(1, OracleTypes.CURSOR); 
            cs.execute(); 
            
            ResultSet rs = (ResultSet) cs.getObject(1); 
            
            while (rs.next()){
                prod = new Producto(); 
                prod.setCodigo_producto(rs.getInt("ID_PRODUCTO"));
                prod.setNombre_producto(rs.getString("NOMBRE"));
                prod.setPrecio(rs.getInt("PRECIO"));
                prod.setStock(rs.getInt("STOCK"));
                prod.setAutor(rs.getString("AUTOR"));
                prod.setVigencia(rs.getString("VIGENCIA"));
                
                lista.add(prod); 
            }
        } catch (SQLException ex) {
            //Logger.getLogger(ProductoDao.class.getName()).log(Level.SEVERE, null, ex);
            
            System.out.println("Error" + ex.getMessage());
        }
        
        return lista;

    }
    
    
    public Producto fun_producto(int codigo_producto){
        Connection acceso = conn.getCnn();
        Producto prod = new Producto(); 

        try {
            CallableStatement cs = acceso.prepareCall("{ call PRODUCTO_PKG.proc_prodDetalle(?, ?) }");
            cs.setInt(1, codigo_producto);
            cs.registerOutParameter(2, OracleTypes.CURSOR); 
            cs.execute(); 
            
            ResultSet rs = (ResultSet) cs.getObject(2);
            
            while(rs.next()){
                prod.setCodigo_producto(rs.getInt("ID_PRODUCTO"));
                prod.setNombre_producto(rs.getString("NOMBRE"));
                prod.setPrecio(rs.getInt("PRECIO"));
                prod.setStock(rs.getInt("STOCK"));
                prod.setAutor(rs.getString("AUTOR"));
                prod.setVigencia(rs.getString("VIGENCIA"));
            }


        } catch (Exception ex) {
            System.out.println("Error" + ex.getMessage());

        }
        
        return prod; 
    
    }
    
    
    public boolean fun_actualizarStock(int codigo_producto, int stock){
        Connection acceso = conn.getCnn();
        boolean resultado = false; 
        
        
        try {
            CallableStatement cs;
            cs = acceso.prepareCall("{ call PRODUCTO_PKG.upd_descuentaStock(?, ?) }");
            cs.setInt(1, codigo_producto);            
            cs.setInt(2, stock);
            
            if(!cs.execute())
            {
                resultado = true; 
            }


        } catch (SQLException ex) {
            System.out.println("Error" + ex.getMessage());

        }
        
        return resultado; 
        
    }
    
    
}

```

# Paso 6,Invocar nuevo web service, con datos de bd
Esta clase se puede visualizar en el paso 1. 

```java

package servicios;

import Modelo.Producto;
import Modelo.ProductoDao;
import java.util.List;
import javax.jws.WebService;
import javax.jws.WebMethod;
import javax.jws.WebParam;
import javax.jws.WebResult;


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

}
```

# Paso 7,Creacion app python 

Profesora recomienda usar una maquina virtual, para asi no danar otras instalaciones que tengamos: 
- pip install virtualenv
- pip install django

1. Crearemos la carpeta con el nombre de nuestro proyecto y la abriremos en vscode
2. crearemos maquina virtual dentro de carpeta : virtualenv ve_maquina
3. Activaremos la maquina llegando al archivo activate.ps1: .\ve_maquina\Scripts\activate [Windows, la profe usa source, que es un comando de mac y linux]
4. En mi caso, tuve que dar permisos para que ejecutara el archivo del paso 4, ya que me mostraba un error: Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
5. Para crear un cliente con python, en esta oportunidad lo usaremos con Django, usando la libreria zeep, que nos permitira crear clientes en django, para consumir nuestros web services. 
6. Crear proyecto de django: django-admin.py startproject clienteSoap 
7. crear app de python: python manage.py startapp core
8. agregar app a settings.py

# Paso 8, Creacion de templates o vistas que utilizaremos 

1. Creacion de archivos html que utilizaremos, primero seguramente seran los archivos base y despues iremos avanzando a medida de los webservices que tengamos construidos.
```html
<!-- base.html -->

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">

    {% load static %}

    <title>Cliente Bodega</title>
</head>
<body>

    {% block contenido  %}
        
    {% endblock contenido %}
    
</body>
</html>

<!--home.html -->
{% extends "./base.html" %}

{% block contenido  %}

    <script>

        $(function() {
            $("#form_buscar").on('submit', function(){
                var formData = new FormData(this); 

                $.ajax({
                    url: "{% url 'post_buscar' %}", 
                    type: "POST", 
                    data: formData, 
                    processData: false,
                    contentType: false, 
                    success: function(respuesta){
                        $("#nombre").val(respuesta.nombre);
                        $("#stock_disponible").val(respuesta.stock); 
                        $("#precio").val(respuesta.precio); 
                        $("#codigo_oculto").val(respuesta.codigo);
                    }
                }); 

                return false; 
            }); 


            
            $("#form_actualizar").on('submit', function(){
                var formData = new FormData(this); 

                $.ajax({
                    url: "{% url 'post_actualizar' %}", 
                    type: "POST", 
                    data: formData, 
                    processData: false,
                    contentType: false, 
                    success: function(respuesta){
                        var mensaje = respuesta.mensaje; 
                        //TODO
                        // $('#table').bootstrapTable('refresh');
                        alert(mensaje);
                        
                    }
                }); 

                return false; 
            })

        })
    </script>

    <h3>Formulario</h3>

    <form method="post" id="form_buscar">
        {% csrf_token %}

        <div class="form-group">
            
            <span>Ingrese Codigo: </span>
            <input type="text" name="codigo" id="codigo" />

            <button type="submit" class="btn btn-success">Buscar Libro</button>

            
        </div>

    </form>

    <hr>
    
    <form method="post" id="form_actualizar">
        {% csrf_token %}

        <div class="form-group">
            

            <span>Nombre</span>
            <input type="text" name="nombre" id="nombre" readonly />
            
            <span>Stock: </span>
            <input type="text" name="stock" id="stock_disponible" readonly />

            <span>Precio </span>
            <input type="text" name="precio" id="precio" readonly />

            <input type="hidden" name="codigo_oculto" id="codigo_oculto"/>
            <span>Ingrese Cantidad a comprar: </span>
            <input type="text" name="stock" id="stock"/>

            <button type="submit" class="btn btn-success">Actualizar Stock</button>

        </div>

    </form>


    <h3>{{mensaje}}</h3>

    <table 
        data-toggle="table" 
        data-sort-name="stargazers_count"
        data-sort-order="desc"
        data-pagination="true"
        data-search="true"
        id="table"
    >

    <thead class="thead-inverse">
        <th  data-sortable="true">Codigo Libro</th>
        <th  data-sortable="true">Nombre Libro</th>
        <th  data-sortable="true">Stock Libro</th>
        <th  data-sortable="true">Precio libro</th>
        <th  data-sortable="true">Vigencia Libro</th>
    </thead>

    <tbody>
        {% for libro in lista %}
            <tr>
                <td>{{libro.codigo}}</td>
                <td>{{libro.nombre}}</td>
                <td>{{libro.precio}}</td>
                <td>{{libro.stock}}</td>
                <td>{{libro.vigencia}}</td>
            </tr>
        {% endfor %}
    </tbody>

    </table>
        
{% endblock contenido %}

```




2. crear archivo url.py en core. Ejemplo clases: 


```python
from django.urls import path
from .views import home
from .views import buscarLibro
from .views import actualizarLibro


urlpatterns = [
    path('', home, name="home"),
    path('post/ajax/buscar', buscarLibro, name="post_buscar"),
    path('post/ajax/actualizar', actualizarLibro, name="post_actualizar")
]

#incluir url creado en urls del proyecto :

from django import urls
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('', include('core.urls')),
    path('admin/', admin.site.urls),
]

```

# Paso 9, crear clases que utilizaremos, ejemplo: 

```python



class Libro:
    codigo = ''
    nombre = ''
    stock = ''
    precio = ''
    vigencia = ''

    def __init__(self, codigo, nombre, stock, precio, vigencia):
        self.codigo = codigo
        self.nombre = nombre
        self.stock = stock
        self.precio = precio
        self.vigencia = vigencia

    def libroArr(self):
        return {
            'codigo': self.codigo,
            'nombre': self.nombre,
            'stock': self.stock,
            'precio': self.precio,
            'vigencia': self.vigencia
        }

    def __str__(self):
        return 'codigo: ' + self.codigo + 'nombre: '+self.nombre + ' stock: ' + self.stock + ' precio: ' + self.precio + ' vigencia: ' + self.vigencia

```

# Paso 10, crear controladores que utilizaremos: 

Aqui es donde se conectara con el web service. 
```python
from zeep import Client
from .libro import Libro


class Controller:
    wsdl = 'http://localhost:8080/servicioGesBod/ServicioGesbod?WSDL'
    client = Client(wsdl)

    def buscarTodos(self):
        listaLibro = []
        lista = self.client.service.consultarProductos()
        for i in range(len(lista)):
            libro = Libro(
                lista[i]['codigo_producto'],
                lista[i]['nombre_producto'],
                lista[i]['precio'],
                lista[i]['stock'],
                lista[i]['vigencia'],
            )
            listaLibro.append(libro)

        return listaLibro

    def buscarLibro(self, codigo):
        print(codigo)
        libro = self.client.service.getProducto(codigo)
        return libro

    def actualizarStock(self, codigo, stock):
        resultado = self.client.service.actualizarStock(codigo, stock)
        return resultado
```

# paso 11, llamar a los controladores desde archivo views.py
```python
from .controller import Controller
from django.shortcuts import render
from .libro import Libro

from django.http import JsonResponse

# Create your views here.


def home(request):
    variables = {
        'mensaje': '',
        'lista': ''
    }

    try:
        controller = Controller()
        lista = controller.buscarTodos()
        variables['lista'] = lista
        variables['mensaje'] = "Datos correctos"
    except:
        variables['mensaje'] = "Sin Datos"

    return render(request, 'core/home.html', variables)


def buscarLibro(request):
    controller = Controller()
    codigo = request.POST.get('codigo')
    libro = controller.buscarLibro(codigo)
    return JsonResponse({
        'codigo': libro.codigo_producto,
        'nombre': libro.nombre_producto,
        'stock': libro.stock,
        'precio': libro.precio,
        'vigencia': libro.vigencia
    })


def actualizarLibro(request):
    controller = Controller()
    codigo = request.POST.get('codigo_oculto')
    stock = request.POST.get('stock')
    resultado = controller.actualizarStock(codigo, stock)
    return JsonResponse({
        'mensaje': resultado
    })
```



# Creacion Clientes JAVA




