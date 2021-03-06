# Eventing-Kafka config

These yaml files install Eventing-Kafka into a cluster which must already have Knative-Eventing installed and have a Kafka
cluster available. The values of the [Kafka Secret](300-kafka-secret.yaml) should be correctly populated. The yaml files
use ko references, and must be installed using ko. `ko apply -f ./config` can be executed here to build and deploy the
project.

## Kafka Providers

Eventing-Kafka supports either a standard Kafka deployment or Azure EventHubs. In order to configure the desired provider,
set the eventing-kafka.kafka.provider field in the [config-eventing-kafka configmap](200-eventing-kafka-configmap.yaml)

The current allowed values are:

* `local`: Standard Kafka installation with no special authorization required (default)
* `confluent`: Confluent Cloud
* `azure`: Azure Event Hubs

The provider chosen effects how authentication as well as admin calls (topic creation, deletion etc) work.

## Credentials

### Install & Label Kafka Credentials In Knative-Eventing Namespace

Eventing-kafka depends on secrets labeled with `eventing-kafka.knative.dev/kafka-secret="true"`, multiple
secrets are supported for the use of the `azure` integration, representing different EventHubs namespaces.  Some fields
may not apply to your particular Kafka implementation and can be left blank. A [sample secret](300-kafka-secret.yaml)
is included in the config directory, but must be modified to hold real values.

Example values for Azure Event Hubs (must be base64 encoded):

```
  namespace: my-cluster-name-1
  brokers: my-cluster-name-1.servicebus.windows.net:9093
  password: Endpoint=sb://my-cluster-name-1.servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX=
  username: $ConnectionString
```

Example values for Confluent Cloud (must be base64 encoded):

```
  brokers: SASL_SSL://my-cluster.eu-west-1.aws.confluent.cloud:9092
  password: XVLEXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
  username: KIELSXXXXXXXXXXX
```

Alternatively, or if you need to specify the broker secret(s) after installation, they may also be created manually:

```
# Example A Creating A Kafka Secret In Knative-Eventing
kubectl create secret -n knative-eventing generic kafka-credentials \
    --from-literal=brokers=<BROKER CONNECTION STRING> \
    --from-literal=username=<USERNAME> \
    --from-literal=password=<PASSWORD> \
    --from-literal=namespace=<AZURE EVENTHUBS NAMESPACE> \
kubectl label secret -n knative-eventing kafka-credentials eventing-kafka.knative.dev/kafka-secret="true"
```
