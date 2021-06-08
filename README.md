# TODO-IPY-U2

#API JAX-WS
Tecnologia Java para construir Clientes y Servicios Web, basados en XML y SOAP. 
Esta api oculta la complejidad de SOAP, resolviendo problemas asociados a creacion manual de WSDL. 

Cliente         <---    SOAP      --->      Web Service
(JAX-WS Runtime)      (Mensaje)             (JAX-WS Runtime)

```java
   import javax.jws.WebService;
   import javax.jws.WebMethod;
   import javax.jws.WebParam;
  
    //actualiza stock
    @WebMethod(operationName="actualizarStock")
    @WebResult(name="resultado")   
    public boolean updateStock( @WebParam(name="codigo_produicto") int codigo_producto, @WebParam(name="cantidadProducto") int stock_producto )
    {
        ProductoDao prodDao = new ProductoDao();
        boolean resultado = prodDao.fun_actualizarStock(codigo_producto, stock_producto); 
        return resultado; 
        
    }
```

1. 
