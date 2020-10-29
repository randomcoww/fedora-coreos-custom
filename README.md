# fedora-coreos-custom

#### Build image

From upstream: https://github.com/coreos/coreos-assembler

```
cosa() {
   env | grep COREOS_ASSEMBLER
   set -x
   podman run --rm -ti --security-opt label=disable --privileged -w /srv                            \
              --uidmap=10000:0:1 --uidmap=0:1:10000 --uidmap 10001:10001:55536                      \
              -v ${PWD}:/srv/ --device /dev/kvm --device /dev/fuse                                  \
              --tmpfs /tmp -v /var/tmp:/var/tmp --name cosa-coreos                                  \
              ${COREOS_ASSEMBLER_CONFIG_GIT:+-v $COREOS_ASSEMBLER_CONFIG_GIT:/srv/src/config/:ro}   \
              ${COREOS_ASSEMBLER_GIT:+-v $COREOS_ASSEMBLER_GIT/src/:/usr/lib/coreos-assembler/:ro}  \
              ${COREOS_ASSEMBLER_CONTAINER_RUNTIME_ARGS}                                            \
              ${COREOS_ASSEMBLER_CONTAINER:-quay.io/coreos-assembler/coreos-assembler:latest} "$@"
   rc=$?; set +x; return $rc
}
```

Check out config
```
cosa init https://github.com/randomcoww/fedora-coreos-custom.git --force
```

Add matchbox image. This host has no internet access and cannot download containers.
```
podman pull quay.io/poseidon/matchbox:latest
podman save --format oci-archive -o matchbox.tar quay.io/poseidon/matchbox:latest
sudo mv matchbox.tar src/config/resources
```

Run build
```
cosa clean && \
cosa fetch && \
cosa build metal4k && \
cosa buildextend-metal && \
cosa buildextend-live
```

Embed ignition from https://github.com/randomcoww/terraform-infra generated under `outputs/ignition`
```
sudo coreos-installer iso ignition embed \
  -i ../terraform-infra/resourcesv2/output/ignition/kvm-0.ign \
  -o kvm-0.iso \
  builds/latest/x86_64/fedora-coreos-*-live.x86_64.iso

sudo coreos-installer iso ignition embed \
  -i ../terraform-infra/resourcesv2/output/ignition/kvm-1.ign \
  -o kvm-1.iso \
  builds/latest/x86_64/fedora-coreos-*-live.x86_64.iso
```
Write kvm-*.iso to disk

Write to existing device

```
sudo coreos-installer iso ignition embed \
  -i ../terraform-infra/resourcesv2/output/ignition/kvm-0.ign \
  /dev/sdb --force

sudo coreos-installer iso ignition embed \
  -i ../terraform-infra/resourcesv2/output/ignition/kvm-1.ign \
  /dev/sdb --force
```
