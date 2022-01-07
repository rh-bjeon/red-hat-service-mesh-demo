# Red Hat Service Mesh Demo

This repository contains the required procedure to deploy an application in Openshift using service integration. The idea of this demo is show the Red Hat Service Mesh features managing traffic flow.

## Prerequisites

- A Red Hat Openshift Cluster 4.8+
- OC Client 4.8+
- Web browser

## Setting Up

- Install Red Hat Service Mesh, Kiali and Red Hat OpenShift distributed tracing platform operators (*Please review the istio, jaeger and kiali pods are up and running*)

```$bash
oc apply -f 00-service-mesh-operators.yaml
oc get po -n openshift-operators -w
```

- Deploy the Red Hat Service Mesh Control Plane - SMCP (*Please review the istiod, prometheus, grafana, jaeger and kiali pods are up and running*)

```$bash
oc new-project istio-system
oc apply -f 01-service-mesh-control-plane.yaml
oc get po -n istio-system -w
```

- Define the Red Hat Service Mesh Member Roll (SMMR)

```$bash
oc new-project jump-app
oc apply -f 02-service-mesh-members-roll.yaml
```

- Deploy the application

```$bash
oc apply -n jump-app -f 03-jump-app-deploy.yaml
```

NOTE: It is required the correct Openshift applications domain is defined in every object properly 

- Test the application via Browser and Kiali (_Please find the URL executing the following command_)

```$bash
oc -n istio-system get route back-golang-istio-system -o jsonpath='{.spec.host}'
oc -n istio-system get route front-javascript-istio-system -o jsonpath='{.spec.host}'
oc -n istio-system get route kiali -o jsonpath='{.spec.host}'
```

## Deployment Strategies

### B/G

- Migrate from B deployment to G

```$bash
oc apply -n jump-app -f 04-jump-app-deploy-blue-green.yaml
```

### Canary

- Steps 1 (90% A / 10% B)

```$bash
oc apply -n jump-app -f 05-jump-app-deploy-canary-step1.yaml
```

- Step 2 (60% A / 40% B)

```$bash
oc apply -n jump-app -f 05-jump-app-deploy-canary-step2.yaml
```

- Step 3 (20% A / 80% B)

```$bash
oc apply -n jump-app -f 05-jump-app-deploy-canary-step3.yaml
```

- Step 3 (0% A / 100% B)

```$bash
oc apply -n jump-app -f 05-jump-app-deploy-canary-step4.yaml
```

### Mirror

- Mirror 100% traffic between versions in Python microservice

```$bash
oc apply -n jump-app -f 06-jump-app-deploy-mirror.yaml
```

## Chaos Monkey

### Fault Injection 

In this scenario, it is introduced a 3 seconds delay in every service Python request will be checked through the Kiali console.

```$bash
oc apply -n jump-app -f 07-jump-app-chaos-fault-injection.yaml
```

### Timeout

In this scenario, it is introduced a fault injection to generate delays in Python service requests and defined a connection timeout in the previous microservice, Springboot. The idea is to generate a certain number of errors due to the delay and the timeout configured.

Once the configuration is applied, it is possible to check the current state of the connection requests between services through the Kiali console.

```$bash
oc apply -n jump-app -f 08-jump-app-chaos-timeout.yaml
```

### Circuit Breaking

In this scenario, it is introduced a circuit breaking rule in order to avoid sending connections to an unavailable service. The idea is to configure the Python destination rule in order to be able to detect errors and avoid sending requests to a this failed service.

```$bash
oc apply -n jump-app -f 09-jump-app-chaos-circuit-breaking-base.yaml
oc scale -n jump-app deploy back-python-v1 --replicas=1
oc apply -n jump-app -f 09-jump-app-chaos-circuit-breaking.yaml
```

NOTE: It is required to open three or more browser with the frontend service in order to send multiple concurrent connections.

## Author

Asier Cidon @RedHat
