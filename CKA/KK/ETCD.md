# ETCDCTL Command Reference

`ETCDCTL` is the command-line interface (CLI) used to interact with **ETCD**.

ETCDCTL can communicate with the ETCD server using **two API versions**:

- Version 2
- Version 3

By default, ETCDCTL uses **API Version 2**. Each API version provides a different set of commands.

---

## ETCDCTL Version 2 Commands

```bash
etcdctl backup
etcdctl cluster-health
etcdctl mk
etcdctl mkdir
etcdctl set
```

---

## ETCDCTL Version 3 Commands

```bash
etcdctl snapshot save
etcdctl endpoint health
etcdctl get
etcdctl put
```

---

## Selecting the API Version

To use the Version 3 API, set the following environment variable:

```bash
export ETCDCTL_API=3
```

> **Note**
>
> - If `ETCDCTL_API` is **not set**, ETCDCTL assumes **Version 2**.
> - Version 3 commands will **not work** while using Version 2.
> - Likewise, Version 2 commands will **not work** when using Version 3.

---

# Using TLS Certificates

ETCDCTL must authenticate with the ETCD API server using TLS certificates.

On the control plane node, the certificate files are typically located at:

```text
/etc/kubernetes/pki/etcd/ca.crt
/etc/kubernetes/pki/etcd/server.crt
/etc/kubernetes/pki/etcd/server.key
```

When running ETCDCTL commands, specify these certificates:

```bash
--cacert /etc/kubernetes/pki/etcd/ca.crt
--cert /etc/kubernetes/pki/etcd/server.crt
--key /etc/kubernetes/pki/etcd/server.key
```

---

# Complete Example

The following command lists the first 10 keys stored in ETCD.

```bash
kubectl exec etcd-controlplane -n kube-system -- sh -c \
"ETCDCTL_API=3 etcdctl get / \
  --prefix \
  --keys-only \
  --limit=10 \
  --cacert /etc/kubernetes/pki/etcd/ca.crt \
  --cert /etc/kubernetes/pki/etcd/server.crt \
  --key /etc/kubernetes/pki/etcd/server.key"
```

---

# Summary

| API Version | Example Commands |
|-------------|------------------|
| Version 2 | `backup`, `cluster-health`, `mk`, `mkdir`, `set` |
| Version 3 | `snapshot save`, `endpoint health`, `get`, `put` |

### API Version

```bash
export ETCDCTL_API=3
```

### Required TLS Files

```text
/etc/kubernetes/pki/etcd/ca.crt
/etc/kubernetes/pki/etcd/server.crt
/etc/kubernetes/pki/etcd/server.key
```

### General Syntax

```bash
ETCDCTL_API=3 etcdctl <command> \
  --cacert /etc/kubernetes/pki/etcd/ca.crt \
  --cert /etc/kubernetes/pki/etcd/server.crt \
  --key /etc/kubernetes/pki/etcd/server.key
```
