## Creating Single Node K8s Cluster

# Cluster for development
Recently I wanted learn kubernetes and it was difficult to find a good development environment which I could use to play with and recreate within seconds when I break it.

I came across rancher which can be deployed on docker as a single docker container and all persistent volume claims can be mapped to a docker volume so data can be preserved.

# Startup
To start the docker container I created a run.sh file with following contents

```
#!/bin/bash
docker run -d --restart=unless-stopped --network=host \
-v /home/manit/rancher/config:/var/lib/rancher \
-v /home/manit/rancher/ssd:/ssd \
-v /mnt/disk/rancher/hdd:/hdd \
--name rancher \
--privileged rancher/rancher:latest
```
I am using host networking above so that when I deploy a kubernetes deployment with a server port it gets available on the localhost (or server public IP address).

I have 2 disks attached to the system, and main disk is an SSD, so I map out 2 directories from host to docker container so that I can store data from PVC where I want.

If you do not want to use host networking, you can use port 80 and 443 and publish them.

# Cleanup
```
#!/bin/bash
docker container rm -f rancher
sudo rm -r ~/rancher/config ~/rancher/ssd /mnt/disk/rancher/hdd/
```
To save the data from kubernetes persistent volumes you may not remove ~/rancher/ssd and /mnt/disk/rancher/hdd.

This gave me an easy way to develop and clean out where something goes wrong.

My setup is running on old Lenovo Thinkpad which is currently connected to my home network.

## References
[Rancher Docs](https://rancher.com/docs/rancher/v2.5/en/installation/other-installation-methods/single-node-docker/)
