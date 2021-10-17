# kafka-connect-file
This is step by step demo for kafka connect for file data source.

STEP 1:  Download the kafka.
    Download the latest Kafka release and extract it
    
    tar -xzf kafka_2.13-3.0.0.tgz
    cd kafka_2.13-3.0.0
    export KAFKA_HOME=`pwd`
    mkdir $KAFKA_HOME/tmp

STEP 2: START THE KAFKA ENVIRONMENT
    
    a)Configure the zookeeper property file which is present in $KAFKA_HOME/config zookeeper.properties
      
        dataDir=$KAFKA_HOME/tmp/zookeeper
        clientPort=2181
        maxClientCnxns=0
        admin.enableServer=false

    b) Start the zookeeper now.
        bin/zookeeper-server-start.sh config/zookeeper.properties
    
    c) Configure the broker. 

        broker.id=0
        listeners=PLAINTEXT://localhost:29092
        advertised.listeners=PLAINTEXT://localhost:29092
        num.network.threads=3
        num.io.threads=8
        socket.send.buffer.bytes=102400
        socket.receive.buffer.bytes=102400
        socket.request.max.bytes=104857600

        #Give the full path of kafka home
        log.dirs=<KAFKA_HOME>/tmp/kafka-logs
        
        num.partitions=1
        num.recovery.threads.per.data.dir=1
        offsets.topic.replication.factor=1
        transaction.state.log.replication.factor=1
        transaction.state.log.min.isr=1
        log.retention.hours=168
        log.segment.bytes=1073741824
        log.retention.check.interval.ms=300000
        zookeeper.connect=localhost:22181
        zookeeper.connection.timeout.ms=6000
        group.initial.rebalance.delay.ms=0

    d) Start borker
        bin/kafka-server-start.sh config/server.properties

STEP 3: Make the directory for standalone file connect.
    
    mkdir kafka_connect_standalone_file_poc
    cd kafka_connect_standalone_file_poc
    export KAFKA_CONNECT_POC=`pwd`

    a) Make input source file dircetory
        mkdir $KAFKA_CONNECT_POC/input_data
    
    b) create connector file.
        vi standalone_file_connector.properties
        name=standalone-connect-file
        connector.class=FileStreamSource
        tasks.max=1
        file=/app/hadoop_users/MNAAS/kafka/data/input.txt
        topic=standalone-connect-file

    b)  configure standalone worker poperty files as given below.

    vi $KAFKA_CONNECT_POC/standalone-worker-file.properties

    bootstrap.servers=localhost:29092
    
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
    rest.port=28086
    rest.host.name=mtmdevhdoped01
    
    # Config is only for standalone workers
    offset.storage.file.filename=standalone.file.source.offsets
    offset.flush.interval.ms=10000

STEP 4: Start kafka connect

    $KAFKA_HOME/bin/connect-standalone.sh \
    $KAFKA_CONNECT_POC/standalone-worker-file.properties \
    $KAFKA_CONNECT_POC/standalone_file_connector.properties

STEP 5: Read the data from topic to verify it.

    $KAFKA_HOME/bin/kafka-console-consumer.sh \
        --bootstrap-server localhost:29092 \
        --topic standalone-connect-file \
        --from-beginning
