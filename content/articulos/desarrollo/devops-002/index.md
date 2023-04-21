--- 
author: agomezguru
date: 2023-04-01T13:18:52-06:00 
title: "Replicación maestro-esclavo en MySQL 8.x" 
tags: 
  - DevOps
  - DB's
  - Replicación
  - Taller
description: Ejemplo de replicación básica en MySQL 8 empleando contenedores de Docker - Taller práctico
summary: En esta ocasión revisaremos juntos un típico caso de uso de las bases de datos relacionales en un taller teórico práctico empleando contenedores de Docker.
categories: 
  - general
--- 


{{< bigcaps L >}}as bases de datos son uno de los elementos clave de toda aplicación web, a no ser que se trate de un simple blog desplegado como un sitio estático. Así, en esta ocasión revisaremos juntos un típico caso de uso de las bases de datos relacionales.
{.alphabet .mb-4}

Probablemente si en este momento te encuentras operando tu aplicación web y utilizas un sistema de base de datos relacional (RDBMS) y no cuentas con un presupuesto cuantioso, la probabilidad de que estés empleando [MySQL/MariaDB/Percona es bastante alta](https://db-engines.com/en/ranking), por ello la elegí como motor de base de datos, para mostrar en este *taller práctico* de ejemplo de uso de contenedores Docker. Además, si este es tu caso, seguramente te servirá de guía muy útil.
{.mb-4}

### Replicación maestro-esclavo
{.blog-desc-big .m-0 .mb-4}

Ciertamente eres un excelente programador. Tu y tu equipo de trabajo realizaron una gran labor, ahora tu aplicación ha empezado a recibir una gran cantidad de tráfico diario y las operaciones de lectura-escritura en disco están empezando a ser un problema. Ya lo solucionaste, cambiaste el disco de trabajo principal por uno de estado sólido, pero las cosas se están poniendo verdaderamente difíciles. Empezaste con 8 Mb de RAM, luego lo creciste a 16, 24, 32 y las cuentas $$$ de tu servidor en nube también están creciendo junto con ella. Es hora de cambiar de táctica, eso que has estado haciendo se denomina escalado vertical de una aplicación, llegó el momento de crecer hacia los lados (escalado horizontal) agregando más máquinas, pero eso también cuesta y es aquí donde entran los contenedores de Docker. Pero antes de imbuirnos en los detalles, primero definamos sin ambigüedades que es esto de la replicación maestro-esclavo.
{.mb-4}

La replicación es el proceso mediante el cual las tablas de un servidor de base de datos MySQL (el maestro) se sincronizan automáticamente a uno o más servidores de base de datos MySQL (los esclavos). En el caso de tener múltiples esclavos se denomina clúster esclavo. Muy importante, la replicación no debe confundirse con las operaciones de copia de seguridad, ya que estas tienen un objetivo totalmente distinto, proteger los datos o la estructura de datos. En contraposición, la función de la replicación es, primordialmente, distribuir las cargas de trabajo de lectura y escritura en varios servidores con fines de escalabilidad. En casos muy específicos, la replicación también se emplea para la conmutación por error (yo prefiero más el método blue/green) o para analizar datos en el esclavo, a fin de disminuir la carga de trabajo del maestro. Este último es otro caso de uso típico que podemos abordar en un futuro.
{.mb-4}

La replicación maestro-esclavo es casi siempre una replicación unidireccional, de maestro a esclavo, en la que la base de datos maestra se emplea para operaciones de escritura, mientras que las operaciones de lectura pueden realizarse en una única base de datos esclava o distribuirse entre varios esclavos. Esto se debe a que la proporción de solicitudes de lectura casi siempre es más alta que la de escritura para la mayoría de las aplicaciones. Además, es muy importante mantener la coherencia en los datos y así el maestro se convierte en la única fuente de verdad, aunque agrega un problema adicional, también se vuelve un único punto de falla (esto lo abordaré en otro artículo). Por otra parte, si tu servidor de base de datos de lectura (el esclavo) comienza a tener dificultades de sobrecarga, siempre podrás agregar servidores de lectura adicionales al clúster (escalado horizontal de los servidores de bases de datos).
{.mb-4}

Un escenario más raro es donde el(los) servidor(es) de base de datos realizan más escrituras que lecturas y requiere de otra topología llamada fragmentación y por el momento, no es tema.
{.mb-4}

El maestro y el(los) esclavo(s) pueden residir en el mismo servidor físico o en servidores diferentes, pero como ya lo mencioné, no queremos escalar los costos a la par, sino todo lo contrario. Por ello vamos a colocar ambos en la misma máquina empleando contenedores de Docker, que además ofrece algunas ventajas de rendimiento y seguridad al omitir el traslado de los datos a través de la red o la conexión a Internet.
{.mb-4}

El proceso de replicación Maestro -> Esclavo puede ser síncrono o asíncrono. La diferencia radica en el tiempo: si los cambios se realizan en el Maestro y el Esclavo al mismo tiempo, se dice que el proceso es sincrónico; si los cambios se ponen en cola y se escriben más tarde, el proceso se considera asíncrono.
{.mb-4}

Hay varias formas de propagar cambios a las bases de datos esclavas. Uno de ellos muy empleado es la replicación basada en triggers, en donde los triggers en las columnas del servidor maestro propagan los cambios de escritura al servidor esclavo.
{.mb-4}

{{< figure src="trigger_replication.jpeg" class="img-fluid mb-3" >}}

Otro mecanismo de replicación llamado replicación basada en filas se introdujo en MySQL 5.1. Es un tipo de registro binario que proporciona la mejor combinación entre integridad de datos y rendimientos. La replicación basada en filas emplea los archivos de registro de transacciones, copiándolos en la otra instancia donde serán ejecutados. Los archivos de registro están destinados a reproducir una versión funcional de la base de datos en caso de falla, por lo que representan un mecanismo ideal para replicar datos de manera sumamente segura sin causar bloqueos en las operaciones de escritura. Este tipo de replicación siempre funciona de forma asíncrona.
{.mb-4}

{{< figure src="log_replication.jpg" class="img-fluid mb-3" >}}

Una vez comprendidos los conceptos clave, daremos inicio a nuestro taller de replicación empleando contenedores Docker.
{.mb-4}

### Taller: Replicación maestro-esclavo en MySQL 8.x
{.blog-desc-big .m-0 .mb-4}

#### Prerrequisitos
{.mb-4}

{{< list >}}
1. Crear una cuenta en [DockerHub](https://hub.docker.com/)
1. Infraestructura de pruebas: [Play with Docker](https://labs.play-with-docker.com/)
1. Número de instancias: 2
1. Abre una instancia de [PWD](https://labs.play-with-docker.com/) en tu navegador
{{< /list >}}

#### Crear una cuenta en DockerHub
{.mb-4}

Para poder ejecutar nuestro taller sin necesidad de perder tiempo instalando cosas, voy a emplear la plataforma [Play with Docker](https://labs.play-with-docker.com/), pero primero debes crear una cuenta en [DockerHub.](https://hub.docker.com/)
{.mb-4}

Si aún no cuentas con una cuenta simplemente entra a la [siguiente liga](https://hub.docker.com/signup) y crea tu cuenta. Es totalmente gratis para uso personal.
{.mb-4}

Ahora con tu cuenta de DockerHub recién creada [accede aquí.](https://labs.play-with-docker.com/) Si al momento de entrar te sale un mensaje como el siguiente:
{.mb-4}

> :warning: `We are really sorry but we are out of capacity and cannot create your session at the moment. Please try again later.`

es indicativo de que el servicio está saturado, solo espera unos minutos y vuelve a intentarlo.
{.mb-4}

#### Algunos detalles de la configuración inicial
{.m-0 .mb-4}

{{< list >}}
- Imagen Docker a utilizar - percona/percona-server:8.0
- Servidor Maestro – 172.19.0.2/16
- Servidor Esclavo – 172.19.0.3/16
- MySQL Port – 3306
- Versión de Percona  – 8.0.32-24
{{< /list >}}

#### Creación de docker-compose.yml
{.m-0 .mb-4}

Para configurar nuestra pequeña infraestructura utilizaremos el siguiente archivo de docker-compose.yml, recuerda que en los archivos de [configuración de YAML](https://yaml.com) es muy importante respetar la identación con espacios:
{.mb-4}

```bash {style=friendly}
  mkdir taller-replicacion && cd $_ && mkdir agomezguru && cd $_

  cat <<EOF > docker-compose.yml
version: '3'

volumes:
  agomezguru-masterdb:
    external: true
  agomezguru-slavedb:
    external: true

services:
  master:
    image: percona/percona-server:8.0
    volumes:
      - agomezguru-masterdb:/var/lib/mysql
      - ../servidores/masterdb:/etc/my.cnf.d
      - ../servidores/respaldos:/respaldos
    environment:
      MYSQL_ROOT_PASSWORD: SinPasswordMaestro
    networks:
      - agomezguru-network

  slave:
    image: percona/percona-server:8.0
    volumes:
      - agomezguru-slavedb:/var/lib/mysql
      - ../servidores/slavedb1:/etc/my.cnf.d
      - ../servidores/respaldos:/respaldos
    depends_on:
      - db
    environment:
      MYSQL_ROOT_PASSWORD: SinPasswordEsclavo
    networks:
     - agomezguru-network

# Isolate docker containers arrays between environments.
networks:
  agomezguru-network:
    driver: bridge
EOF
```

Ahora necesitamos crear los volúmenes nombrados requeridos por las instancias de base de datos:
{.mt-4 .mb-4}

```bash  {style=friendly}
  docker volume create --name agomezguru-masterdb
  docker volume create --name agomezguru-slavedb
```

Ya para finalizar la construcción de nuestra pequeña cama de pruebas creamos los archivos de configuración de cada instancia:
{.mt-4 .mb-4}

```bash  {style=friendly}
  cd .. && mkdir servidores && cd $_ && mkdir respaldos && mkdir masterdb
  mkdir slavedb1 && cd masterdb

  cat <<\EOF > config-file.cnf
# Config Settings Master DB:
[mysqld]
server-id                       = 1
binlog_format                   = ROW
log-bin                         = mysql-bin
key_buffer_size                 = 64M
max_allowed_packet              = 64M
max_connections                 = 210
explicit_defaults_for_timestamp = 1

[mysqldump]
max_allowed_packet = 64M
EOF

cd .. && cd slavedb1

cat <<\EOF > config-file.cnf
# Config Settings Slave DB:
[mysqld]
server-id                       = 101
binlog_format                   = ROW
log-bin                         = mysql-bin
key_buffer_size                 = 64M
max_allowed_packet              = 64M
max_connections                 = 210
explicit_defaults_for_timestamp = 1
#relay-log                       = 

[mysqldump]
max_allowed_packet = 64M

EOF
```

En resumen, nuestro directorio de trabajo debe verse así:
{.mt-4 .mb-4}

```bash {style=friendly}
  cd ~/taller-replicacion && tree

  .
  ├── agomezguru
  │   └── docker-compose.yml
  └── servidores
      ├── masterdb
      │   └── config-file.cnf
      ├── respaldos
      └── slavedb1
          └── config-file.cnf

  5 directories, 3 files
```

con dos volúmenes nombrados:
{.mt-4 .mb-4}

```bash {style=friendly}
  docker volume ls

  DRIVER    VOLUME NAME
  local     agomezguru-masterdb
  local     agomezguru-slavedb
```

Ahora ya podemos iniciar nuestras dos instancias de bases de datos y empezar nuestra práctica, pero antes quiero explicar por qué seleccioné esa configuración para los servidores Maestro y Esclavo. 
{.mt-4 .mb-4}

Como ya dije, emplearemos la replicación basada en filas (*binlog_format*). Entonces, para configurar este tipo de replicación, es obligatorio habilitar los registros binarios (*log-bin*). Asimismo, es necesario configurar los ID en los servidores, tanto en el servidor Maestro como en el(los) servidor(es) Esclavo(s), el cual debe ser único e irrepetible.
{.mb-4}

#### Maestro: Paso #1
{.m-0 .mb-4}

Iniciando los contenedores y revisando que todo esté en orden:
{.mb-4}

```bash {style=friendly}
  cd ~/taller-replicacion/agomezguru

  docker-compose up -d

  [+] Running 8/8
  ⠿ master Pulled                                                               23.3s
  ⠿ slave Pulled                                                                23.3s
    ⠿ 56f7a0abdb77 Pull complete                                                 5.9s
    ⠿ e84fb532f1e3 Pull complete                                                 6.0s
    ⠿ 15377560deef Pull complete                                                 6.3s
    ⠿ 659e56984cf7 Pull complete                                                22.3s
    ⠿ bdc622d377c1 Pull complete                                                22.5s
    ⠿ 753d3189b481 Pull complete                                                22.6s

  [+] Running 3/3
  ⠿ Network agomezguru_agomezguru-network  Created                               0.1s
  ⠿ Container agomezguru-master-1          St...                                 0.8s
  ⠿ Container agomezguru-slave-1           Sta...                                0.9s

  docker-compose ps

  NAME                  COMMAND                  SERVICE   STATUS    PORTS
  agomezguru-master-1   "/docker-entrypoint.…"   master    running   3306/tcp, 33060/tcp
  agomezguru-slave-1    "/docker-entrypoint.…"   slave     running   3306/tcp, 33060/tcp
```

Comprobando conectividad entre contenedores:
{.mt-4 .mb-4}

```bash {style=friendly}
  docker exec -it agomezguru-master-1 ping agomezguru-slave-1

  PING agomezguru-slave-1 (172.19.0.3) 56(84) bytes of data.
  64 bytes from agomezguru-slave-1.agomezguru_agomezguru-network (172.19.0.3): icmp_seq=1 ttl=64 time=0.286 ms
  64 bytes from agomezguru-slave-1.agomezguru_agomezguru-network (172.19.0.3): icmp_seq=2 ttl=64 time=0.121 ms
  64 bytes from agomezguru-slave-1.agomezguru_agomezguru-network (172.19.0.3): icmp_seq=3 ttl=64 time=0.145 ms
  64 bytes from agomezguru-slave-1.agomezguru_agomezguru-network (172.19.0.3): icmp_seq=4 ttl=64 time=0.122 ms
  64 bytes from agomezguru-slave-1.agomezguru_agomezguru-network (172.19.0.3): icmp_seq=5 ttl=64 time=0.132 ms
  ^C
  --- agomezguru-slave-1 ping statistics ---
  5 packets transmitted, 5 received, 0% packet loss, time 4003ms
  rtt min/avg/max/mdev = 0.121/0.161/0.286/0.063 ms

  docker exec -it agomezguru-slave-1 ping agomezguru-master-1

  PING agomezguru-master-1 (172.19.0.2) 56(84) bytes of data.
  64 bytes from agomezguru-master-1.agomezguru_agomezguru-network (172.19.0.2): icmp_seq=1 ttl=64 time=0.155 ms
  64 bytes from agomezguru-master-1.agomezguru_agomezguru-network (172.19.0.2): icmp_seq=2 ttl=64 time=0.094 ms
  64 bytes from agomezguru-master-1.agomezguru_agomezguru-network (172.19.0.2): icmp_seq=3 ttl=64 time=0.089 ms
  64 bytes from agomezguru-master-1.agomezguru_agomezguru-network (172.19.0.2): icmp_seq=4 ttl=64 time=0.125 ms
  64 bytes from agomezguru-master-1.agomezguru_agomezguru-network (172.19.0.2): icmp_seq=5 ttl=64 time=0.082 ms
  ^C
  --- agomezguru-master-1 ping statistics ---
  5 packets transmitted, 5 received, 0% packet loss, time 4000ms
  rtt min/avg/max/mdev = 0.082/0.109/0.155/0.027 ms
```

Accediendo y revisando las tablas del Maestro:
{.mt-4 .mb-4}

```bash {style=friendly}
  $ docker exec -u root -it agomezguru-master-1 bash

  [mysql@657c8d90f1ce /]# mysql -u root -pSinPasswordMaestro

  mysql> show databases;

  +--------------------+
  | Database           |
  +--------------------+
  | information_schema |
  | mysql              |
  | performance_schema |
  | sys                |
  +--------------------+
  4 rows in set (0.01 sec)

  mysql> exit
```

#### Maestro: Paso #2
{.mt-4 .mb-4}

Sin salir de la línea de comandos del contenedor, hagamos una copia de seguridad completa del Maestro usando *mysqldump*.
{.mb-4}

```bash {style=friendly}
  [root@657c8d90f1ce /]# cd respaldos/

  [root@657c8d90f1ce respaldos]# mysqldump -u root -pSinPasswordMaestro --all-databases --source-data=1 --events --routines --triggers --single-transaction > maestro.sql
```

#### Maestro: Paso #3
{.mt-4 .mb-4}

En seguida vamos a crear un usuario para la replicación.
{.mb-4}

```bash {style=friendly}
  mysql -u root -pSinPasswordMaestro

  mysql> CREATE USER agomezguru@'%' IDENTIFIED BY 'SinPasswordBD';

  mysql> GRANT REPLICATION SLAVE ON *.* TO agomezguru@'%';

  mysql> flush privileges;

  mysql> exit

  [root@657c8d90f1ce respaldos]# exit
```

#### Esclavo: Paso #1
{.mt-4 .mb-4}

Restaurar la copia de seguridad del servidor Maestro (Paso #2) en el servidor esclavo.
{.mb-4}

```bash {style=friendly}
  $ docker exec -it agomezguru-slave-1 bash

  [mysql@cb8aa9414f55]$ cd respaldos/

  [mysql@cb8aa9414f55 respaldos]$ mysql -u root -pSinPasswordEsclavo < maestro.sql

  [mysql@cb8aa9414f55 respaldos]$ exit
```

#### Esclavo: Paso #2
{.mt-4 .mb-4}

Con la ejecución de los siguientes comandos vamos a extraer, de entre las primeras 35 líneas del archivo de copia de seguridad, el nombre y la posición del archivo de registro binario.
{.mb-4}

```bash {style=friendly}
  $ cd ~/taller-replicacion/servidores/respaldos

  $ head -n 35 maestro.sql | grep 'MASTER_LOG_FILE'

  CHANGE MASTER TO MASTER_LOG_FILE='mysql-bin.000003', MASTER_LOG_POS=157;
```

#### Esclavo: Paso #3
{.mt-4 .mb-4}

Con estos datos construiremos la siguiente instrucción de MySQL, la cual vamos a ejecutar dentro del contenedor Esclavo. Aquí MASTER_HOST es el nombre de host del servidor Maestro, MASTER_USER y MASTER_PASSWORD son el nombre de usuario y la contraseña que creamos en el *[Maestro: Paso 3](#maestro-paso-3).* MASTER_LOG_FILE y MASTER_LOG_POS son del paso anterior.
{.mb-4}

***Nota:*** Desde MySQL Server 8.0.11 se utiliza como método de autenticación *'caching_sha2_password'*  pero si lo dejas con este valor dará un error y la replicación no iniciará. Para evitarlo voy a forzar el inicio de sesión con el método anterior empleando GET_MASTER_PUBLIC_KEY=1
{.mb-4}

```bash {style=friendly}
  $ docker exec -it agomezguru-slave-1 bash

  [mysql@cb8aa9414f55]$ mysql -u root -pSinPasswordEsclavo

  mysql> CHANGE MASTER TO MASTER_HOST='master',
  MASTER_USER='agomezguru', MASTER_PASSWORD='SinPasswordBD',
  MASTER_LOG_FILE='mysql-bin.000003', MASTER_LOG_POS=157;

  mysql> CHANGE MASTER TO GET_MASTER_PUBLIC_KEY=1;
```

#### Esclavo: Paso #4
{.mt-4 .mb-4}

Ahora vamos a iniciar la replicación verificando su estado:
{.mb-4}

```bash {style=friendly}
  mysql> START SLAVE;

  mysql> SHOW SLAVE STATUS\G;

  *************************** 1. row ***************************
                Slave_IO_State: Waiting for source to send event
                    Master_Host: db
                    Master_User: agomezguru
                    Master_Port: 3306
                  Connect_Retry: 60
                Master_Log_File: mysql-bin.000003
            Read_Master_Log_Pos: 869
                Relay_Log_File: cb8aa9414f55-relay-bin.000002
                  Relay_Log_Pos: 1038
          Relay_Master_Log_File: mysql-bin.000003
              Slave_IO_Running: Yes
              Slave_SQL_Running: Yes
                Replicate_Do_DB: 
            Replicate_Ignore_DB: 
            Replicate_Do_Table: 
        Replicate_Ignore_Table: 
        Replicate_Wild_Do_Table: 
    Replicate_Wild_Ignore_Table: 
                    Last_Errno: 0
                    Last_Error: 
                  Skip_Counter: 0
            Exec_Master_Log_Pos: 869
                Relay_Log_Space: 1255
                Until_Condition: None
                Until_Log_File: 
                  Until_Log_Pos: 0
            Master_SSL_Allowed: No
            Master_SSL_CA_File: 
            Master_SSL_CA_Path: 
                Master_SSL_Cert: 
              Master_SSL_Cipher: 
                Master_SSL_Key: 
          Seconds_Behind_Master: 0
  Master_SSL_Verify_Server_Cert: No
                  Last_IO_Errno: 0
                  Last_IO_Error: 
                Last_SQL_Errno: 0
                Last_SQL_Error: 
    Replicate_Ignore_Server_Ids: 
              Master_Server_Id: 1
                    Master_UUID: 26e33209-d247-11ed-b92f-0242ac130002
              Master_Info_File: mysql.slave_master_info
                      SQL_Delay: 0
            SQL_Remaining_Delay: NULL
        Slave_SQL_Running_State: Replica has read all relay log; waiting for more updates
            Master_Retry_Count: 86400
                    Master_Bind: 
        Last_IO_Error_Timestamp: 
      Last_SQL_Error_Timestamp: 
                Master_SSL_Crl: 
            Master_SSL_Crlpath: 
            Retrieved_Gtid_Set: 
              Executed_Gtid_Set: 
                  Auto_Position: 0
          Replicate_Rewrite_DB: 
                  Channel_Name: 
            Master_TLS_Version: 
        Master_public_key_path: 
          Get_master_public_key: 1
              Network_Namespace: 
  1 row in set, 1 warning (0.01 sec)

  mysql> exit

  [mysql@cb8aa9414f55 /]$ exit
```

Para verificar que la replicación está funcionado bien, revisa que se muestren los siguientes valores:
{.mt-4 .mb-4}

```bash {style=friendly}
  Slave_IO_Running: Yes
  Slave_SQL_Running: Yes
  Seconds_Behind_Master: 0
```

Si ambos valores **Slave_IO_Running** y **Slave_SQL_Running** son **Yes**, entonces está replicando correctamente. Si alguno de los valores es "No", revisa todos los pasos cuidadosamente.
{.mb-4}

Si el valor de **Seconds_Behind_Master** es **0** significa que la replicación en el Esclavo está sincronizado con su Maestro. Si el valor es **10**, significa que el Esclavo está 10 segundos por detrás de su Maestro.
{.mb-4}

#### Esclavo: Paso #5
{.mt-4 .mb-4}

Ya que hemos comprobado que todo está en orden, solo nos falta el último paso, configurar el servidor Esclavo para que, si necesitas interrumpir la ejecución de tus contenedores, cuando los vuelvas a iniciar reinicie la replicación sin contratiempos. Para ello vamos a quitar el comentario de **#relay-log** y configuraremos el valor devuelto por **Relay_Log_File** en el paso anterior:
{.mb-4}

```bash {hl_Lines=6, style=friendly}
  $ cd ~/taller-replicacion/servidores/slavedb1

  $ vim config-file.cnf

  relay-log = cb8aa9414f55-relay-bin
```

### Verificación final del arreglo maestro-esclavo
{.blog-desc-big .m-0 .mt-4 .mb-4}

Con esto hemos concluido satisfactoriamente la configuración del arreglo. Solo para confirmar que efectivamente está replicando en forma correcta, voy a crear una tabla en el Maestro con algún dato y los cambios se deben de reflejar en el Esclavo.
{.mb-4}

```bash {style=friendly}
  $ docker exec -it agomezguru-master-1 bash
  [mysql@657c8d90f1ce /]$ mysql -u root -pSinPasswordMaestro

  mysql> CREATE SCHEMA taller_replicacion;
  mysql> use taller_replicacion;
  mysql> CREATE TABLE IF NOT EXISTS mi_test (
      id INT AUTO_INCREMENT PRIMARY KEY,
      title VARCHAR(255) NOT NULL,
      description TEXT,
      created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
  )  ENGINE=INNODB;

  mysql> INSERT INTO mi_test(title, description)
  VALUES("Aprendiendo a replicar",
  "Taller teórico práctico");

  mysql> exit
  [mysql@657c8d90f1ce /]$ exit

  $ docker exec -it agomezguru-slave-1 bash

  [mysql@cb8aa9414f55]$ mysql -u agomezguru -pSinPasswordMaestro
  mysql> show databases;

  +--------------------+
  | Database           |
  +--------------------+
  | information_schema |
  | mysql              |
  | performance_schema |
  | sys                |
  | taller_replicacion |
  +--------------------+
  5 rows in set (0.01 sec)

  mysql> use taller_replicacion

  mysql> show tables;

  +------------------------------+
  | Tables_in_taller_replicacion |
  +------------------------------+
  | mi_test                      |
  +------------------------------+
  1 row in set (0.00 sec)

  mysql> describe mi_test;

  +-------------+--------------+------+-----+-------------------+-------------------+
  | Field       | Type         | Null | Key | Default           | Extra             |
  +-------------+--------------+------+-----+-------------------+-------------------+
  | id          | int          | NO   | PRI | NULL              | auto_increment    |
  | title       | varchar(255) | NO   |     | NULL              |                   |
  | description | text         | YES  |     | NULL              |                   |
  | created_at  | timestamp    | YES  |     | CURRENT_TIMESTAMP | DEFAULT_GENERATED |
  +-------------+--------------+------+-----+-------------------+-------------------+
  4 rows in set (0.00 sec)

  mysql> select * from mi_test;

  +----+------------------------+---------------------------+---------------------+
  | id | title                  | description               | created_at          |
  +----+------------------------+---------------------------+---------------------+
  |  1 | Aprendiendo a replicar | Taller teórico práctico | 2023-04-06 14:48:09 |
  +----+------------------------+---------------------------+---------------------+
  1 row in set (0.00 sec)

  mysql> exit
  [mysql@657c8d90f1ce /]$ exit
```

Si eres observador, habrás notado que al introducir la contraseña del servidor Esclavo ya no utilicé la configurada inicialmente en el archivo docker-composer.yml, sino la misma del servidor Maestro, esto es así porque al sacar una copia completa en el [Maestro: Paso #2](#maestro-paso-2) también clonó su contraseña.
{.mt-4}

### Conclusión
{.blog-desc-big .m-0 .mt-4 .mb-4}

La replicación maestro-esclavo mediante filas empleando logs binarios, es un método muy eficiente que, sumado a un despliegue típico empleando contenedores Docker, nos permite desplegar una infraestructura poco costosa en recursos informáticos y en consecuencia muy *ad-hoc* para su empleo en proyectos de largo alcance, que debidamente combinado con otras tecnologías que iremos descubriendo juntos a lo largo de este blog, nos permite alcanzar niveles en los acuerdos de servicio (SLA's) cercanos al 99.98%.
{.mb-4}

### Referencias
{.blog-desc-big .m-0 .mt-4 .mb-4}

{{< list >}}

- [Configure Master-Slave Replication in MySQL/MariaDB/Percona](https://kapilyadavalli.wordpress.com/2020/03/26/configure-a-simple-master-slave-replication-in-mysql-mariadb-percona/)

- [Setting up Basic Master-Slave Replication in MySQL 8](https://webyog.com/blog/monyog/setting-basic-master-slave-replication-mysql-8/)

- [Play with Docker](https://labs.play-with-docker.com/)

- [DockerHub](https://hub.docker.com/)
{{< /list >}}

### Créditos
{.blog-desc-big .m-0 .mt-4 .mb-4}

{{< list gray >}}

- Figura 1. Trigger-based replication | From Wikimedia Commons, the free media repository

- Figura 2. Log-based replication | From Wikimedia Commons, the free media repository
{{< /list >}}
