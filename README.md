# TODO-IPY-U2

API JAX-WS
Tecnologia Java para construir Clientes y Servicios Web, basados en XML y SOAP. 
Esta api oculta la complejidad de SOAP, resolviendo problemas asociados a creacion manual de WSDL. 

Cliente         <---    SOAP      --->      Web Service
(JAX-WS Runtime)      (Mensaje)             (JAX-WS Runtime)

```java
   import javax.jws.WebService;
   import javax.jws.WebMethod;
   import javax.jws.WebParam;
  
   @WebService (serviceName = "nombreServicio" )
   {
    
       @webMethod(operationName = "Hello")
       public String hello(@WebParam(name="name") String txt)
       {
         return "Hello" + txt + " !"; 
       }
  
   }
```

1. 
