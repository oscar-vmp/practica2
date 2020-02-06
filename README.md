# Practica 2 (Spark Streaming - Kafka)

## Primare parte

Envio de un ficheco __.json__ mediante Kafka através de la consola (PRODUCTOR) y recoger el resultado del fichero enviado anteriormente desde la consula del (CONSUMIDOR).

Tenemos un fichero __.json__ que le hemos alojado en la carpeta:

![Carpeta instalacion](/imagenes/carpeta_ficheros.jpg "Carpeta instalación")

Si abrimos el fichero __personal.json__, podemos ver el contenido y que formato:

![Fichero_json](/imagenes/fichero_json.jpg "Fichero json")

Para probar esta primera partes tenemos que abrir dos termianles, una para que simule ser un __PRODUCTOR__ y el otro terminal que simule ser el __CONSUMIDOR__

En parte del PRODUCTOR lo primero que tenemos que hacer es crear un topic, en nuestro caso el *topic* se va a llamar **topicPractica**, al cual el __CONSUMIDOR__ se va subscribir. Para realizar esto, en el terminal del PRODUCER ejecutaremos:

      bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic topicPractica

Una vez tenemos ya creado el topic lo que tenemso que lanzar el fichero __personal.json__ a traver del __kafka-console-producer.sh__ y para ello tenemos que ejecutar la siguiente sentencia:

      cat /home/keepcoding/Documentos/personal.json| bin/kafka-console-producer.sh --broker-list localhost:9092 --topic topicPractica > /dev/null

![Productor_sh](/imagenes/productor_sh.jpg "Productor sh")

Por otra parte, en el terminal que simularemos el CONSUMIDOR, tendremos que ejecutar la siguiente sentencia:

      bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic topicPractica --from-beginning

![Consumidor_sh](/imagenes/consumidor_sh.jpg "Consumidor sh")

Una vez lanzado el fichero por el __PRODUCTOR__, vemos abajo como lo recoge el fichero el __CONSUMIDOR__:

![Resulatdo_envio_sh](/imagenes/resultado_pc_sh.jpg "Resultado del envio por sh")

## SEGUNDA PARTE

Debbemos enviar el fichero  __personal.json__ del apartado anterior mediante la consolsa (PRODUCTOR), y luego generar un código en SCALA que lo recoja y realize una transformación, por el cual de los registros que trae el fichero, filtrar los registros que cumplan una condicion.
El fichero como indicamos anteriormente se encuentra ubicado:

      /home/keepcoding/Documentos


El fichero __personal.json__  tiene la siguente estructura:

![Fichero_json](/imagenes/fichero_json.jpg "Fichero json")

Al resultado que queremos llegar tras ser procesado por el CONSUMER es tener el subconjunto de registros como el siguiente:

![Fichero_json_final](/imagenes/fichero_json_final.jpg "Fichero json final")


Lo primero que vamos hacer es un nuevo Proyecto en __IntelliJ__:

![Proyecto nuevo](/imagenes/proyectto.jpg "Proyecto nuevo")

Para nuestro proyecto que vamos a utizar *Kafka* y *Spark Sql*, necesitamos cargar las siguientes dependencias:

      libraryDependencies += "org.apache.spark" %% "spark-sql" % "2.4.0"% "Provided"
      libraryDependencies += "org.apache.spark" %% "spark-streaming-kafka-0-10" % "2.4.0"
      libraryDependencies += "org.apache.spark" %% "spark-sql-kafka-0-10" % "2.4.0"

El siguiente paso será crear el fichero código, para que haga de CONSUMIDOR de fichero __personal.json__ enviado por el PRODUCTOR. En nuestro caso el objeto de scala que se ha creado es el siguiente __consumer.scala__:

      package kafka


      import org.apache.log4j.{Level, Logger}
      import org.apache.spark.sql.SparkSession
      import org.apache.spark.sql.types.{IntegerType, StringType, StructType}
      import org.apache.spark.sql.functions.{from_json,col}


      object consumer {
            def main(args: Array[String]): Unit= {
                  Logger.getLogger("practica").setLevel(Level.ERROR)

                  val spark = SparkSession.builder().appName("practica").master("local[2]").getOrCreate()

                  val df = spark.readStream
                              .format("kafka")
                              .option("kafka.bootstrap.servers","localhost:9092")
                              .option("subscribe","topicPractica")
                              .option("startingOffsets","earliest")
                              .load()
                  df.printSchema()
                  //castear los datos leidos en formato kafka para convertirlos en Strings
                  val res=df.selectExpr("CAST(value AS STRING)")
                  val schema=new StructType()
                                    .add("id",IntegerType)
                                    .add("first_name",StringType)
                                    .add("last_name",StringType)
                                    .add("email",StringType)
                                    .add("gender",StringType)
                                    .add("ip_address",StringType)
                  import spark.implicits._
                  val persona=res.select(from_json(col("value"),schema).as("data"))
                                    .select("data.*")
                                    .filter("data.first_name not in ( 'Noell','Jeanette')")

                  persona.writeStream
                              .format("console")
                              .outputMode("append")
                              .start()
                              .awaitTermination()

            }
      }

Aquí en el código poemos ver que le CONSUMIDOR est asubscrito a un topic llamado *topicPractica*, y luego para generar lo que la práctica pide, se ha tenido que realizar una *transformación*, en concreto se ha utilizado la transformación **filter**.

