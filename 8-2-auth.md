Roles and RoleBindings are created namespace-wise. By default they are created in the default namespace and control access within it

# Role:

```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["list", "get", "create", "update", "delete"]
```
- If you want to scope which particular resource within a pod/deployment etc. you can also add the `rules[].resourceGroups[]` field


# Role Binding:
- A role binding grants the permissions defined in a role to a user or set of users. It holds a list of subjects (users, groups, or service accounts), and a reference to the role being granted. A RoleBinding grants permissions within a specific namespace

```
apiVersion: rbac.authorization.k8s.io/v1
# This role binding allows "jane" to read pods in the "default" namespace.
# You need to already have a Role named "pod-reader" in that namespace.
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
# You can specify more than one "subject"
- kind: User
  name: jane # "name" is case sensitive
  apiGroup: rbac.authorization.k8s.io
roleRef:
  # "roleRef" specifies the binding to a Role / ClusterRole
  kind: Role #this must be Role or ClusterRole
  name: pod-reader # this must match the name of the Role or ClusterRole you wish to bind to
  apiGroup: rbac.authorization.k8s.io
```


- To test whether you have the perms to do something you can do something like:
`kubectl auth can-i create deployments` OR `kubectl auth can-i delete nodes`

- To test if a different user has the perms to do something you can run:
- `kubectl auth can-i create deployments --as dev-user` OR
- `kubectl auth can-i create deployments --as dev-user --namespace kube-system` OR
- To get a sub-resource/specific resource: `k auth can-i get pods/dark-blue-app --as dev-user --namespace=blue`

- The allowed authorization modes are stored in the `--authorization` flag inside the `kube-apiserver.yaml`


# ClusterRoles
- Used for cluster-scoped resources like nodes
- However, you can use it for non-cluster-scoped resources. That would mean that the access is not namespaced, and the user has access to the resources across all namespaces

```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  # "namespace" omitted since ClusterRoles are not namespaced
  name: secret-reader
rules:
- apiGroups: [""]
  #
  # at the HTTP level, the name of the resource for accessing Secret
  # objects is "secrets"
  resources: ["secrets"]
  verbs: ["get", "watch", "list"]
```

- You can still use a RoleBinding for a ClusterRole. You would do that to scope a particular ClusterRole on a namespace level for users who need limited access.
- This kind of reference lets you define a set of common roles across your cluster, then reuse them within multiple namespaces.


# ClusterRoleBinding
```
apiVersion: rbac.authorization.k8s.io/v1
# This cluster role binding allows anyone in the "manager" group to read secrets in any namespace.
kind: ClusterRoleBinding
metadata:
  name: read-secrets-global
subjects:
- kind: Group
  name: manager # Name is case sensitive
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: secret-reader
```

# Creating a User 
- Generate a kubernetes CSR
- Approve the CSR through kubectl
- Create a Role and Role Binding
- To verify, run `kubectl auth can-i <do something> --as <new_user>`