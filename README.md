# Practica 2 (Spark Streaming - Kafka)


## Parte de Investigación

En la maquina virtual tenemos instalado Zeppelin, para arrancar el programa tenemos que ir a la carpeta __bin__ donde está instalado el programa.

![Carpeta instalacion](/imagenes/z01.jpg "Carpeta instalación")

Para arrancar Zeppelin tenemos que ejecutar la siguiente sentencia:

      ./zeppelin-daemon.sh start

![Programa instalacion](/imagenes/z02.jpg "Programa instalación")

Cuando se ejecute con exito dirá un mensaje como el siguiente:

![Programa arrancado](/imagenes/z05.jpg "Programa arrancado")

Entonces para ver si el programa esta arrancado, vamos al **Firefox** en nuestro caso en la maquina virtual y tecleamos la URL:

      http://localhost:8081
      ó
      http://127.0.0.1:8081
 
![Inicio Zeppelin](/imagenes/z1.jpg "Inicio Zeppelin")

Una vez que tenemos arrancado, vamos a proceder a configurar el SPARK que tenemos instalado en nuestra MV, la versión 

      versión 2.4.4
Para ello tenemos que pulsar en donde pone

![Anonymous](/imagenes/z3.jpg "Anonymous")

Entonces nos saldrá un menú y tenemos que pulsar en la opción **Interpreter**

![Menu](/imagenes/z4.jpg "Menú")

Una vez que estamos en **Interpreters**

![Interpreters](/imagenes/z5.jpg "Interpreters")

Lo que tenemos que hacer ahora es crear un nuevo __Interpreters__, entonces tenemos que pulsar en el botón **Create**

![Interpreters create](/imagenes/z50.jpg "Interpreters create")

Nos aparece una pantalla para configurar el nuevo Interpreters

![Interpreters nuevo](/imagenes/z6.jpg "Interpreters nuevo")

Entonces temos que rellenar los casillas siguientes:

![Interpreters casillas](/imagenes/z7.jpg "Interpreters casillas")
![Interpreters casillas1](/imagenes/z9.jpg "Interpreters casillas1")

Luego pulsamos a **Save**

![Interpreters save](/imagenes/z10.jpg "Interpreters save")

Entonces nos lleva de nuevo al pantalla de **Interpreters**, y le podemos buscar

![Interpreters spark2](/imagenes/z11.jpg "Interpreters spark2")

Para que los cambios se materialicen tenemos que pulsar sobre el boton de **restart**

![Interpreters restart](/imagenes/z111.jpg "Interpreters restart")

Y pulsamos **OK**, sobre la venta de diálogo que sale a continuación

![Interpreters dialog](/imagenes/z13.jpg "Interpreters dialog")

Nos llevará al ventana principal de **Zeppelin**, una vez alli procederemos a crear un nuevo **Notebook**

![Notebook nuevo](/imagenes/z141.jpg "Notebook nuevo")

A nuestro nuevo Notebook le llamaremos **Numero de registros**, pero lo más importante es elegir el **Interpreter** que nosotros hemos creado anteriormente llamado **spark2**

![Note nuevo](/imagenes/z14.jpg "Note nuevo")

Una vez que hemos pulsado sobre **Create**, se nos abre el nuevo Note creado a poder realizar el ejercicio.

![Note editar](/imagenes/z15.jpg "Note editar")

Para comprobar con que versión de Spark estamos utilizando, ejecutamos la siguiente sentencia:

          sc.version
![Version Spark](/imagenes/z21.jpg "Version Spark")

Con esto comprobamos que tenemos bien configurado Zeppelin para realizar el ejercicio.

### Ejercicio

![Parte1 eje](/imagenes/zeje1.jpg "Parte1 eje")
![Parte2 eje](/imagenes/zeje2.jpg "Parte2 eje")
![Parte3 eje](/imagenes/zeje3.jpg "Parte3 eje")
![Parte4 eje](/imagenes/zeje4.jpg "Parte4 eje")
![Parte5 eje](/imagenes/zeje5.jpg "Parte5 eje")
![Parte6 eje](/imagenes/zeje6.jpg "Parte6 eje")
![Parte7 eje](/imagenes/zeje7.jpg "Parte7 eje")
![Parte8 eje](/imagenes/zeje8.jpg "Parte8 eje")
![Parte9 eje](/imagenes/zeje9.jpg "Parte9 eje")

