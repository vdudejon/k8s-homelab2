# Portworx
Ensure the Portworx Operator is installed, see `app-of-apps/operators/portworx.`

To install Portworx Community Edition, you will need to go to Portworx Central and create the manifest using their wizard.

Once you have the manifest, convert the secret part into a sealed secret:
1. Save the secret to a file where `kubeseal` is installed
2. Run the following, which will save the Sealed Secret to `sealedsecret.yaml`:
    ```bash
    kubeseal --controller-name sealed-secrets --controller-namespace sealed-secrets kubeseal -f secret.yaml -w sealedsecret.yaml
    ```
3. You can safely place the sealedsecret.yaml file into GitHub.
4. Delete `secret.yaml`

Paste the `StorageCluster` manifest into `portworx.yaml`

Once the cluster is up and running correctly, unmark the local-path storageclass as default:
    ```bash
    kubectl patch storageclass local-path -p '{"metadata": {"annotations": {"storageclass.kubernetes.io/is-default-class": "false"}}}'
    ```

## Troubleshooting Portworx
You can use the `pxctl` command from pods running the Portworx cluster.
    ```bash
    kubectl -n portworx get pods -l name=portworx
    kubectl -n portworx exec -it px-cluster-<longname> -- /bin/sh
    pxctl status
    ...
    ```

## Benchmark
You can run kbench from Rancher Longhorn like so:
```bash
kubectl apply -f https://raw.githubusercontent.com/yasker/kbench/main/deploy/fio.yaml
```

Observe the results: 
```bash
kubectl logs -l kbench=fio -f
```

Cleanup:
```bash
kubectl delete -f https://raw.githubusercontent.com/yasker/kbench/main/deploy/fio.yaml
```

My results:
```
TEST_FILE: /volume/test
TEST_OUTPUT_PREFIX: test_device
TEST_SIZE: 30G
Benchmarking iops.fio into test_device-iops.json
Benchmarking bandwidth.fio into test_device-bandwidth.json
Benchmarking latency.fio into test_device-latency.json

=========================
FIO Benchmark Summary
For: test_device
CPU Idleness Profiling: disabled
Size: 30G
Quick Mode: disabled
=========================
IOPS (Read/Write)
        Random:         107,707 / 42,508
    Sequential:         111,469 / 50,472

Bandwidth in KiB/sec (Read/Write)
        Random:      1,983,882 / 273,075
    Sequential:      2,335,763 / 284,312


Latency in ns (Read/Write)
        Random:         39,650 / 405,257
    Sequential:         35,059 / 400,585
```