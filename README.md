# kafka-connect-file
This is step by step demo for kafka connect for file data source.


STEP 1:  Download the kafka.
    Download the kafka from https://www.apache.org/dyn/closer.cgi?path=/kafka/3.0.0/kafka_2.13-3.0.0.tgz and extract it as given below.
   
    tar -xzf kafka_2.13-3.0.0.tgz
    cd kafka_2.13-3.0.0
    export KAFKA_HOME=`pwd`

STEP 2: START THE KAFKA ENVIRONMENT
    
    a)Configure the zookeeper property file which is present in $KAFKA_HOME/config zookeeper.properties
      
        dataDir=/tmp/zookeeper
        clientPort=2181
        maxClientCnxns=0
        admin.enableServer=false

    b) Start the zookeeper now.
        $KAFKA_HOME/bin/zookeeper-server-start.sh $KAFKA_HOME/config/zookeeper.properties
    
    c) Make sure to change the following proerties in your $KAFKA_HOME/config/server.properties
        listeners=PLAINTEXT://localhost:9092
        advertised.listeners=PLAINTEXT://localhost:9092

    d) Start borker
        $KAFKA_HOME/bin/kafka-server-start.sh $KAFKA_HOME/config/server.properties

STEP 3: Make the directory for standalone file connect.
    
    mkdir kafka_connect_standalone_file_poc
    cd kafka_connect_standalone_file_poc
    export KAFKA_CONNECT_POC=`pwd`

    a) Make input source file dircetory
        mkdir $KAFKA_CONNECT_POC/input_data
        mkdir $KAFKA_CONNECT_POC/offset
    
    b) create connector file.
        vi standalone_file_connector.properties
        name=standalone-connect-file
        connector.class=FileStreamSource
        tasks.max=1
        #file=$KAFKA_CONNECT_POC/input_data/input_file_name
        file=/app/hadoop_users/MNAAS/kafka/data/input.txt
        topic=standalone-connect-file

    b)  configure standalone worker poperty files as given below.
    
        vi $KAFKA_CONNECT_POC/standalone-worker-file.properties
    
        bootstrap.servers=localhost:9092

        #Converter configuration
        key.converter=org.apache.kafka.connect.json.JsonConverter
        key.converter.schemas.enable=false
        value.converter=org.apache.kafka.connect.json.JsonConverter
        value.converter.schemas.enable=false

        # internal key to JsonConverter
        internal.key.converter=org.apache.kafka.connect.json.JsonConverter
        internal.key.converter.schemas.enable=false
        internal.value.converter=org.apache.kafka.connect.json.JsonConverter
        internal.value.converter.schemas.enable=false

        # Rest API
        rest.port=8083
        rest.host.name=localhost

        # Config is only for standalone workers
        #offset.storage.file.filename=$KAFKA_CONNECT_POC/offset/standalone_connect_file_offset
        offset.storage.file.filename=standalone_connect_file_offset
        offset.flush.interval.ms=10000

STEP 4: Start kafka connect

    $KAFKA_HOME/bin/connect-standalone.sh \
    $KAFKA_CONNECT_POC/standalone-worker-file.properties \
    $KAFKA_CONNECT_POC/standalone_file_connector.properties

STEP 5: Read the data from topic to verify it.

    $KAFKA_HOME/bin/kafka-console-consumer.sh \
        --bootstrap-server localhost:9092 \
        --topic standalone-connect-file \
        --from-beginning
