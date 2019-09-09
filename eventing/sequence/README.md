## Prerequisite
kafka broker is deployed using [steps](https://github.com/qiujian16/knative-practice/blob/master/eventing/kafka-broker/README.md)

## Configure a squence eventing
1. Create a set of squence [services](https://github.com/qiujian16/knative-practice/blob/master/eventing/sequence/services.yaml)
2. Create a [sequence]https://github.com/qiujian16/knative-practice/blob/master/eventing/sequence/sequence.yaml
3. Create [sequence trigger](https://github.com/qiujian16/knative-practice/blob/master/eventing/sequence/sequence-trigger.yaml)
4. Create event-display serivce
```
cd $GOPATH/src/knative.dev/eventing-contribe/config/tools
ko apply -f event-display
```
5. Create [display trigger](https://github.com/qiujian16/knative-practice/blob/master/eventing/sequence/display-trigger.yaml)

## Verify the setting
1. Get kafka broker
```
kubectl get service kafka-broker
```
2. issue a request
```
curl -H "Host: kafka-broker.default.svc.cluster.local" http://{broker IP} -X POST   -H "X-B3-Flags: 1"   -H "CE-SpecVersion: 0.2"   -H "CE-Type: dev.knative.foo.bar"   -H "CE-Time: 2018-04-05T03:56:24Z"   -H "CE-ID: 45a8b444-3213-4758-be3f-540bf93f85ff"   -H "CE-Source: dev.knative.example"   -H 'Content-Type: application/json'   -d '{ "message": "lol" }'
```
