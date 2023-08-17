# User Management K8S

### Prequisites&#x20;

* Kubernetes cluster
* kubectl command-line tool

> &#x20;<mark style="color:blue;">**Note:**</mark>** You must have at least two nodes in the cluster**&#x20;

To create namespace named `examplens:`

```shell
kubectl create namespace examplens
```

\-----------------------------------------------------------------------------------------------------

## Role Base Access Control

1. <mark style="color:yellow;">**Creating Resources**</mark><mark style="color:yellow;">:</mark> You can control whether users or service accounts can create resources (e.g., pods, services, deployments) within the namespace.
2. <mark style="color:yellow;">**Updating Resources**</mark><mark style="color:yellow;">:</mark> You can specify who can update or modify existing resources in the namespace.
3. <mark style="color:yellow;">**Deleting Resources**</mark><mark style="color:yellow;">:</mark> You can manage the ability to delete resources within the namespace.
4. <mark style="color:yellow;">**Viewing Resources**</mark>: You can control whether users or service accounts can view the resources within the namespace.
5. <mark style="color:yellow;">**ConfigMaps and Secrets**</mark><mark style="color:yellow;">:</mark> Permissions to read or modify ConfigMaps and Secrets within the namespace can be controlled.
6. <mark style="color:yellow;">**Resource Quotas**</mark><mark style="color:yellow;">:</mark> You can define resource quotas for a namespace to limit the amount of CPU, memory, and other resources that can be consumed.
7. <mark style="color:yellow;">**Pod Security Policies**</mark>: You can enforce specific security policies at the namespace level to control the security context of pods.
8. <mark style="color:yellow;">**Network Policies**</mark><mark style="color:yellow;">:</mark> You can define network policies to control traffic flow between pods within the namespace.
9. <mark style="color:yellow;">**Custom Resource Definitions (CRDs)**</mark>: If you have custom resources, you can manage who can create, update, and delete them in the namespace.

## Apply Resource Quotas RBAC

