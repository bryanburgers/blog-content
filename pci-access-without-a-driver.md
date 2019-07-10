At work recently, I had a new PCI device that I needed to experiment with. I
was dreading writing a Linux kernel driver to talk to it. It turns out, Linux
makes it possible to read and write to a PCI device's memory space without a
driver! Woohoo!

Linux provides a [sysfs interface to PCI devices][sysfs-pci]. From that
interface, the memory space can be `mmap`ed and then read and written. No
driver involved.

As a quick example, we can use `lspci` to get information about a particular
device.

```bash
$ vendor="10ee" # Use your device ID
$ device="7014" # Use your vendor ID
$ lspci -d $vendor:$device -nvv
04:00.0 1180: 10ee:7014
    ...
    Region 0: Memory at f7300000 (32-bit, non-prefetchable) [size=128K]
```

Then we can look at the sysfs interface, at `/sys/bus/pci/devices/`. The first
bit of data in the output of `lspci` gives the location of the device on the
bus, that we can use when traversing the sysfs interface.

```bash
$ ls -alF /sys/bus/pci/devices/0000\:04\:00.0/
total 0
drwxr-xr-x 3 root root      0 Jul  1 12:42 ./
drwxr-xr-x 8 root root      0 Jul  1 12:42 ../
-rw-r--r-- 1 root root   4096 Jul  9 12:48 broken_parity_status
-r--r--r-- 1 root root   4096 Jul  1 12:42 class
-rw-r--r-- 1 root root   4096 Jul  9 12:44 config
-r--r--r-- 1 root root   4096 Jul  1 12:42 device
...
-r--r--r-- 1 root root   4096 Jul  1 12:43 resource
-rw------- 1 root root 131072 Jul  1 12:43 resource0
...
-r--r--r-- 1 root root   4096 Jul  1 12:42 vendor
```

This interface has some useful files like `vendor` and `device` that confirm
that we have the right device. These are also useful for programatically
finding the correct device, rather than using `lspci`.

```
$ cat /sys/bus/pci/devices/0000\:04\:00.0/vendor
0x10ee
$ cat /sys/bus/pci/devices/0000\:04\:00.0/device
0x7014
```

Looking back at the `lspci` output, we can also find memory resources and
addresses. These are represented as `resource0`...`resourceN` in the sysfs
interface. That's what we use to get access to the PCI memory space.

Open the `resource0` file (which can be some number other than 0 depending on
the device).

```c
int fd = open("/sys/bus/pci/devices/0000:04:00.0/resource0", O_RDWR | O_SYNC);
```

Then use the memory address and size from the `lspci` output to `mmap` the
file.

```c
void* base_address = (void*)0xf7300000;
size_t size = 128 * 1024; // 128K
void* void_memory = mmap(base_address,
                         size,
                         PROT_READ | PROT_WRITE,
                         MAP_SHARED,
                         fd,
                         0);
uint16_t* memory = (uint16_t*)void_memory;
```

Now `memory` provides direct access to read and write the PCI memory space.
We can hack away!

```c
// Read the value of the first register
uint16_t first_register = memory[0];

// Write a value to the third register
memory[2] = 0x0007;
```

Now, this isn't the perfect scenario. For one, we need to be `root` to access
this memory space. For two, there's no sign of interrupt handling anywhere.

But for basic poking around on a new device, it works pretty slick. No kernel
module development required.

[sysfs-pci]: https://github.com/torvalds/linux/blob/master/Documentation/filesystems/sysfs-pci.txt
