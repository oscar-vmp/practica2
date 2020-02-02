# Practica 2 (Spark Streaming - Kafka)

## Primare parte

Creación de estructura Kafka STREAMING.
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


__consumer.scala__:

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
                              .option("sstartingOffsets","earliest")
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

