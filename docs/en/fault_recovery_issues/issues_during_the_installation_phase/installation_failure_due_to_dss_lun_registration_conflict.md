# Installation Failure Due to DSS LUN Registration Conflicts

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-08-05T07:47:27.403Z pushedAt=2026-08-17T00:46:29.657Z -->

## Symptom

In two-node or multi-node deployment scenarios, an anomaly in the `DSS` shared disk may cause installation failure. Common symptoms are as follows:

* Failure during the `install` or `start` phase of the installation process.
* The `DSS` component fails to start.
* Abnormal information appears in the following logs:

```text
/opt/ograc/log/dss/run/instance.log
```

When the DSS registration phase fails, a typical error in `/opt/ograc/log/dss/run/instance.log` is as follows:

```text
ERROR [pid: 2127231] [MainThread] [tid:281460975927024] [dssctl.py:596] Reghl node cmd[source ~/.bashrc && /opt/ograc/dss/bin/dsscmd reghl -D /opt/ograc/dss] failed, details: Begin to register,
Failed to get vg non entry info when reghl, errcode is -1.
 detail reason[2031]: The volume group has not been initialized.
Failed to register.
```

## Common Causes

* The shared `LUN` has been registered by another cluster or a historical environment.
* A previous installation was not properly uninstalled, leaving residual registration information.
* Other services have performed `Persistent Reservation` on the shared disk.

## Troubleshooting

### Viewing LUN Registration Information

```shell
sg_persist --in --read-keys /dev/xxx
```

If the following content is returned, it indicates that registration information already exists on the LUN:

```text
PR generation=0xb6, 2 registered reservation keys follow:
0x1
0x2
```

### Cleaning Up Registration Information

Perform the cleanup operation for each registered key:

```shell
sg_persist --out --clear --param-rk=<key> /dev/xxx
```

Where `<key>` is the reservation key obtained in the previous step.

### Confirming the Cleanup Result

Run the command again:

```shell
sg_persist --in --read-keys /dev/xxx
```

If the output is as follows:

```text
there are NO registered reservation keys
```

The shared disk registration information has been successfully cleaned up. You can re-execute the installation or startup process.
