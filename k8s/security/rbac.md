# RBAC

Map a user to a role. Every request to K8S is authenticated. Providing the identity caller in the request.
After that, it performs the authorization of the request.

## Identity In K8S

Makes a distinction between user identities and service account identities.
K8S uses a generic interface for authentication providers that provides a username and optionally the set of groups of that user.

The authentication could be done using:

- Basic Auth
- x509 client
- Static token files
- IAM
- Webhooks auth

## Example

```yaml
---
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
namespace: default
name: pod-and-services
rules:
  - apiGroups: [""]
resources: ["pods", "services"]
verbs: ["create", "delete", "get", "list", "patch", "update", "watch"]
```

And use a `RoleBinding`

```yaml
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  namespace: default
  name: pods-and-services
subjects:
  - apiGroup: rbac.authorization.k8s.io
  kind: User
  name: alice
  - apiGroup: rbac.authorization.k8s.io
  kind: Group
  name: mydevs
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: pod-and-services
```

Or use any built-in roles:

- cluster-admin
- admin
- edit
- view

We can get rbac if the current user can perform anything with:

```bash
kubectl auth can-i create pods

kubectl auth can-i get pods --subresource=logs
```

You can combine roles:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: edit
...
aggregationRule:
  clusterRoleSelectors:
  - matchLabels:
    rbac.authorization.k8s.io/aggregate-to-edit: "true"
```