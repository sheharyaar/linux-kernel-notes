> **Note :** When working with `kref`, please refer to : [Kref Rules - Kernel Docs](https://docs.kernel.org/core-api/kref.html#kref-rules)  
### kobject  
- it’s an object of type `struct kobject`, used a lot in sysfs  
- One of the **key functions** of a kobject is to **serve as a reference counter** for the object in which it is embedded.  
> If all that you want to use a kobject for is to provide a reference counter for your structure, please use the struct kref instead; a kobject would be overkill.  
- It is rare for kernel code to create a standalone kobject, they are usually embedded within some other structure which contains the stuff the code is really interested in. There is an exception. (See [exception](Kobjects%20family.md#Exception) section below).  
> No structure should **EVER** have more than one kobject embedded within it. If it does, the reference counting for the object is sure to be messed up and incorrect  
  
- Example of kobject embedded in a struct  
```c  
struct uio_map {  
        struct kobject kobj;  
        struct uio_mem *mem;  
};  
```  
  
- To get the reference of the object that embeds kobject (uio_map in this case), **do not use any tricks**, use `container_of(ptr, type, member)` macro.   
- Example : `struct uio_map *u_map = container_of(kp, struct uio_map, kobj);` where ptr is the pointer to the kobject.   
- For convenience, programmers often define a simple macro for **back-casting** kobject pointers to the containing type. Eg : `#define to_map(map) container_of(map, struct uio_map, kobj)` and invoked as : `struct uio_map *map = to_map(kobj);`  
  
```c  
struct kobject {  
        const char              *name;  
        struct list_head        entry;  
        struct kobject          *parent;  
        struct kset             *kset;  
        const struct kobj_type  *ktype;  
        struct kernfs_node      *sd; /* sysfs directory entry */  
        struct kref             kref;  
  
        unsigned int state_initialized:1;  
        unsigned int state_in_sysfs:1;  
        unsigned int state_add_uevent_sent:1;  
        unsigned int state_remove_uevent_sent:1;  
        unsigned int uevent_suppress:1;  
  
#ifdef CONFIG_DEBUG_KOBJECT_RELEASE  
        struct delayed_work     release;  
#endif  
};  
  
```  
#### Precautions  
1. Because kobjects are dynamic, they must not be declared statically or on the stack, but instead, **always allocated dynamically**.  
2. Flow of initialisation is :    
	- `kobject_init()` → `kobject_add()`   **or** `kobject_init_and_add()`  
	- after initialisation, announce to the world using `kobject_uevent()`.  This should be done only after any attributes or children of the kobject have been initialized properly, as userspace will instantly start to look for them when this call happens.  
3. `kobject_set_name()` is **legacy** now name is used in kobject_add(). To **rename** use `kobject_rename()`  
> Remember to check return values for functions that are non-void return type. Errors are possible and handle them accordingly, else there will be race and other issues.  
#### Exception  
Sometimes all that a developer wants is a way to create a simple directory in the sysfs hierarchy, and not have to mess with the whole complication of ksets, show and store functions, and other details. This is the one exception where a single kobject should be created.  
  
`kobject_create_and_add()` : will create a kobject and place it in sysfs in the location underneath the specified parent kobject.  
  
To create simple attributes associated with this kobject, use: `sysfs_create_file()`  
  
#### krelease  
- The code must be notified **asynchronously** whenever the last reference to one of its kobjects goes away  
- This is done using kobject’s `release()` method  
```c  
void my_object_release(struct kobject *kobj)  
{  
        struct my_object *mine = container_of(kobj, struct my_object, kobj);  
  
        /* Perform any additional cleanup on this object, then... */  
        kfree(mine);  
}```  
  
> Note, the name of the kobject is available in the release function, but it must NOT be changed within this callback. Otherwise there will be a memory leak in the kobject core  
- the release() method is **not stored in the kobject** itself; instead, it is associated with the `ktype`  
### ktype  
- A ktype is the type of object that embeds a kobject. Every structure that embeds a kobject needs a corresponding ktype.  
```c  
struct kobj_type {  
        void (*release)(struct kobject *kobj);  
        const struct sysfs_ops *sysfs_ops;  
        const struct attribute_group **default_groups;  
        const struct kobj_ns_type_operations *(*child_ns_type)(struct kobject *kobj);  
        const void *(*namespace)(struct kobject *kobj);  
        void (*get_ownership)(struct kobject *kobj, kuid_t *uid, kgid_t *gid);  
};  
```  
- a pointer to this structure must be specified when you call [`kobject_init()`](https://docs.kernel.org/driver-api/basics.html#c.kobject_init "kobject_init") or [`kobject_init_and_add()`](https://docs.kernel.org/driver-api/basics.html#c.kobject_init_and_add "kobject_init_and_add").  
### ksets  
- kset is merely a collection of kobjects that want to be associated with each other  
- There is **no restriction** that they be of the same ktype, but be very careful if they are not.  
	- A kset can be used by the kernel to track “all block devices”   
	- A kset is also a subdirectory in sysfs, where the associated kobjects with the kset can show up.   
	- Ksets can support the “**hotplugging**” of kobjects and influence how uevent events are reported to user space. 	  
![Pasted image 20240405010214.png](../../output/Assets/Pasted%20image%2020240405010214.png)  
- For hotplugging there are operations that are invoked whenever a kobject enters or leaves the kset. They are able to determine whether a user-space hotplug event is generated for this change, and to affect how that event is presented.  
  
> For a more complete example of using ksets and kobjects properly, see the example programs `samples/kobject/{kobject-example.c,kset-example.c}`, which will be built as loadable modules if you select `CONFIG_SAMPLE_KOBJECT`.  
  
  
### References  
- [The zen of kobjects](https://lwn.net/Articles/51437/) (deprecated, theory valuable only – subsystems is not present in newer kernels 4+)  
- [Everything you never wanted to know about kobjects, ksets, and ktypes - Kernel Docs](https://docs.kernel.org/core-api/kobject.html)  
- [Adding reference counters (krefs) to kernel objects - Kernel Docs](https://docs.kernel.org/core-api/kref.html)  
