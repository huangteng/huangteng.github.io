title: Character Driver Note
date: 
tags: Linux driver
toc: true
---


This post explains how a char driver is developed and used in Linux Embeded System. The reference document is from Linux Driver Development Edition 3. The example is scull driver.


# How to access char devices

## Char devices file nodes

A character devices are accessed through names in the filesystem. Those names are called special files or device files or simply nodes of the filesystem tree (located in /dev directory), as an example below:

![](/images/dev_index.png)

The first letter "c" represents char devices, and 251 is the major number, the next columne 0,1,2,3 to 80 is the minor number. Here all the "scull*" devices have the same major number, which means all of them are managed by the driver 251; while the minor number is used to determine exactly which "physical" device is being referred to.

## Method to create device files

### Dynamic Allocation of Major Numbers

As we know, the command "insmod" is used to install the new modules to the kernel, during kernel will assign a major number to the new drivers, the information is saved in file /proc/devices as an example below:

![](/images/proc_devices.png)

The new device "scull", "scullp", "sculla" are assigned with the same major number 251. Once the major numbers are assigned to these new devices in file /proc/devices, the script "scull_load" or "scull.init" ("create_files" in function load_devices can make the device nodes and assign them with the same number). 

### Function to obtain the device numbers

Here is the scull driver example:

``` bash
/*
 * Get a range of minor numbers to work with, asking for a dynamic
 * major unless directed otherwise at load time.
 */
if (scull_major) {
	dev = MKDEV(scull_major, scull_minor);
	result = register_chrdev_region(dev, scull_nr_devs, "scull");
} else {
	result = alloc_chrdev_region(&dev, scull_minor, scull_nr_devs,
			"scull");
	scull_major = MAJOR(dev);
}
```
_<b>When the device number is pre-defined</b>_, this method is used to register new devices

int <b><i>register_chrdev_region</i></b>(dev_t first, unsigned int count, char *name);

* first: the beggining device number of the range to allocate
* count: the totoal number of contiguous device numbers to request
* name: the name of devices

_<b>When you want the device major number to be randomly allocated</b>_, this method is used:


int <b><i>alloc_chrdev_region</i></b>(dev_t *dev, unsigned int firstminor, unsigned int count, char *name);

* *dev: output-only parameter which is used to hold the 1st number in your allocated range
* firstminor: the first minor number to use
* count and name is the same as the above method

### Function to unregister the devices

void <b><i>unregister_chrdev_region(dev_t first, unsigned int count)</i></b>;


### script to create related device file nodes

Here is example from scull.init script in function *load_device()*

* Step 1: extract the device major number from /proc/devices

``` bash
MAJOR=`awk "\\$2==\"$DEVICE\" {print \\$1}" /proc/devices`
```

* Step 2: create file nodes

``` bash
# function create_files
cd /dev
local devlist=""
local file
while true; do
if [ $# -lt 2 ]; then break; fi
file="${DEVICE}$1"
mknod $file c $MAJOR $2
devlist="$devlist $file"
shift 2
done
```

Through debugging, the parameters passed into this function is:
``` bash
0 0 1 1 2 2 3 3 priv 16 pipe0 32 pipe1 33 pipe2 34 pipe3 35 single 48 uid 64 wuid 80
```