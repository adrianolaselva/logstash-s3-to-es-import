
# Importador de transações PayGo (Logstash)


### Variáveis de ambiente


```env

    INSTANCE_NAME=[Nome da instância do docker padrão "logstash${INSTANCE_NAME}" (ex: "_instancia_01")]
    LS_DEBUG=[Habilita e desabilita o modo debugger (ex: true/false)]
    
    LS_JAVA_OPTS=[memória da JVM para execução do logstash (ex: "-Xmx2g -Xms2g")]
    LS_HEAP_SIZE=[Tamanho da memória heap (ex: "2g")]
    
    DEPLOY_RESOURCES_LIMITS_CPUS=[Limite de cpu do container (ex: "4.0")]
    DEPLOY_RESOURCES_LIMITS_MEMORY=[Limite de memória do container (ex: "6g")]
    
    ES_URL=[URL elasticsearch (ex: "https://localhost:9200")]
    ES_USER=[usuário elasticsearch (ex: "elastic")]
    ES_PWD=[senha elasticsearch (ex: "changeme")]
    ES_DOCUMENT_TYPE=[Document type elasticsearch]
    
    S3_ACCESS_KEY_ID=[access_key aws]
    S3_SECRET_ACCESS_KEY=[secret_access_key aws]
    S3_REGION=[region aws (ex: "sa-east-1")]
    S3_BUCKET=[bucket s3  (ex: "bucket_01")]
    S3_TO_BUCKET=[bucket backup (ex: "bucket_01_bkp")]
    S3_PREFIX=[prefixo do bucket a ser importado (ex: "arquivos/a_importar")]
    S3_BACKUP=[habilita/desabilita backup (ex: true/false)]
    
    SINCEDB_PATH=[arquivo com referência da importação (ex: "/usr/share/logstash/pipeline/.sincedb")]
    SERVIDOR_DIR=[diretório com configurações de estabelecimentos para atachar (ex: "../servidor")]

```

### Plugins

- logstash-output-jdbc


### Inicializar container

```sh
    docker-compose up --build -d
```

### Script de importação


##### O script com as configurações de importação deve ser adicionado no diretório "./pipeline/*.conf" ex: 

```logstash

input 
{
      s3 {
          bucket => "${S3_BUCKET}"
          access_key_id  => "${S3_ACCESS_KEY_ID}"
          secret_access_key => "${S3_SECRET_ACCESS_KEY}"
          region => "${S3_REGION}"
          prefix => "${S3_PREFIX}"
          backup_to_bucket => "${S3_TO_BUCKET}"
          delete => false
          sincedb_path => "${SINCEDB_PATH}"
      }
}

filter 
{
    mutate {
        add_field => {
            "[@metadata][debug]" => "${LS_DEBUG:false}"
        }
        add_field => {
            "[@metadata][indexDate]" => "%{+YYYYMMdd}"
        }
    }
    ruby {
        code => "require 'digest/md5';
        event.set('[@metadata][computed_id]', Digest::MD5.hexdigest(event.get('[message]')))"
    }

    csv {}
}

output {

    if [@metadata][debug] == "true" {
        stdout { codec => rubydebug }
    } else {
        elasticsearch {
            action => "index"
            hosts => "${ES_URL}"
            index => "${ES_INDEX}%{[@metadata][indexDate]}"
    	    user => "${ES_USER}"
    	    document_type => "${ES_DOCUMENT_TYPE}"
    	    password => "${ES_PWD}"
    	    document_id => "%{[@metadata][computed_id]}"
        }
    }
}

```



