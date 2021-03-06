
SealedSecrets
=============

The cluster has a [sealed-secrets](https://github.com/bitnami-labs/sealed-secrets) controller.

- `SealedSecrets` are encrypted and safe to store in git
- use `kubeseal` to generate `SealedSecret` yaml files
- controller decrypts them and creates `Secret` objects

These operations require `kubectl` with certain RBAC rights.
So currently this is an admin only operation.

If needed we can provide devs with the public key. This way they don't need kubectl access to create SealedSecrets.

The reason for using SealedSecrets is to make disaster recovery easier.

## Creating/Updating SealedSecrets

Example of creating a `SealedSecret` from an existing `Secret`:

```
kubectl -n stg get secret your-secret -o yaml > your-secret.yaml 
# Edit your-secret.yaml to your hearts content
kubeseal \
    --controller-namespace infra \
    --controller-name sealed-secrets \
    --format=yaml <your-secret.yaml > your-secret-as-a-sealedsecret.yaml
```

NOTE: You can't change the `name` or `namespace` field of a `SealedSecret` yaml file as these values are used to to encrypt the content. If you change these decryption will fail.

## Backup and Recovery

The master/private key is backed up in `it-admin`.

The recovery process is to replace the auto-generated private key with the key from backup and restart the controller:

```
kubectl replace secret -n infra sealed-secrets-key -f sealed-secrets-key.yaml
kubectl delete pod -n infra -l app=sealed-secrets
```
