# CI-Secret-Bootstrap

This tool extends the [populate-secrets-from-bitwarden.sh](https://github.com/openshift/release/blob/c8c89d08c56c653b91eb8c7580657f7ce522253f/ci-operator/populate-secrets-from-bitwarden.sh)
to support mirroring secrets cross Kubernetes/OpenShift-clusters.

## Args and config.yaml

We use `--kubeconfig` to specify the path to a [kube config](https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/)
that the tool will load and use it to access clusters for writing secrets.

It expects a configuration like the one below which specifies the mapping from the items
in BitWarden and the targeting secret.

```yaml
- from:
    key-name-1:
      bw_item: item-name-1
      field: field-name-1
    key-name-2:
      bw_item: item-name-1
      field: field-name-2
    key-name-3:
      bw_item: item-name-1
      attachment: attachment-name-1
    key-name-4:
      bw_item: item-name-2
      field: field-name-1
    key-name-5:
      bw_item: item-name-2
      attachment: attachment-name-1
    key-name-6:
      bw_item: item-name-3
      attachment: attachment-name-2
  to:
    - cluster: default
      namespace: namespace-1
      name: prod-secret-1
    - cluster: build01
      namespace: namespace-2
      name: prod-secret-2

```

where `cluster` is `context` name in the `kubeconfig`:

* `default`: `https://api.ci.openshift.org:443`, and
* `build01`: `https://api.build01.ci.devcluster.openshift.com:6443`.

So the above configuration tells the tool to use the following data to
create a secret with its `key` as `secret.data.key` and the following as `secret.data.value`:

* `field`s of `field-name-1` and `field-name-2`, and the `attachment` of `attachment-name-1` in
Bitwarden item `item-name-1`, and

* `field` of `field-name-3`, and the `attachments` of `attachment-name-2` and `attachment-name-3` in
Bitwarden item `item-name-2`.

And then the secret will be populated to

* the `secret` `prod-secret-1` in `namespace-1` on the `default` cluster, and
* the `secret` `prod-secret-2` in `namespace-2` on the `build01` cluster.


## Run

```bash
$ echo -n "bw_password" > /tmp/bw_password 
$ ci-secret-bootstrap --bw-password-path=/tmp/bw_password -bw-user kerberos_id@redhat.com --kubeconfig <path_to_kubeconfig_file> --config <path_to_config.yaml>

```

where `kubeconfig` contains the `contexts` for the `default` cluster and the `build01` cluster.
