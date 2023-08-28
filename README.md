# poc-kafka-connect-worker

![Alt text](image-1.png)

# Kafka Connect

    - "Kafka Connect é um componente gratuito e open-source do Apache Kafka que trabalha com hub de dados, centralizando para integrações simples entre banco de dados, key-value stores, search indexes e file systems"

    - link documentation: https://docs.confluent.io/platform/current/connect/index.html

# Kafka Converters

    - "As tasks utilizam os 'converters' para mudar o formato dos dados tanto para leitura ou escrita no kafka"

    - Documentation Converters: https://docs.confluent.io/platform/current/connect/index.html#connect-converters

    - Formatos de dados aceitos pelo kafka para converter

        - AvroConverter
        - ProtobufConverter
        - JsonSchemaConverter
        - JsonConverter
        - StringConverter
        - ByteArrayConverter

![Alt text](image.png)

# Kafka DLQ - Deat Letter Queue

    - Documentation Deat Letter Queue: https://docs.confluent.io/platform/current/connect/index.html#dead-letter-queue

    - "Quando há um registro inválido, independente da razão, o erro pode ser tratado nas configurações do conector através da propriedade "erros.tolerance". Esse tipo de configuração pode ser realizado apenas para conectores do tipo 'Skin'".

    - Existem dois valores válidos para errors.tolerance:

        - none( padrão ) - Faz a tarefa falhar imediatamente. Deve ser consultado os logs para entender o problema ocorrido.
        - all - Erros são ignorados e o processo continua normalmente.

    - Criar um Topico da fila para as Deat Letter Queue

        - Assim os sistema não se compromete em parar, ele armazena os dados nessa fila com as mensagens de erro.
        - Os erro são informados no header da mensagem.
        - Assim fica mais estrategico para recuperar as mensagens.

        - errors.tolerance = all
        - errors.deadletterqueue.topic.name = <dead-letter-topic-name>

# Kafka Connectors Hub

    - O link a baixo permite acessar e escolher os tipos de conectore ao kafka
    - Documentation: https://www.confluent.io/hub/

    - Documentation database de exemplo de conector com mysql: https://www.confluent.io/hub/debezium/debezium-connector-mysql
    - Documentation database de exemplo de conector com mongo: https://www.confluent.io/hub/debezium/debezium-connector-mongodb

# Acessando ao control-center do confluence

    - Assim que os containers estiver no ar

        - http://localhost:9021

# Configurando MySQL

    - Para saber o nome do container que está executando o MySQL: docker ps

    - Entrar no banco de dados do MySQL: docker exec -it poc-kafka-connect-worker-mysql-1 bash

        - Dentro do container MySQL docker.

        - Comando para acessar ao Banco do MySQL: mysql -uroot -p db_fullcycle_mysql

            - Em seguida digite a senha do MySQL: root.

            - Consultar tabela no banco do MySQL: show tables;

            - Criar tabela no banco do MySQL: create table categories (id int auto_increment primary key, name varchar(255) not null);
                - Resposta: Query OK, 0 rows affected (0.03 sec)

            - Inserir dados na tabela de categories banco do MySQL: Insert into categories (name) values ('Eletronicos');

            - Select na tabelade de categories banco do MySQL: select * from categories;

# Configurando conector do MySQL

    - Documentação de conector kafka connect com MySQL: https://debezium.io/documentation/reference/2.3/connectors/mysql.html

    - Criando as configurações do connectores.

        - raiz do projeto: connectors/mysql.properties
            - Criar uma classe de cofiguração de conexão.

        # Criando uma worker standalone
        # driver de connector mysql
            connector.class=io.debezium.connector.mysql.MySqlConnector
        # O kafka connect criará um topic de historico
            database.history.kafka.topic=mysql_history

    - Com o container do kafka connect no online.
        - Acesse o caminho para fazer upload da configuração da classe do kafka connect: CONNECT CLUSTERS / CONNECT-DEFAULT/ CONNECT

            - Deve ser feito o upload do connectors/mysql.properties, que está no projeto.
            - Assim que o arquivo o connectors/mysql.properties, intergrar ao upload.
                - Clique> Continuar
                - Depois em: Lauch
                - Os Connectores devem estar running
                - Consulte os Topics: Deve ser criado topicos de instancias do mysql
                    - Topic: mysql-server.bd_fullcycle_mysql.categories = Defina Partição= 0
                        - Será monstrado as mensagens
                        - Consulte o mensagens que foi criada:
                                "payload": {
                                    "before": null,
                                    "after": {
                                    "id": 1,
                                    "name": "Eletronicos"
                                    },
                                    "source": {
                                    "version": "1.2.2.Final",
                                    "connector": "mysql",
                                    "name": "mysql-server",
                                    "ts_ms": 0,
                                    "snapshot": "last",
                                    "db": "bd_fullcycle_mysql",
                                    "table": "categories",
                                    "server_id": 0,
                                    "gtid": null,
                                    "file": "mysql-bin.000003",
                                    "pos": 788,
                                    "row": 0,
                                    "thread": null,
                                    "query": null
                                    },
                                    "op": "c",
                                    "ts_ms": 1692904752787,
                                    "transaction": null
                                }

            - Adicionando mais duas categorias ao banco de dados.

                mysql> Insert into categories (name) values ('Cozinha');
                Query OK, 1 row affected (0.01 sec)

                mysql> Insert into categories (name) values ('Cama Mesa e Banho');

            - O kafka connect está recebendo os dados sincronizado com Topic

# Persistindo dados no MongoDb com conector

    - conficguração do MongoDB: mongodb.properties

        - O name está recebendo o nome de sink, para que seja sincronizado com MySQL exemplo.
        - Driver de conexao: com.mongodb.kafka.connect.MongoSinkConnector
        - Nome da base de datos que será percisitido os dados: database=bd_fullcycle_mongo
        - Adicionando o transforme value para pegar somente o necessario que vem do pauload: transforms=extractValue
        - Informe o caminho do payload que vai utilizar: transforms.extractValue.type=org.apache.kafka.connect.transforms.ExtractField$Value
        - Informe o valor da value que está no payload que irá utilizar: transforms.extractValue.field=after
            - Documentação trantando o transforme: https://docs.confluent.io/platform/current/connect/transforms/extractfield.html

                name=mongo-sink-from-mysql
                connector.class=com.mongodb.kafka.connect.MongoSinkConnector
                task.max=1
                topics=mysql-server.db_fullcycle_mysql.categories
                connection.uri=mongodb://root:root@mongodb/
                database=bd_fullcycle_mongo
                transforms=extractValue
                transforms.extractValue.type=org.apache.kafka.connect.transforms.ExtractField$Value
                transforms.extractValue.field=after

    - Acessando o MongoDB Express: http://localhost:8085

# Kafka Confluent Cloud

    - document: https://www.confluent.io/confluent-cloud/

        - Crie uma conta free, para seu cluster para estudos em desenvolvimento

        - Cluster criado um cluster para fins de teste de desenvolvimento: https://confluent.cloud/environments/env-5wgp3q/clusters

# Apache Kafka na AWS

    - Criando cluster para strteam de mensage de test para desenvolvimento.
