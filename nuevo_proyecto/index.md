
# Creación de un nuevo proyecto
1. Ingrese a Visual Studio 2022 y seleccione la opción **Crear un proyecto**  

![image](./img/crear_un_proyecto.png)  

2. Busque y seleccione la plantilla **Aplicación web ASP.NET Core (Modelo-Vista-Controlador)**.  

![image](./img/tipo_aplicacion.png)  

3. Haga clic en **Siguiente**  

4. Puede cambiar las configuraciones de su proyecto en el siguiente cuadro de diálogo. Luego haga clic en **Siguiente**.  

![image](./img/configure_su_nuevo_proyecto.png)  

5. En el siguiente cuadro de diálogo se muestra la versión del Framework. Sugiero que no la cambie. Solo haga clic en **Crear**  

![image](./img/informacion_adicional.png)  

Así se mostrará su nuevo proyecto ya creado:  

![image](./img/proyecto_creado.png)  

# Creación de un contexto a partir de una base de datos existente.

En este caso se utilizará **Microsoft SQL Server**

1. Haga clic derecho en el nombre del proyecto (**WebApplication1**)

2. Seleccione la opción **Administrar paquetes NuGet...**

3. Instale los paquetes siguientes:  

![image](./img/paquete1.png)  

![image](./img/paquete2.png)  

![image](./img/paquete3.png)  

Este es el cuadro de diálogo que se utiliza para instalar los paquetes indicados arriba:  

![image](./img/instalacion_paquetes.png)  

4. Crear las entidades a partir de la base de datos.

* Vaya al menú **Herramientas**.
* Seleccione la opción **Administrador de paquetes NuGet**
* Seleccione **Consola del administrador de paquetes**  
* Escriba el comando para generar el contexto. 

***FORMA 1***

```
Scaffold-DbContext "Server=ITCHAD32;Database=db1;Trusted_Connection=True; Trust Server Certificate=true;" Microsoft.EntityFrameworkCore.SqlServer -OutputDir Models -force
```

***FORMA 2***

```
Scaffold-DbContext "Server=ITCHAD32;Database=db1;User Id=sa;Password=adminsql; Trust Server Certificate=true;" Microsoft.EntityFrameworkCore.SqlServer -OutputDir Models -force
```

Cuando se ha creado el contexto, se presenta un mensaje como el siguiente:  

![image](./img/mensaje_satisfactorio.png)  

A continuación se muestra el contexto creado:  

![image](./img/contexto_creado.png)  

El modelo está creado en función de las tablas de la base de datos **bd1** que actualmente solo tiene la tabla **Productos**  

![image](./img/tablas_bd1.png)  

# ACTUALIZAR EL CONTEXTO

Si agrega nuevas tablas, puede actualizar el contexto utilizando el siguiente comando:  

```
Scaffold-DbContext "Server=ITCHAD32;Database=db1;User Id=sa;Password=adminsql; Trust Server Certificate=true;" Microsoft.EntityFrameworkCore.SqlServer -OutputDir Models -Tables Empleados, Usuarios, Productos -force
```

Actualizar solo una tabla  

```
Scaffold-DbContext "Server=ITCHAD32;Database=bd1;User Id=sa;Password=adminsql; Trust Server Certificate=true;" Microsoft.EntityFrameworkCore.SqlServer -OutputDir Models -table Productos -force
```  
