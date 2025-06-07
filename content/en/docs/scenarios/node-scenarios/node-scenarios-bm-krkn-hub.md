---
title: Node Scenarios on Bare Metal using Krkn-Hub
description: >
date: 2017-01-05
weight: 5
---

### Node Scenarios Bare Metal
This scenario disrupts the node(s) matching the label on a bare metal Kubernetes/OpenShift cluster. Actions/disruptions supported are listed [here](https://github.com/krkn-chaos/krkn/blob/master/docs/node_scenarios.md)

#### Run
Unlike other krkn-hub scenarios, this one requires a specific configuration due to its unique structure. 
You must set up the scenario in a local file following the [scenario syntax](https://github.com/krkn-chaos/krkn/blob/main/scenarios/openshift/baremetal_node_scenarios.yaml), 
and then pass this file's base64-encoded content to the container via the SCENARIO_BASE64 variable.

If enabling [Cerberus](https://github.com/krkn-chaos/krkn#kraken-scenario-passfail-criteria-and-report) to monitor the cluster and pass/fail the scenario post chaos, refer [docs](https://krkn-chaos.dev/docs/cerberus/installation/). Make sure to start it before injecting the chaos and set `CERBERUS_ENABLED` environment variable for the chaos injection container to autoconnect.

```
$ podman run --name=<container_name> --net=host \
    -env-host=true \
    -e SCENARIO_BASE64="$(base64 -w0 <scenario_file>)" \
    -v <path-to-kube-config>:/home/krkn/.kube/config:Z -d quay.io/krkn-chaos/krkn-hub:node-scenarios-bm
$ podman logs -f <container_name or container_id> # Streams Kraken logs
$ podman inspect <container-name or container-id> --format "{{.State.ExitCode}}" # Outputs exit code which can considered as pass/fail for the scenario
```

```
$ docker run $(./get_docker_params.sh) --name=<container_name> --net=host \
    -e SCENARIO_BASE64="$(base64 -w0 <scenario_file>)" \
    -v <path-to-kube-config>:/home/krkn/.kube/config:Z -d quay.io/krkn-chaos/krkn-hub:node-scenarios-bm
OR 
$ docker run \
     -e SCENARIO_BASE64="$(base64 -w0 <scenario_file>)" \
     --net=host -v <path-to-kube-config>:/home/krkn/.kube/config:Z -d quay.io/krkn-chaos/krkn-hub:node-scenarios-bm

$ docker logs -f <container_name or container_id> # Streams Kraken logs
$ docker inspect <container-name or container-id> --format "{{.State.ExitCode}}" # Outputs exit code which can considered as pass/fail for the scenario
```

**TIP**: Because the container runs with a non-root user, ensure the kube config is globally readable before mounting it in the container. You can achieve this with the following commands:
```kubectl config view --flatten > ~/kubeconfig && chmod 444 ~/kubeconfig && docker run $(./get_docker_params.sh) --name=<container_name> --net=host -v ~kubeconfig:/home/krkn/.kube/config:Z -d quay.io/krkn-chaos/krkn-hub:<scenario>```

#### Supported parameters

The following environment variables can be set on the host running the container to tweak the scenario/faults being injected:

ex.) 
`export <parameter_name>=<value>`

Parameter               | Description                                                           | Default
----------------------- | -----------------------------------------------------------------     | ------------------------------------ |
ACTION                  | Action can be one of the [following](https://github.com/krkn-chaos/krkn/blob/master/docs/node_scenarios.md) | node_stop_start_scenario |
LABEL_SELECTOR          | Node label to target                                                  | node-role.kubernetes.io/worker       |
NODE_NAME               | Node name to inject faults in case of targeting a specific node; Can set multiple node names separated by a comma      | ""                                   |
INSTANCE_COUNT          | Targeted instance count matching the label selector                   | 1                                    |
RUNS                    | Iterations to perform action on a single node                         | 1                                    |
CLOUD_TYPE              | Cloud platform on top of which cluster is running, supported platforms - aws, vmware, ibmcloud, bm           | aws |
TIMEOUT                 | Duration to wait for completion of node scenario injection             | 180                                |
DURATION                | Duration to stop the node before running the start action - not supported for vmware and ibm cloud type             | 120                                |
KUBE_CHECK       | Connect to the kubernetes api to see if the node gets to a certain state during the node scenario   | False                               |
PARALLEL     | Run action on label or node name in parallel or sequential, set to true for parallel | False |
BMC_USER                 | Only needed for Baremetal ( bm ) - IPMI/bmc username | "" |
BMC_PASSWORD             | Only needed for Baremetal ( bm ) - IPMI/bmc password | "" |
BMC_ADDR                 | Only needed for Baremetal ( bm ) - IPMI/bmc username | "" |

See list of variables that apply to all scenarios [here](https://deploy-preview-87--krkn-chaos.netlify.app/docs/scenarios/all-scenario-env/) that can be used/set in addition to these scenario specific variables

#### Demo
You can find a link to a demo of the scenario [here](https://asciinema.org/a/ANZY7HhPdWTNaWt4xMFanF6Q5)

**NOTE** In case of using custom metrics profile or alerts profile when `CAPTURE_METRICS` or `ENABLE_ALERTS` is enabled, mount the metrics profile from the host on which the container is run using podman/docker under `/home/krkn/kraken/config/metrics-aggregated.yaml` and `/home/krkn/kraken/config/alerts`. For example:
