<!-- Fill key.json with key having access to Google big query  -->

docker-compose up -d

docker-compose exec broker kafka-topics \
--create \
--bootstrap-server localhost:9092 \
--replication-factor 1 \
--partitions 1 \
--topic test \
--config confluent.value.subject.name.strategy=io.confluent.kafka.serializers.subject.RecordNameStrategy

curl -X PUT -H "Content-Type: application/json" \
 --data '{"compatibility": "NONE"}' \
 http://localhost:8081/config/test-value

curl -X POST -H "Content-Type: application/vnd.schemaregistry.v1+json" http://localhost:8081/subjects/test.WithArray/versions -d @schema-with-array.json
curl -X POST -H "Content-Type: application/vnd.schemaregistry.v1+json" http://localhost:8081/subjects/test.WithoutArray/versions -d @schema-without-array.json

curl -X POST -H "Content-Type: application/vnd.schemaregistry.v1+json" \
--data "{\"schema\": $(curl -s http://localhost:8081/subjects/Kafka1-value/versions/latest | jq '.schema')}" \
http://localhost:8081/subjects/Kafka2-value/versions

docker-compose exec schema-registry kafka-avro-console-producer \
 --broker-list broker:29092 \
 --topic test \
 --property schema.registry.url=http://localhost:8081 \
 --property value.schema.id=1 \
 --producer-property value.subject.name.strategy=io.confluent.kafka.serializers.subject.RecordNameStrategy

{"normal_field": "123", "my_array": ["a", "b"]}

docker-compose exec schema-registry kafka-avro-console-producer \
 --broker-list broker:29092 \
 --topic test \
 --property schema.registry.url=http://localhost:8081 \
 --property value.schema.id=2 \
 --producer-property value.subject.name.strategy=io.confluent.kafka.serializers.subject.RecordNameStrategy

{"normal_field": "456"}

curl -i -X POST -H "Accept:application/json" -H "Content-Type:application/json" http://localhost:8083/connectors/ -d @connector.json
