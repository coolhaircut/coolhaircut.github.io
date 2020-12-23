# ZFS Opinions
* __Version__: 0.8.4

## Zpools

### Zpool - Properties
The following properties can be set at creation time, import time, and later changed with the `zpool set` command:

| Property | Value | Short Explaination | Comment |
| :-: | :-: | :- | :- |
| __ashift__ | `9` *or* `12` | Pool sector size exponent of 2 | Values from 9 to 16 are ll valid; also, the value 0 (the default) means to auto detect using the kernel's block layer. For best performance, the pool sector size should the same as the disk sector size (`9` = 512 bytes, `12` = 4096 bytes). |
| __autoexpand__ | `on` | Automatically grow pool when `vdevs` are added | Enables pools to resize itself according to the size of the expanded device.  If the device is part of a mirror or raidz then all devices within that mirror/raidz group must be expanded before the new space is made available to the pool. This is **off** by default. |
| __autoreplace__ | `off` | Automatically replace failed disks with new disks found in the same location | If set to off, device replacement must be initiated by the administrator by using the zpool replace command.  If set to on, any new device, found in the same physical location as a device that previously belonged to the pool, is automatically formatted and replaced. The default behavior is **off**. |
| __bootfs__ | `(unset)` | Identifies the default bootable dataset for the root pool | |
| __cachefile__ | `(blank)` | Path to pool location cache data |  Discovering all pools on system startup requires a cached copy of the configuration data that is stored on the root file system. Can be fixed with `zpool import -c`. Setting it to the value **none** creates a temporary pool that is never cached, and the **"" (empty string)** uses the default location. |
| __comment__ | `text` | ASCII text comment | Will be stored such that it is available even if the pool becomes faulted. Can provide additional information about a pool using this property. |
| __delegation__ | `on` | Allow non-privileged users based on the dataset permissions | |
| __failmode__ | `continue` | Behavior in the event of catastrophic pool failure | This condition is typically a result of a loss of connectivity to the underlying storage device(s)  or a failure of all devices within the pool. **wait** blocks until available, **continue** keeps trucking, **panic** crashes. |
| __autotrim__ | `off` |Freed, unallocated space periodically trimmed | This allows block device `vdevs` which support DISCARD (mainly SSDS) to reclaim unused blocks.  The default setting for this property is off. *(__Note__: This can put significant stress on the underlying storage devices,  depending of how well the specific device handles TRIM commands.  For lower end devices it is often possible to achieve most of the benefits of automatic trimming by running an on-demand (manual) TRIM periodically using the `zpool trim` command.)* |
| __listsnapshots__ | `off` | Don't list snapshots associated with this pool when zfs list is run without the -t option. | The default value is **off**. |
| __multihost__ | `off` | Don't check pool activity during `zpool import` |  When a pool is determined to be active it cannot be imported, even with the `-f` option.  This property is intended to be used in failover configurations where multiple hosts have access to a pool on shared storage. |

## Zpool Features
| Feature | Value | Comment |
| :-: | :-: | :- |
| __allocation_classes__ | enabled | Enables the creations of special `allocation` type `vdevs` (dedup tables, metadata, single filesystems).<br />*(__Note__: Probably want to keep the `ashift` value consistent if using these.)* |
| __async_destroy__ | enabled | Allows "lazy" instant removal of filesystems (does not block until the enitre fstree is traversed). |
| __bookmarks_v2__ | enabled | Enables larger version 2 bookmarks to be created *(__Note__: List bookmarks using `fs list -t bookmark -r $zpool`)* |
| __device_removal__ | enabled | Enables top level `vdevs` to be removed and shirnk the pool. %CAUTION% |
|

### Zpool Create Code
```
VDEVS=(/dev/sda1 /dev/sdb2)             # 2 example disks
ZPOOL="deadpool"                        # whatever name you want
sudo zpool create $ZPOOL ${VDEVS[@]}    \
  -o feature@allocation_classes=enabled \
  -o feature@async_destroy=enabled      \
  -o feature@bookmarks_v2=enabled       \
  -o feature@embedded_data=enabled      \
  -o feature@empty_bpobj=enabled        \
  -o feature@enabled_txg=enabled        \
  -o feature@extensible_dataset=enabled \
  -o feature@filesystem_limits=enabled  \
  -o feature@hole_birth=enabled         \
  -o feature@large_blocks=enabled       \
  -o feature@lz4_compress=enabled       \
  -o feature@project_quota=enabled      \
  -o feature@resilver_defer=enabled     \
  -o feature@spacemap_histogram=enabled \
  -o feature@spacemap_v2=enabled        \
  -o feature@userobj_accounting=enabled \
  -o feature@zpool_checkpoint=enabled   \
  -o failmode=continue                  \
  -o ashift=12
 ```

## Zvol

### Zvol Create Command
```
sudo zfs create $ZPOOL/$VOLUME_NAME     \
  -o canmount=off                       \
  -o compression=on                     \
  -o atime=off                          \
  -o relatime=on                        \
  -o checksum=fletcher4                 \
  -o compression=lz4                    \
  -o xattr=off                          \
  -o utf8only=on                        \
```

#### Encryption Passphrase Options
```
  -o encryption=on                       \
  -o keyformat=passphrase                \
  -o keylocation=prompt                  \
  -o pbkdf2iters=250000)                 \
```

#### Encryption Key File Options
Generate a random 32 byte key with `dd if=/dev/random of=/path/to/key bs=32 count=1`

```
  -o encryption=on                        \
  -o keyformat=raw                        \
  -o keylocation=/path/to/key             \
```

#### Larger recordsizes
Some applications that write consistently sized blocks (e.g. databases, torrents) may see performance benefits from using a matching `recordsize`.

```
  -o recordsize=16k
``` 