![Filtro](/imagenes/filtro.jpg "Transformación aplicada")

Una vez ya tenemos el CONSUMIDOR creado, la siguiente paso es probar que funciona, para ello ponemos el PRODUCTOR y el CONSUMIDOR a funcionar:

      cat /home/keepcoding/Documentos/personal.json| bin/kafka-console-producer.sh --broker-list localhost:9092 --topic topicPractica > /dev/null     

![Productor_sh](/imagenes/productor_sh.jpg "Productor sh")

El salida que sale del consumidor es la que esperabamos:

![Salida](/imagenes/salida.jpg "Salida")


Ahora se ha realizado una modificación en el código del CONSUMIDOR, que consite en eliminar dos nombres de la columna _first_name_, dejando todas las filas:

      package kafka


      import org.apache.log4j.{Level, Logger}
      import org.apache.spark.sql.SparkSession
      import org.apache.spark.sql.types.{IntegerType, StringType, StructType}
      import org.apache.spark.sql.functions.{from_json,col,when}


      object consumer {
            def main(args: Array[String]): Unit= {
                  Logger.getLogger("practica").setLevel(Level.ERROR)

                  val spark = SparkSession.builder().appName("practica").master("local[2]").getOrCreate()

                  val df = spark.readStream
                              .format("kafka")
                              .option("kafka.bootstrap.servers","localhost:9092")
                              .option("subscribe","topicPractica")
                              .option("startingOffsets","earliest")
                              .load()
                  df.printSchema()
                  //castear los datos leidos en formato kafka para convertirlos en Strings
                  val res=df.selectExpr("CAST(value AS STRING)")
                  val schema=new StructType()
                        .add("id",IntegerType)
                        .add("first_name",StringType)
                        .add("last_name",StringType)
                        .add("email",StringType)
                        .add("gender",StringType)
                        .add("ip_address",StringType)
                  import spark.implicits._
                  val persona=res.select(from_json(col("value"),schema).as("data"))
                        .select(    col("data.id"),
                                    when (col("data.first_name")==="Noell" || col("data.first_name")==="Jeanette","")
                                          .otherwise(col("data.first_name")).alias("first_name"),
                                    col("data.last_name"),
                                    col("data.email"),
                                    col("data.gender"),
                                    col("data.ip_address"))
                        //.filter("data.first_name not in ( 'Noell','Jeanette')")

                  persona.writeStream
                        .format("console")
                        .outputMode("append")
                        .start()
                        .awaitTermination()

              }
        }

En este caso, en lugar de realizar una transformación, se ha realizado un _CASE_ en SQL, que en nuetro caso es realizar __WHEN__, como mostramos a continuación:

![Opcion segunda](/imagenes/opcion2.jpg "Segunda opción")

Posteriormente, si lo ejecutamos el resultado que sale es el que se muestra:

![Resultado segunda opcion](/imagenes/resultado2.jpg "Resultado segunda opción")


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
Vamos ejecutando por partes el código:

Traemos las *librerias* necesarias para la práctica

![Parte1 eje](/imagenes/Parte1.jpg "Parte1 eje")

Creamos un *SparkSession*

![Parte2 eje](/imagenes/Parte2.jpg "Parte2 eje")

Creamos un *schema* para asociarsele posteriormente el DataFrame del fichero que se carge:

![Parte3 eje](/imagenes/Parte3.jpg "Parte3 eje")

Se procede a cargar el fichero __amigos.csv__ y le asocia el *schema* que se ha creado anteriormente.

![Parte4 eje](/imagenes/Parte4.jpg "Parte4 eje")

Importamos la librería

![Parte5 eje](/imagenes/Parte5.jpg "Parte5 eje")

Se crea una tabla con el nombre de *amigos*

![Parte6 eje](/imagenes/Parte6.jpg "Parte6 eje")

Mediate Spark Sql,se ejecuta la sentencia Sql pata contar los registros de una tabla:

![Parte7 eje](/imagenes/Parte7.jpg "Parte7 eje")

Mediante la *acción* **collect**, nos taremos los datos del cluster a nuestro equipo

![Parte8 eje](/imagenes/Parte8.jpg "Parte8 eje")

Y posteriormente presentamos el datos de número de registros que nos pedía el enunciado

![Parte9 eje](/imagenes/Parte9.jpg "Parte9 eje")


El código que se ha ejecutado en Zeppelin ha sido el siguiente:

      import org.apache.spark.sql.SparkSession
      import org.apache.spark.sql.types.{IntegerType,StringType,StructType}

      val spark=SparkSession.builder().master("local").appName("MiApp1").getOrCreate()

      val schema=new StructType().add("id",IntegerType,false)
                        .add("nombre",StringType,true)
                        .add("edad",IntegerType,true)
                        .add("amigos",IntegerType,false)

      val df=spark.read.format("csv").schema(schema).option("delimiter",",").load("file:///home/keepcoding/Documentos/amigos.csv")
      import spark.implicits._
      df.createOrReplaceTempView("amigos")
      val numeroFilas = spark.sql("SELECT count(*) FROM amigos")
      val resultado=numeroFilas.collect()
      val numero_filas=resultado.head.getLong(0)
