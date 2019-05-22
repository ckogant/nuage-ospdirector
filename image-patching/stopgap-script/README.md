Steps:

Clone this repo onto the machine that is accessible to the nuage-rpms repo and make sure the machine also has `libguestfs-tools` installed.

```
yum install libguestfs-tools -y
git clone https://github.com/nuagenetworks/nuage-ospdirector.git
cd nuage-ospdirector
git checkout OSPD14
cd image-patching/stopgap-script/
```

Copy the overcloud-full.qcow2 from undercloud-director /home/stack/images/ to this location and make a backup of overcloud-full.qcow2

`cp overcloud-full.qcow2 overcloud-full-bk.qcow2`

Now run the below command by providing required values

### With GPG KEY   
 

Make sure to copy GPG-Key file to the same folder as "nuage_overcloud_full_patch.py" patching script location.

`python nuage_overcloud_full_patch.py --RhelUserName='<value>' --RhelPassword='<value>' --RhelPool=<pool-id> --RepoName=<value> --RepoBaseUrl=http://IP/reponame --ImageName='<value>' --Version=14 --RpmPublicKey='GPG-Key'`


### Without GPG Key

`python nuage_overcloud_full_patch.py --RhelUserName='<value>' --RhelPassword='<value>' --RhelPool=<pool-id> --RepoName=<value> --RepoBaseUrl=http://IP/reponame --ImageName='<value>' --Version=14 --no-signing-key`


This script takes in following input parameters:
  * RhelUserName is the user name for the RedHat Enterprise Linux subscription.
  * RhelPassword is the password for the RedHat Enterprise Linux subscription
  * RepoName is the name of the local repository hosting the Nuage RPMs.
  * RepoBaseUrl is the base URL for the repository hosting the Nuage RPMs (such as http://IP/reponame)
  * AVRSBaseUrl is the base URL for the repository hosting the 6Wind and AVRS RPMs (such as http://IP/reponame)
  * RhelPool is the RedHat Enterprise Linux pool to which the base packages are subscribed. instructions to get this can be found [here](https://access.redhat.com/documentation/en-us/red_hat_openstack_platform/14/html/director_installation_and_usage/preparing-for-director-installation#preparing-the-undercloud) in the 11th point.
  * ImageName is the name of the qcow2 image (for example, overcloud-full.qcow2)
  * Version is the OpenStack Platform director version (for Rocky, the version is 14).
  * RpmPublicKey is the GPG-Key file name
  * no-signing-key is 'Image patching proceeds with package signature verification disabled'

If image patching fails for some reason then remove the partially patched overcloud-full.qcow2 and create a copy of it from backup image before retrying image patching again.

```
rm overcloud-full.qcow2
cp overcloud-full-bk.qcow2 overcloud-full.qcow2
```

Once the patching is done successfully copy back the patched image to /home/stack/images/ on undercloud-director

Run the below commands:

```
[stack@director ~]$ source ~/stackrc
(undercloud) [stack@director ~]$ openstack image list
```

1. If above command returns null, run the below command
` (undercloud) [stack@director images]$ openstack overcloud image upload --image-path /home/stack/images/`

2. If it returns something like below
```
+--------------------------------------+------------------------+
| ID                                   | Name                   |
+--------------------------------------+------------------------+
| 765a46af-4417-4592-91e5-a300ead3faf6 | bm-deploy-ramdisk      |
| 09b40e3d-0382-4925-a356-3a4b4f36b514 | bm-deploy-kernel       |
| ef793cd0-e65c-456a-a675-63cd57610bd5 | overcloud-full         |
| 9a51a6cb-4670-40de-b64b-b70f4dd44152 | overcloud-full-initrd  |
| 4f7e33f4-d617-47c1-b36f-cbe90f132e5d | overcloud-full-vmlinuz |
+--------------------------------------+------------------------+
```

then run the below commad
```
(undercloud) [stack@director images]$ openstack overcloud image upload --update-existing --image-path /home/stack/images/
(undercloud) [stack@director images]$ openstack overcloud node configure $(openstack baremetal node list -c UUID -f value)
```