Create a Namespace (if you haven't already) If you don't have a namespace in which you want to set resource quotas, create one using the `kubectl create namespace` command:

```bash
bashCopy codekubectl create namespace gpu-namespace
```

Define a Resource Quota with GPU Resource Limit Create a YAML file (e.g., `gpu-resource-quota.yaml`) to define the resource quota with GPU resource limits. The following is an example of a resource quota that limits the total amount of CPU, memory, and the number of GPUs that can be used within the namespace:

```yaml
yamlCopy codeapiVersion: v1
kind: ResourceQuota
metadata:
  name: gpu-resource-quota
  namespace: gpu-namespace
spec:
  hard:
    cpu: "2"
    memory: 4Gi
    nvidia.com/gpu: "1"  # Limiting the number of NVIDIA GPUs to 1
```

In this example, we set a limit of 2 CPU cores, 4Gi of memory, and 1 NVIDIA GPU for the `gpu-namespace` namespace.

Apply the Resource Quota Apply the resource quota to the Kubernetes cluster using the `kubectl apply` command:

```bash
bashCopy codekubectl apply -f gpu-resource-quota.yaml
```

Verify the Resource Quota To check if the resource quota was applied successfully, you can use the `kubectl describe` command:

```bash
bashCopy codekubectl describe resourcequota gpu-resource-quota -n gpu-namespace
```

This command will display the details of the resource quota you defined, including the GPU limit.

Test the Resource Quota with GPU Resources Now that the resource quota is in place, you can create and deploy resources within the namespace. Kubernetes will automatically enforce the quota limits you specified. If you try to create resources that exceed the defined limits, Kubernetes will prevent their creation and display an error message.

For example, let's try creating a Pod that requests more GPU resources than the quota allows:

```yaml
yamlCopy codeapiVersion: v1
kind: Pod
metadata:
  name: gpu-pod
  namespace: gpu-namespace
spec:
  containers:
    - name: gpu-container
      image: nvidia/cuda:11.0-base
      resources:
        requests:
          cpu: "1"
          memory: "2Gi"
        limits:
          nvidia.com/gpu: "2"
```

When you apply this Pod YAML, Kubernetes will reject it because it exceeds the GPU quota limit.

Remember that GPU resources are specified using custom resource names, such as `nvidia.com/gpu` in the example above. The exact name may vary depending on the GPU provider and the installed Kubernetes GPU support.

That's it! You've set up a Resource Quota to limit GPU resource consumption within a Kubernetes namespace. Make sure to tailor the quota according to your specific GPU requirements.

## Apply Network Policies RBAC

Enable Network Policies Before you can use Network Policies, you need to ensure that your Kubernetes cluster has a network plugin that supports Network Policies. Some popular plugins that support Network Policies are Calico, Cilium, and Weave. Check with your cluster administrator to verify that Network Policies are enabled.

Create a Namespace (if you haven't already) If you don't have a namespace in which you want to set Network Policies, create one using the `kubectl create namespace` command:

```bash
bashCopy codekubectl create namespace my-namespace
```

Define a Network Policy Create a YAML file (e.g., `network-policy.yaml`) to define the Network Policy. The following is an example of a simple Network Policy that allows only specific pods to communicate with each other:

```yaml
yamlCopy codeapiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-communication
  namespace: my-namespace
spec:
  podSelector:
    matchLabels:
      app: my-app
  ingress:
    - from:
        - podSelector:
            matchLabels:
              role: backend
```

In this example, we create a Network Policy named `allow-communication` in the `my-namespace` namespace. The policy allows any pod with the label `app: my-app` to receive incoming connections only from pods labeled with `role: backend`.

Apply the Network Policy Apply the Network Policy to the Kubernetes cluster using the `kubectl apply` command:

```bash
bashCopy codekubectl apply -f network-policy.yaml
```

Verify the Network Policy To check if the Network Policy was applied successfully, you can use the `kubectl describe` command:

```bash
bashCopy codekubectl describe networkpolicy allow-communication -n my-namespace
```

This command will display the details of the Network Policy you defined.

Test the Network Policy Now that the Network Policy is in place, pods that don't match the specified rules will be unable to communicate with each other. Test the Network Policy by deploying pods that should be allowed to communicate and pods that shouldn't be allowed. Verify that communication is restricted based on the rules you defined.

Keep in mind that Network Policies apply only within the same namespace. If you want to enforce network isolation across namespaces or the entire cluster, you can consider using additional network plugins or solutions.

Please note that the exact syntax and capabilities of Network Policies might vary slightly depending on the network plugin used in your cluster. Always refer to the documentation of your specific network plugin for detailed information on using Network Policies.

## Manage Resources by Time

Create a Namespace (if you haven't already) If you don't have a namespace in which you want to set Resource Quotas, create one using the `kubectl create namespace` command:

```bash
bashCopy codekubectl create namespace my-namespace
```

&#x20;Define a Resource Quota Create a YAML file (e.g., `initial-resource-quota.yaml`) to define the initial Resource Quota with default CPU and memory limits. For example:

```yaml
yamlCopy codeapiVersion: v1
kind: ResourceQuota
metadata:
  name: my-resource-quota
  namespace: my-namespace
spec:
  hard:
    cpu: "1"
    memory: 1Gi
```

Apply the initial Resource Quota using:

```bash
bashCopy codekubectl apply -f initial-resource-quota.yaml
```

Create the Custom Script Write a custom script (e.g., `adjust-resource-quota.sh`) that updates the Resource Quota with new CPU and memory limits. The script could be as follows:

```bash
bashCopy code#!/bin/bash

# Determine the current hour of the day
current_hour=$(date +%H)

# Check if it's during peak hours (e.g., from 8 AM to 6 PM)
if [ $current_hour -ge 8 ] && [ $current_hour -lt 18 ]; then
  cpu_limit="2"
  memory_limit="2Gi"
else
  cpu_limit="1"
  memory_limit="1Gi"
fi

# Update the Resource Quota with the new limits
kubectl patch resourcequota my-resource-quota -n my-namespace \
  --type=merge \
  -p '{"spec":{"hard":{"cpu":"'${cpu_limit}'","memory":"'${memory_limit}'"}}}'
```

Make sure to grant execution permission to the script:

```bash
bashCopy codechmod +x adjust-resource-quota.sh
```

Create the Cron Job Create a YAML file (e.g., `adjust-resource-quota-cronjob.yaml`) to define the Cron Job that runs the custom script periodically:

```yaml
yamlCopy codeapiVersion: batch/v1
kind: CronJob
metadata:
  name: resource-quota-adjustment
  namespace: my-namespace
spec:
  schedule: "0 8-17 * * 1-5" # Run every minute from 8 AM to 5 PM, Monday to Friday
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: adjust-resource-quota
            image: <your_custom_script_image> # Replace with your custom script's image
            command: ["/bin/sh", "-c", "/path/to/adjust-resource-quota.sh"]
          restartPolicy: OnFailure
```

Apply the Cron Job using:

```bash
bashCopy codekubectl apply -f adjust-resource-quota-cronjob.yaml
```

The Cron Job will now execute the custom script every minute between 8 AM and 5 PM from Monday to Friday, updating the Resource Quota with different CPU and memory limits based on whether it's during peak hours or off-peak hours.

Please note that this is a simplified example, and in a real-world scenario, you may need to adjust the script and Cron Job schedule to match your specific requirements and time intervals. Also, ensure that the script is running in a secure environment with appropriate permissions to interact with the Kubernetes API.

## Manage Memory, CPU, and API Resources

***

[**Configure Default Memory Requests and Limits for a Namespace**](https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/memory-default-namespace/)

Define a default memory resource limit for a namespace, so that every new Pod in that namespace has a memory resource limit configured.

[**Configure Default CPU Requests and Limits for a Namespace**](https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/cpu-default-namespace/)

Define a default CPU resource limits for a namespace, so that every new Pod in that namespace has a CPU resource limit configured.

[**Configure Minimum and Maximum Memory Constraints for a Namespace**](https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/memory-constraint-namespace/)

Define a range of valid memory resource limits for a namespace, so that every new Pod in that namespace falls within the range you configure.

[**Configure Minimum and Maximum CPU Constraints for a Namespace**](https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/cpu-constraint-namespace/)

Define a range of valid CPU resource limits for a namespace, so that every new Pod in that namespace falls within the range you configure.

[**Configure Memory and CPU Quotas for a Namespace**](https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/quota-memory-cpu-namespace/)

Define overall memory and CPU resource limits for a namespace.

[**Configure a Pod Quota for a Namespace**](https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/quota-pod-namespace/)

Restrict how many Pods you can create within a namespace.
