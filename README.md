# Istio Installer
This image is used by the [rancher-istio chart](https://github.com/rancher/charts "rancher-istio chart") to intall istio with various configurations.

This installer uses shell scripts to run istioctl install, upgrade and uninstall commands. 
## Architecture
Uses `istioctl` and `nginx` to complete install, upgrade and delete comands for the Istio service mesh. 

#### Overview
This image uses a series of bash scripts to execute `istioctl` binary comands. The `istioctl` binary is an Istio configuration command line utility for managing your Istio service mesh.

The current implementation of the `istioctl upgrade` command makes a call to external github to download Istio manifests to complete the upgrade process. This will fail when running in an airgapped enviornment. To temporarily resolve the issue, this image also uses an nginx server to intercept that call and download the necessary manifest from the docker container. The image bundles the Istio manifests for the versions of Istio that are currently released in a [rancher-istio chart](https://github.com/rancher/charts "rancher-istio chart").

## Making Changes
#### How to upgrade the istio version
In the Dockerfile, change the environmnet variable `ENV ISTIO_VERSION` to the version of istio that you would like the installer to support.

In addition to making the version change in the Dockerfile, you will need to update the `istio_version_array` in `scripts/fetch_istio_releases.sh` to support upgrades in airgapped environments. To do this, you can move the previous version from the `ENV ISTIO_VERSION` in the Dockerfile to the `istio_version_array`. If it already exists there, then its likely you don't need to add a new version to the array.

In general, you would want to ensure that all previous rancher-istio supported versions are present in the `istio_version_array`, in order from least to greatest. You would also want to ensure that any istio versions that were released with an out of band (OOB) rancher-istio release have been included. The easiest way to verify that all the correct versions are present is to validate the versions.txt against the versions that are in the release branches of the [rancher/charts ](https://github.com/rancher/charts "rancher/charts ") or againts the existing [istio-installer image tags](https://hub.docker.com/r/rancher/istio-installer/tags "istio-installer image tags")

> Note: The v`istio_version_array` does not include rancher-istio chart version, just the version of istio that is supported by the rancher-istio chart.

Finally, you need to validate that the create_istio_system.sh and the uninstall_istio_system.sh script commands are still valid. This can be done by checking that the [istioctl commands](https://istio.io/latest/docs/reference/commands/istioctl/ "istioctl commands") have not changed. You should also check the [release notes](https://istio.io/latest/news/ "release notes") for any significant changes to how istioctl is used. If there were changes to the commands, update the create_istio_system.sh  and the uninstall_istio_system.sh scripts accordingly.

## Build
The istio-installer image is built using drone when a new tag is created and pushed to repository. See drone.yaml for drone configurations.

When building the image locally, run `docker build -t <your_docker_hub_repo>/istio-installer:<tag> .`

####  How to tag
The istio installer uses the following version format for tags: `<x.x.x>-rancher<x>`

The `<x.x.x>` should be set to the version of istio the installer is going to be used for. This can be found in the Dockerfile as the value for the `ENV ISTIO_VERSION`

The `rancher<x>` is the build number. This number starts at 1 and increases with each re-tag with the same istio version.`

After all of the changes have been merged into the master branch, fetch a fresh copy of the master branch to your local machine. From there, you can run the following commands to tag and push the tag to the repo
```
git tag <x.x.x>-rancher<x>
git push <istio_installer_remote_branch> <x.x.x>-rancher<x>
```
Once the tag is pushed to github, drone will build the docker image and push it to the [rancher/istio-installer](https://hub.docker.com/r/rancher/istio-installer "rancher/istio-installer") docker hub repository.

**An example tag for a new istio version and push may look like this:**
```
git tag 1.9.6-rancher1
git push upstream 1.9.6-rancher1
```
If you need to re-tag the same version, that may look like this:
```
git tag 1.9.6-rancher2
git push upstream 1.9.6-rancher2
```
