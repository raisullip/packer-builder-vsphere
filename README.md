# packer-builder-vsphere

## Installation instructions

1. It is supposed that you already have Go(and [Packer](https://github.com/hashicorp/packer)), [Docker-compose](https://docs.docker.com/compose/install/) and [Glide](https://github.com/Masterminds/glide) set.

1. Download the sourcces from [github.com/LizaTretyakova/packer-builder-vsphere](github.com/LizaTretyakova/packer-builder-vsphere)

1. `cd` to `$GOPATH/go/src/github.com/LizaTretyakova/packer-builder-vsphere` (or wherever it was downloaded)

1. Get the dependencies
```
$ glide install
```

5. Build the binaries
```
$ docker-compose run build
```

6. The template for this builder is like following:
```json
{
    "variables": {
            "url": "{{env `YOUR_VSPHERE_URL`}}",
            "username": "{{env `YOUR_VSPHERE_USERNAME`}}",
            "password": "{{env `YOUR_VSPHERE_PASSWORD`}}",
            "ssh_username": "{{env `TEMPLATE_VM_SSH_USERNAME`}}",
            "ssh_password": "{{env `TEMPLATE_VM_SSH_PASSWORD`}}",
            "dc_name": "{{env `TEMPLATE_VM_DATACENTER`}}",
            "template": "{{env `TEMPLATE_VM_NAME`}}",
            "host": "{{env `TARGET_HOST`}}"
    },
    "builders": [
        {
            "type": "vsphere",
            "url": "{{user `url`}}",
            "username": "{{user `username`}}",
            "password": "{{user `password`}}",
            "ssh_username": "{{user `ssh_username`}}",
            "ssh_password": "{{user `ssh_password`}}",
            "dc_name": "{{user `dc_name`}}",
            "template": "{{user `template`}}",
            "vm_name": "new_vm_name",
            "host": "{{user `host`}}",
            "resource_pool": "your_target_resource_pool",
            "datastore": "your_target_datastore",
            "RAM": "1024",
            "cpus": "2",
            "shutdown_command": "echo '{{user `ssh_password`}}' | sudo -S shutdown -P now"
        } 
    ],
    "provisioners": [
        {
              "type": "shell",
              "inline": ["echo foo"]
        }
    ]
}
```
where `vm_name`, `RAM`, `cpus` and `shutdown_command` are parameters of the new VM. 
Parameters `ssh_*`, `dc_name` (datacenter name) and `template` (the name of the base VM) are for the base VM, 
on which you are creating the new one (note that VMWare Tools should be already installed on this template machine).
`vm_name` and `host` (describe the name of the new VM and the name of the host where we want to create it) are required parameters; you can also specify `resource_pool` (if you don't, the builder will try to detect the default one) and `datastore` (**important**: if your target host differs from the initial one, you **have to** specify `datastore`; in case you stay within the same host, this parameter can be omitted). 
`url`, `username` and `password` are your vSphere parameters.
You need to set the appropriate values in the `variables` section before proceeding.

7. Now you can go to the `bin/` directory
```
$ cd ./bin
```
and try the builder
```
$ packer build template.json
```

## Builder parameters
I will repeat myself here a bit just to make the things clearer a bit.
### Required parameters:
* `username`
* `password`
* `template`
* `vm_name`
* `host`
### Optional parameters:
* Destination parameters:
    * `resource_pool`
    * `datastore` (but is required if you move between hosts)
* Hardware configuration:
    * `cpus`
    * `ram`
    * `shutdown_command`
* `ssh_username`
* `ssh_password`
* `dc_name` (source datacenter)

## Progress bar
You can find it [here](https://github.com/LizaTretyakova/packer-builder-vsphere/projects/1) as well.

- [x] hardware customization of the new VM (cpu, ram)
- [x] clone from template (not only from VM)
- [x] clone to alternate host, resource pool and datastore
- [ ] enable linked clones
- [ ] support Windows guest systems
- [ ] enable VM-to-template conversion
- [ ] tests
- [ ] add a shutdown timeout
- [ ] further hardware customization:
    * resize disks
    * ram reservation
    * cpu reservation