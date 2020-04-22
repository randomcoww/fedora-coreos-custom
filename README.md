# fedora-coreos-custom

From upstream: https://github.com/coreos/coreos-assembler

```
cosa() {
   env | grep COREOS_ASSEMBLER
   set -x
   podman run --rm -ti --security-opt label=disable --privileged --cap-add CAP_SYS_ADMIN            \
              --uidmap=10000:0:1 --uidmap=0:1:10000 --uidmap 10001:10001:55536                      \
              -v ${PWD}:/srv/ --device /dev/kvm --device /dev/fuse                                  \
              --tmpfs /tmp -v /var/tmp:/var/tmp --name cosa                                         \
              ${COREOS_ASSEMBLER_CONFIG_GIT:+-v $COREOS_ASSEMBLER_CONFIG_GIT:/srv/src/config/:ro}   \
              ${COREOS_ASSEMBLER_GIT:+-v $COREOS_ASSEMBLER_GIT/src/:/usr/lib/coreos-assembler/:ro}  \
              ${COREOS_ASSEMBLER_CONTAINER_RUNTIME_ARGS}                                            \
              ${COREOS_ASSEMBLER_CONTAINER:-quay.io/coreos-assembler/coreos-assembler:latest} "$@"
   rc=$?; set +x; return $rc
}

cosa init https://github.com/randomcoww/fedora-coreos-custom.git

## Add ignition file
curl http://127.0.0.1:8080/ignition?ign=kvm-0 | jq '.ignition.version = "3.0.0"' | sudo tee src/config/resources/base.ign

## Add matchbox image
podman pull quay.io/poseidon/matchbox:latest
podman save --format oci-archive -o matchbox.tar quay.io/poseidon/matchbox:latest
sudo cp matchbox.tar src/config/resources

## Add flatcar images
pushd src/config/resources
sudo curl -LO https://edge.release.flatcar-linux.net/amd64-usr/current/flatcar_production_pxe.vmlinuz
sudo curl -LO https://edge.release.flatcar-linux.net/amd64-usr/current/flatcar_production_pxe_image.cpio.gz
popd

cosa clean && cosa fetch && cosa build && cosa buildextend-live
```
