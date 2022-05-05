# Istio Service Mesh practices

## Propagate trace headers
Although Istio proxies are able to automatically send spans, they need some hints to tie together the entire trace. Applications need to propagate the appropriate HTTP headers so that when the proxies send span information, the spans can be correlated correctly into a single trace.

To do this, an application needs to collect and propagate the following headers from the incoming request to any outgoing requests:
```basic
x-request-id
x-b3-traceid
x-b3-spanid
x-b3-parentspanid
x-b3-sampled
x-b3-flags
x-ot-span-context
```
Below is a list of the most common trace context standards:
- [B3](https://github.com/openzipkin/b3-propagation)
- [W3C](https://www.w3.org/TR/trace-context/)

### Spring Boot
When using Spring Boot, [Spring Cloud Sleuth](https://spring.io/projects/spring-cloud-sleuth#overview) can be used to configure distributed tracing with minimal effort.

When Spring Cloud Sleuth is added to your Spring Boot application, the following parameters are 
- `spring.sleuth.propagation-keys`: Used for propagating additional headers that are not part of the trace context standard. For example `x-request-id`.
- `spring.sleuth.propagation.type`: Specify a trace context standard. Options are `AWS` `B3` `W3C`. (`B3` is the default)

## Enable excluded resources in Kiali
By default the following resources are excluded from Kiali:
- CronJob
- DeploymentConfig
- Job
- ReplicationController
- StatefulSet

In order to enable any of the excluded resources above, follow this [link](https://access.redhat.com/solutions/5359141)

## Appy necessary labels to pods
For each pods some labels need to be configured in order for Istio to work correctly.
Each pod requires the `app` and `version` labels

The `app` label: Each deployment should have a distinct app label with a meaningful value. The app label is used to add contextual information in distributed tracing.

The `version` label: This label indicates the version of the application corresponding to the particular deployment.

Additional documentation [here](https://istio.io/latest/docs/ops/deployment/requirements/)

## Specify port protocols to services
By specifying port protocols in a service, Istio is able to determine the traffic type and provide additional capabilities.
This can be done by one of two ways:
- By the name of the port: name: `<protocol>[-<suffix>]`.
- In Kubernetes 1.18+, by the appProtocol field: `appProtocol: <protocol>`.

```yaml
kind: Service
metadata:
  name: myservice
spec:
  ports:
  - number: 3306
    name: database
    appProtocol: https
  - number: 80
    name: http-web
```

Additional documentation [here](https://istio.io/latest/docs/ops/deployment/requirements/)
