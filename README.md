# Wordpress tarea

# Índice

1. [Introducción](#introducción)
   - [Objetivo de la práctica](#objetivo-de-la-práctica)
   - [Estructura del proyecto](#estructura-del-proyecto)

2. [Infrastructura](#infrastructura)
   - [Creación de la VPC](#creación-de-la-vpc)
     - [¿Qué es una VPC?](#qué-es-una-vpc)
     - [Creación de subredes](#creación-de-subredes)
   - [Creación de las instancias](#creación-de-las-instancias)
     - [Balanceador](#balanceador)
     - [NFS](#nfs)
     - [Servidores Web (2)](#servidores-web-2)
     - [BBDD](#bbdd)
   - [Configuración de grupos de seguridad](#configuración-de-grupos-de-seguridad)
     - [Grupo de seguridad Balanceador](#grupo-de-seguridad-balanceador)
     - [Grupo de seguridad Servidores Web](#grupo-de-seguridad-servidores-web)
     - [Grupo de seguridad BBDD](#grupo-de-seguridad-bbdd)

3. [Provisionamiento de las instancias](#provisionamiento-de-las-instancias)
   - [Instancia Balanceador](#instancia-balanceador)
   - [Instancia NFS](#instancia-nfs)
   - [Instancias Servidores Web](#instancias-servidores-web)
   - [Instancia MySQL](#instancia-mysql)


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

![image](https://github.com/user-attachments/assets/824f8119-e651-4fc4-9097-838a790e8082)


Una vez en la consola de VPC hay un botón que destaca más que los demás, ahí es dónde hay que darle para crear la VPC.
![image](https://github.com/user-attachments/assets/a035a5aa-9597-4b3a-b022-4c73b7098019)



Se dirigirá directamente a la consola de creación de VPC.

Hay que definir el nombre y la cantidad de subredes, todo muy intuitivo y nada complejo. En VPC y más te da la opción de crear las subredes mientras creas la VPC, esa es la diferencia entre **solo la VPC**  **y VPC y más.**
![image](https://github.com/user-attachments/assets/321db33c-0a56-4674-8d26-6882645dfd4f)


El siguiente paso es definir la red en la que quieres crear la VPC y una vez definido ya da paso a las subredes.  
![image](https://github.com/user-attachments/assets/39cc6e35-68cd-43dc-8b90-30534489bc9a)



### Creación de subredes

El mínimo de hosts que debes de tener en una subred de AWS es de 16 por lo que no se puede pasar de **/28.** 

Es necesario tener una subred pública para el balanceador y 2 privadas, una privada para los servidores web y el NFS y la 2ª subred privada para el servidor de BBDD.

**VPC y más** te da la opción por defecto de crear 2 subredes públicas y 2 privadas basándose en la dirección de red que se ha especificado anteriormente. Luego solo hay que asociar las instancias a las subredes pertinentes.

![image](https://github.com/user-attachments/assets/1b62f83a-b585-4f76-b7b4-1e20b31ee86a)


Por último solo queda crear la VPC.

![image](https://github.com/user-attachments/assets/2f1e604a-ddcd-44ae-b342-91432f2bd1a1)


## Creación de las instancias

Hay que crear un total de 5 instancias.

- Balanceador
- NFS
- Servidores Web (2)
- BBDD

Hay que dirigirse a la consola de EC2 para crear las instancias.

![image](https://github.com/user-attachments/assets/aff26658-a06d-4d0e-89c1-c8617145874b)


![image](https://github.com/user-attachments/assets/f8aed546-9952-45f8-9fa1-9133b969699d)


Después de lanzar la instancia hay que definir el nombre y el tipo de ami que se va a utilizar.

![image](https://github.com/user-attachments/assets/6fb65620-130d-4e88-a4a4-ee37343d1629)

En tipo de instancia yo he escogido la t2.small aunque tenga un coste ligeramente superior.

![image](https://github.com/user-attachments/assets/9ef4add3-ee99-4dda-983e-2a13895c7861)


Y en configuraciones de red se aplica la VPC y subred correspondiente, al ser el ejemplo de la máquina balanceador se creará en la subred pública.

![image](https://github.com/user-attachments/assets/2bc7e14b-5a82-42cd-951a-235ad5ff18b7)


### Configuración de grupos de seguridad

Para los grupos de seguridad hay 3 creados, uno por capa, cada uno permitiendo los respectivos paquetes. Todo lo que no sea permitido significa que está denegado.

**NOTA: Todos permiten el tráfico SSH para poder conectarme en caso de error.**

### Grupo seguridad balanceador.

Permite el tráfico de entrada HTTP/S y SSH.

![image](https://github.com/user-attachments/assets/eeba1999-834b-4b27-98a7-414e12e00e81)


### Grupo seguridad ServidoresWEB.

Permite el tráfico de entrada HTTP, NFS y MSSQL.

![image](https://github.com/user-attachments/assets/05a8faf8-13f9-46b6-a41a-49cf689222b9)


### Grupo seguridad BBDD.

Permite el tráfico de entrada MYSQL.

![image](https://github.com/user-attachments/assets/6be1014c-92f1-4bfd-b343-47fb587772a8)


# 2. Provisionamiento de las instancias

## Instancia balanceador.
Se ha asociado una IP elástica con la siguiente IP y DOMINO.
![image](https://github.com/user-attachments/assets/da40ecfd-8b68-472b-a131-4de4d5f31dde)


La función de esta instancia es redirigir los paquetes a las máquinas que actúan como servidores web siguiendo un método Round Robin.

Para el correcto funcionamiento de la instancia que ejerce la función de balanceador ha sido necesario ejecutar los comandos que se ven en la imagen. Se ha utilizado ese script para el funcionamiento del balanceador.

1.  Instalar los paquetes 
2. Habilitar los módulos de balanceador y proxy
3. Copia el sitio virtual por defecto y crea uno nuevo
4. Comenta la linea **Document root** de /etc/apache2/sites-available/000-default.conf y añade la configuración de balanceador.
5. Deshabilita el sitio virtual por defecto y habilita el nuevo, por último reinica apache para aplicar los cambios.
6. Por último fuera del fichero se han hecho una serie de comandos para que funcione el sitio seguro.

![image](https://github.com/user-attachments/assets/948f1e2e-0e11-496d-81fe-797488b51af8)


## Instancia NFS

La única función que tiene el NFS es alojar los recursos a los que acceden los servidores web.

1. Primero instala los paquetes. Servidor nfs, programa para descomprimir y descargar php y mysql para comprobaciones.
2. Crea la carpeta compartida, asigna permisos y añade en el /etc/exports el rango de ip que pueden acceder al recurso y que permisos tienen.
3. Instala wordpress dentro de la carpeta compartida, asigna los permisos y la carpeta compartida y reinicia el servidor para aplicar cambios.

![image](https://github.com/user-attachments/assets/86e2282e-1a9e-4836-afe0-39efd86dd9db)


## Instancias servidores WEB

Los servidores web deben acceder a la carpeta compartida del servidor NFS. Los sitios virtuales de apache apuntan hacía la carpeta compartida del NFS.

1. Instala apache, los módulos PHP y el cliente NFS
2. Cambia el DocumentRoot apuntando a la carpeta compartida y añade unas líneas y habilita el punto .htacces en el directorio compartido.
3. Copia el sitio default que ha sido editado y lo pega en un nuevo sitio que será el que sea habilitado.
4. Crea la carpeta en la que va a estar el contenido compartido, la monta, habilita el nuevo sitio virtual y habilita el  nuevo sitio virtual deshabilitando el default.
    
![image](https://github.com/user-attachments/assets/55b137ce-992d-4abd-b67f-3f0ecc6b1d76)

    

```bash
#el siguiente comando lo que hace es realizar el automontado de la carpeta compartida.
echo “192.168.10.29:/var/nfs/shared     /nfs/shared nfs auto,nofail,noatime,nolock,intr,tcp,actimeo=1800 0 0” >> /etc/fstab
```

## Instancia MYSQL

En esta instancia lo único necesario es crear una base de datos con un usuario con permisos para editarla.

1. Instalar los paquetes necesarios como el servicio mysql.
2. Crear una base de datos especificando el rango de red desde donde pueden tener acceso a la base de datos.

![image](https://github.com/user-attachments/assets/df787bd6-bf7c-4191-a669-c1741b7cd980)

