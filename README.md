# Wordpress tarea

Curso: Aplicaciones Web (https://www.notion.so/Aplicaciones-Web-134efbb7467181749017e02281325d39?pvs=21)
Fecha: 5 de diciembre de 2024 → 5 de diciembre de 2024
Tipo: Tarea
Estado: En Proceso

# 1. Introducción

El objetivo de la práctica es desplegar un CMS en 3 capas en AWS. Para conseguirlo he creado varios ficheros de provisionamiento.

- 1 Fichero para el balanceador
- 1 Fichero para el NFS
- 1 Fichero para los servidores web (2)
- 1 Fichero para la base de datos.

Después he creado 1 VPC en AWS con 3 subredes y he configurado las tablas de enrutamiento correspondientes para que solo se pueda acceder desde fuera a la máquina que actúa como balanceador. 

**NOTA: Los pasos a seguir son solo un ejemplo de como crear los objetos en AWS, no es como lo he hecho yo realmente en el momento de realizar la práctica.**

Infrastructura 

## Creación de la VPC

### ¿Qué es una VPC?

Una VPC es una parte aislada de la nube de AWS que contiene objetos de AWS, como instancias de Amazon EC2.

Para la cración de la VPC hay que dirigirse al apartado de **VPC** en AWS.

![image.png](image.png)

Una vez en la consola de VPC hay un botón que destaca más que los demás, ahí es dónde hay que darle para crear la VPC.

![image.png](image%201.png)

Se dirigirá directamente a la consola de creación de VPC.

Hay que definir el nombre y la cantidad de subredes, todo muy intuitivo y nada complejo. En VPC y más te da la opción de crear las subredes mientras creas la VPC, esa es la diferencia entre **solo la VPC**  **y VPC y más.**

![image.png](image%202.png)

El siguiente paso es definir la red en la que quieres crear la VPC y una vez definido ya da paso a las subredes.

![image.png](image%203.png)

### Creación de subredes

El mínimo de hosts que debes de tener en una subred de AWS es de 16 por lo que no se puede pasar de **/28.** 

Es necesario tener una subred pública para el balanceador y 2 privadas, una privada para los servidores web y el NFS y la 2ª subred privada para el servidor de BBDD.

**VPC y más** te da la opción por defecto de crear 2 subredes públicas y 2 privadas basándose en la dirección de red que se ha especificado anteriormente. Luego solo hay que asociar las instancias a las subredes pertinentes.

![image.png](image%204.png)

Por último solo queda crear la VPC.

![image.png](image%205.png)

## Creación de las instancias

Hay que crear un total de 5 instancias.

- Balanceador
- NFS
- Servidores Web (2)
- BBDD

Hay que dirigirse a la consola de EC2 para crear las instancias.

![image.png](image%206.png)

![image.png](image%207.png)

Después de lanzar la instancia hay que definir el nombre y el tipo de ami que se va a utilizar.

![image.png](image%208.png)

En tipo de instancia yo he escogido la t2.small aunque tenga un coste ligeramente superior.

![image.png](image%209.png)

Y en configuraciones de red se aplica la VPC y subred correspondiente, al ser el ejemplo de la máquina balanceador se creará en la subred pública.

![image.png](image%2010.png)

### Configuración de grupos de seguridad

Para los grupos de seguridad hay 3 creados, uno por capa, cada uno permitiendo los respectivos paquetes. Todo lo que no sea permitido significa que está denegado.

**NOTA: Todos permiten el tráfico SSH para poder conectarme en caso de error.**

### Grupo seguridad balanceador.

Permite el tráfico de entrada HTTP/S y SSH.

![image.png](image%2011.png)

### Grupo seguridad ServidoresWEB.

Permite el tráfico de entrada HTTP, NFS y MSSQL.

![image.png](image%2012.png)

### Grupo seguridad BBDD.

Permite el tráfico de entrada MYSQL.

![image.png](image%2013.png)

# 2. Provisionamiento de las instancias

## Instancia balanceador.

La función de esta instancia es redirigir los paquetes a las máquinas que actúan como servidores web siguiendo un método Round Robin.

Para el correcto funcionamiento de la instancia que ejerce la función de balanceador ha sido necesario ejecutar los comandos que se ven en la imagen. Se ha utilizado ese script para el funcionamiento del balanceador.

1.  Instalar los paquetes 
2. Habilitar los módulos de balanceador y proxy
3. Copia el sitio virtual por defecto y crea uno nuevo
4. Comenta la linea **Document root** de /etc/apache2/sites-available/000-default.conf y añade la configuración de balanceador.
5. Deshabilita el sitio virtual por defecto y habilita el nuevo, por último reinica apache para aplicar los cambios.

![image.png](image%2014.png)

## Instancia NFS

La única función que tiene el NFS es alojar los recursos a los que acceden los servidores web.

1. Primero instala los paquetes. Servidor nfs, programa para descomprimir y descargar php y mysql para comprobaciones.
2. Crea la carpeta compartida, asigna permisos y añade en el /etc/exports el rango de ip que pueden acceder al recurso y que permisos tienen.
3. Instala wordpress dentro de la carpeta compartida, asigna los permisos y la carpeta compartida y reinicia el servidor para aplicar cambios.

![image.png](image%2015.png)

## Instancias servidores WEB

Los servidores web deben acceder a la carpeta compartida del servidor NFS. Los sitios virtuales de apache apuntan hacía la carpeta compartida del NFS.

1. Instala apache, los módulos PHP y el cliente NFS
2. Cambia el DocumentRoot apuntando a la carpeta compartida y añade unas líneas y habilita el punto .htacces en el directorio compartido.
3. Copia el sitio default que ha sido editado y lo pega en un nuevo sitio que será el que sea habilitado.
4. Crea la carpeta en la que va a estar el contenido compartido, la monta, habilita el nuevo sitio virtual y habilita el  nuevo sitio virtual deshabilitando el default.
    
    ![image.png](image%2016.png)
    

```bash
#el siguiente comando lo que hace es realizar el automontado de la carpeta compartida.
echo “192.168.10.29:/var/nfs/shared     /nfs/shared nfs auto,nofail,noatime,nolock,intr,tcp,actimeo=1800 0 0” >> /etc/fstab
```

## Instancia MYSQL

En esta instancia lo único necesario es crear una base de datos con un usuario con permisos para editarla.

1. Instalar los paquetes necesarios como el servicio mysql.
2. Crear una base de datos especificando el rango de red desde donde pueden tener acceso a la base de datos.

![image.png](image%2017.png)
