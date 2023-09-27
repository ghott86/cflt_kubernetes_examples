# Confluent for Kubernetes Examples on OpenShift

Official [Confluent for Kubernetes documentation](https://docs.confluent.io/operator/current/overview.html).

Confluent Component Images available here: https://hub.docker.com/u/confluentinc

Watch the walkthrough: https://youtu.be/qepFNPhrL08

## Prerequisites

The following prerequisites are assumed for all secnario workflows:

* A Kubernetes cluster - any CNCF conformant version
* Helm 3 installed on your local machine
* Kubectl installed on your local machine
* A namespace created in the Kubernetes cluster - `confluent`
* Kubectl configured to target the `confluent` namespace:
  ```
  kubectl config set-context --current --namespace=confluent
  ```
* This repo cloned to your workstation:
  ```
  git clone git@github.com:ghott86/cflt_kubernetes_examples.git
  ```

Note: There are several ways to provision Confluent Platform (security, no security, auto generated certs, etc). The quickest way to prototype with services is to setup with no security first.

If required, you will need to make changes to the image locations, openshift domain, and namespaces, etc.

---
---

# Security considerations for Red Hat Openshift

## Pod Security

CFK, by default, configures the Pod Security Context to run as non-root with a UID and GUID `1001`.

In Red Hat OpenShift, by default Security Context Constraint (SCC) does not allow a pod to run with the above security context.

You've got two options to align with the Security Context Constraint (SCC) needs on Red Hat OpenShift:

1) Recommended: Deploy with default SCC policy
2) Advanced: Deploy with a custom SCC policy

For the quick start below, we'll use option 1, default SCC policy.

In every Confluent component CustomResource, the `podSecurityContext` attribute is added; example below:

```
vi $TUTORIAL_HOME/cflt_examples/security/openshift-security/confluent-platform-with-defaultSCC.yaml
...
spec:
  podTemplate:
    podSecurityContext: {} # Disable the custom pod security context, to use the default
...
```

---
---

# QuickStart

## OpenShift QuickStart Deployment Steps: 3 Node w/ Default SCC:

1. Create the namespace to use.
    ```
    kubectl create namespace confluent
    ```
2. Set this namespace to default for your Kubernetes context.
     ```
     kubectl config set-context --current --namespace confluent
     ```
3. Set up the Helm Chart:
     ```
     helm repo add confluentinc https://packages.confluent.io/helm
     ```
4. Install Confluent For Kubernetes using Helm [Detailed Instructions Here](https://docs.confluent.io/operator/current/co-deploy-cfk.html#deploy-co-using-the-download-bundle). Note: we disable the custom pod security context and use teh default context with the set podSecurity command:
     ```
     helm upgrade --install confluent-operator confluentinc/confluent-for-kubernetes --set podSecurity.enabled=false --namespace confluent
     ```
5. Check that the Confluent For Kubernetes pod comes up and is running:
     ```
     kubectl get pods
     ```
6. Review Confluent Platform configs
     ```
     Confluent Platform components are installed as custom resources (CRs). 

     You can configure all Confluent Platform components as custom resources. However, in this example, all components are configured in a single file and deployed all at once with one ``kubectl apply`` command.

     The entire Confluent Platform is configured in one configuration file: $TUTORIAL_HOME/cflt_examples/security/openshift-security/confluent-platform-with-defaultSCC.yaml

     In this configuration file, there is a custom Resource configuration spec for each Confluent Platform component - replicas, image to use, resource allocations.
     ```
7. Deploy Confluent Platform:
     ```
     kubectl apply -f $TUTORIAL_HOME/cflt_examples/security/openshift-security/confluent-platform-with-defaultSCC.yaml
     ```
8. Check that all Confluent Platform resources are deployed:
     ```
     kubectl get confluent
     ```
9. Get the status of any component. For example, to check Kafka:
     ```
     kubectl describe kafka
     ```

---
---

## Validation:

1. Review sample producer application used for validation:
     ```
     The producer app is packaged and deployed as a pod on Kubernetes. The required topic is defined as a KafkaTopic custom resource in $TUTORIAL_HOME/cflt_examples/quickstart-deploy/producer-app-data.yaml
     ```
2. Deploy the producer app:
     ```
     kubectl apply -f $TUTORIAL_HOME/cflt_examples/quickstart-deploy/producer-app-data.yaml
     ```
3. Set up port forwarding to Control Center web UI from local machine:
     ```
     kubectl port-forward controlcenter-0 9021:9021
     ```
4. Browse to Control Center:
     ```
     Open browser and navigate to: http://localhost:9021
     ```
5. Validate functionality:
     ```
     Check that the ``elastic-0`` topic was created and that messages are being produced to the topic.
     ```

---
---

## Tear Down:

1. Shut down producer app:
     ```
     kubectl delete -f $TUTORIAL_HOME/cflt_examples/quickstart-deploy/producer-app-data.yaml
     ```
2. Shut down Confluent Platform:
     ```
     kubectl delete -f $TUTORIAL_HOME/cflt_examples/security/openshift-security/confluent-platform-with-defaultSCC.yaml
     ```
     ```
     helm uninstall confluent-operator
     ```