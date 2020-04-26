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
```

Build container VM image
```
pushd src/config
sudo ln -s manifest-containerd.yaml manifest.yaml
popd

cosa clean && cosa fetch && cosa build && cosa buildextend-live

cp builds/latest/x86_64/fedora-coreos-*.dev.0-live-kernel-x86_64 \
   src/config/resources/fedora-coreos-live-kernel
cp builds/latest/x86_64/fedora-coreos-*.dev.0-live-initramfs.x86_64.img \
   src/config/resources/fedora-coreos-live-initramfs.img
```

Build KVM hypervisor image
```
pushd src/config
sudo ln -s manifest-kvm.yaml manifest.yaml
popd
```

Add ignition file from https://github.com/randomcoww/terraform-infra 
```
curl http://127.0.0.1:8080/ignition?ign=kvm-0 \
   | sudo tee src/config/overlay.d/10custom/usr/lib/dracut/modules.d/40ignition-conf/base.ign
```

Add matchbox image
```
podman pull quay.io/poseidon/matchbox:latest
podman save --format oci-archive -o matchbox.tar quay.io/poseidon/matchbox:latest
sudo mv matchbox.tar src/config/resources
```

Run build
```
cosa clean && cosa fetch && cosa build && cosa buildextend-live
```