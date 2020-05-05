# install-omv
OpenMediaVault Installation (Including unsupported and not recommended)


Procedures to do an OpenMediaVault installation including prodedures both unsupported (on a RAID protected filesystem and sharing os space with data) and not recommended (using and USB flash drive).

![](/images/omv_logo.png)
OpenMediaVault is available at  https://www.openmediavault.org/ and installation procedure at https://openmediavault.readthedocs.io/en/5.x/installation/index.html. I will be using version 5.x and only specifying here the steps which are not reflected in the documentation.


* [References & Credits](#references-credits)
* [Summary](#summary)



### Format Example

- Objetive: Ansible Phases 2 optimizes compliance and operation.

> Go to slide 25

__**GO BACK TO THE FIRST K8S CLUSTER (rhel3)**__ and run the below command:

```shell
./19_iaac_ansible_day2.sh
```


## References & Credits

This document is based on the following resources. I really thank the authors for sharing their knowledge and trouble:
Installing OpenMediaVault on RAID-1 array - https://lazic.info/josip/post/installing-openmediavault-on-raid-device/


## Summary

- Objetive: Cover OpenMediaVault unsupported and not recommended installation procedures, and hopefully save you some time!
