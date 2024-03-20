# Rolling updates

- `kubectl rollout status deploy/<name>`
- `kubectl rollout history deploy/<name>`

Deployment strategies:
 - Recreate - Delete all, create all after
 - Rolling Update - Delete one, create one      # The default behavior

# Rollback
- `kubectl rollout undo deploy/<name>`   # Will destroy the new pods and create new pods one at a time using replicasets