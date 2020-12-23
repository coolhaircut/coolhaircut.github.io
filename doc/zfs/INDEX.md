# ZFS Opionions

## Zpool Features
* __allocation_classes__: `enabled`
  * Enables the creations of special `allocation` type `vdevs` (dedup tables, metadata, single filesystems). *(__Note__: Probably want to keep the `ashift` value consistent if using these.)*
* __async_destroy__: `enabled`
  * Allows "lazy" instant removal of filesystems (does not block until the enitre fstree is traversed).
* __bookmarks__: `enabled`

### Command
```
VDEVS=(/dev/sda1 /dev/sdb2) # 2 example disks
ZPOOL="deadpool"            # whatever name you want

sudo zpool create $ZPOOL ${VDEVS[@]}    \
  -o feature@allocation_classes=enabled \
  -o feature@async_destroy=enabled      \
  -o feature@bookmarks=enabled          \
  -o feature@embedded_data=enabled      \
  -o feature@empty_bpobj=enabled        \
  -o feature@enabled_txg=enabled        \
  -o feature@extensible_dataset=enabled \
  -o feature@filesystem_limits=enabled   \
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

## Create zfs volume
sudo zfs create ${ZPOOL-tank}/${ZVOL-dset}     \
-o canmount=off                                \
-o compression=on                              \
-o atime=off                                   \
-o relatime=on                                 \
-o checksum=fletcher4                          \
-o compression=lz4                             \
-o xattr=off                                   \
-o utf8only=on                                 \

## w/ ENCRYPTION PASSPHRASE
-o encryption=on                               \
-o keyformat=passphrase                        \
-o keylocation=prompt                          \
-o pbkdf2iters="$(((RANDOM % 250000)+100000))" \

## w/ ENCRYPTION KEYFILE
#sudo dd if=/dev/urandom of=/key bs=32 count=1
-o encryption=on                               \
-o keyformat=raw                               \
-o keylocation=/root/.config/zfs/.key          \

## w/ BITTORRENT
-o recordsize=16k                              \
