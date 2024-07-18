# External Access to OpenShift Instance

## Overview
This document provides a guide on how to push files to an instance within OpenShift from another container. This can be useful for various scenarios, such as deploying applications or transferring configuration files.

## Prerequisites
- Access to an OpenShift cluster.
- A running container within the OpenShift cluster.
- Another container or system from which you want to push files.

## Steps

### Step 1: Expose the Container
First, expose the container you want to push files to by creating a service.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: target-service
spec:
  ports:
  - port: 22
    targetPort: 22
  selector:
    app: target-app
  type: ClusterIP
```

### Step 2: Create a Route
Create a route to make the service accessible from outside the cluster.

```yaml
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  name: target-route
spec:
  to:
    kind: Service
    name: target-service
  port:
    targetPort: 22
```

### Step 3: Push Files
From another container or system, use `scp` or any other file transfer method to push files to the exposed container.

```sh
scp /path/to/local/file username@target-route-host:/path/to/remote/directory
```

Replace `username` with the appropriate username for the target container, `target-route-host` with the hostname provided by the route, and the paths with the appropriate local and remote paths.

## Example

### Example Service and Route
Here is an example of a service and route configuration:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: example-service
spec:
  ports:
  - port: 22
    targetPort: 22
  selector:
    app: example-app
  type: ClusterIP
---
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  name: example-route
spec:
  to:
    kind: Service
    name: example-service
  port:
    targetPort: 22
```

### Example File Push
From another container or system, push a file to the example container:

```sh
scp /local/file.txt user@example-route-host:/remote/directory/
```

## Conclusion
By following these steps, you can push files to a container within an OpenShift instance from an external source. This method leverages OpenShift's service and route capabilities to securely expose the container for file transfers.

oc adm policy add-scc-to-user anyuid -z default
sudo mount -t nfs nfs-server-alpine.nfs-server.svc.cluster.local:/nfsshare /mnt/nfs