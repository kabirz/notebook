
## 结构体
```c
struct kobject {
	const char		*name;
	struct list_head	entry;
	struct kobject		*parent;
	struct kset		*kset;
	const struct kobj_type	*ktype;
	struct kernfs_node	*sd; /* sysfs directory entry */
	struct kref		kref;
#ifdef CONFIG_DEBUG_KOBJECT_RELEASE
	struct delayed_work	release;
#endif
	unsigned int state_initialized:1;
	unsigned int state_in_sysfs:1;
	unsigned int state_add_uevent_sent:1;
	unsigned int state_remove_uevent_sent:1;
	unsigned int uevent_suppress:1;
};
```
## 描述

1.  name：表示内核对象的名称，通常以字符串形式表示。
	获取name的方法：
	```c
	static inline const char *kobject_name(const struct kobject *kobj)
	{
		return kobj->name;
	}
	```
	 设置name的方法：
	 ```c
	 int kobject_set_name(struct kobject *kobj, const char *fmt, ...);
	 int kobject_set_name_vargs(struct kobject *kobj, const char *fmt, va_list vargs);
	 int kobject_rename(struct kobject *kobj, const char *new_name);
	 ```
2.  entry：表示 kobject 在内核对象链表中的位置，用于管理和操作 kobject 对象。
		entry 会被加到对应的set中
	```c
	# 两种情况
	# kobject
	int kobject_add(struct kobject *kobj, struct kobject *parent, const char *fmt, ...);
	int kobject_add_varg(struct kobject *kobj, struct kobject *parent,
			const char *fmt, va_list vargs);
	# kset
	int kset_register(struct kset *k)
	{
		......
		kset_init(k);
		err = kobject_add_internal(&k->kobj);
		......
		kobject_uevent(&k->kobj, KOBJ_ADD);
		return 0;
	}
	
	static int kobject_add_internal(struct kobject *kobj);
	/* add the kobject to its kset's list */
	static void kobj_kset_join(struct kobject *kobj)
	{
		......
		kset_get(kobj->kset);
		spin_lock(&kobj->kset->list_lock);
		list_add_tail(&kobj->entry, &kobj->kset->list); //=================
		spin_unlock(&kobj->kset->list_lock);
	}
	
	/* remove the kobject from its kset's list */
	static void kobj_kset_leave(struct kobject *kobj)
	{
		......
		spin_lock(&kobj->kset->list_lock);
		list_del_init(&kobj->entry); //=================
		spin_unlock(&kobj->kset->list_lock);
		kset_put(kobj->kset);
	}
	```
3.  parent：表示 kobject 的父对象，用于形成层次结构。
	如果有所属的set，默认是ket的object。也可以在kobject_add参数中指定
4. kset：表示 kobject 所属的 kset，用于管理和操作 kobject 对象。
5.  ktype：表示 kobject 的类型，包含 kobject 的属性(default_groups)和方法(sysfs_ops)。
	 ktype结构体
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
	 ktype会在kobject_init的时候设置。
	 例：
	 ```c
	 # drivers/base/core.c
	 void device_initialize(struct device *dev); --> kobject_init(&dev->kobj, &device_ktype);
	 static struct kobject *class_dir_create_and_add(struct class *class,
			 struct kobject *parent_kobj)；
	 ```
6.   sd：表示 kobject 的 sysfs 目录项，用于在 sysfs 文件系统中表示 kobject 对象。
	 通常会在
	 ```c
	 int sysfs_create_dir_ns(struct kobject *kobj, const void *ns)
	 ```
	 中设置
7.   kref：表示 kobject 的引用计数，用于管理和操作 kobject 对象的生命周期。
	 引用计算在
	 ```c
	 static void kobject_init_internal(struct kobject *kobj)；
	 // 增加引用
	 struct kobject *kobject_get(struct kobject *kobj)；
	 // 释放引用，如果计数为0，会调用kobject_release
	 void kobject_put(struct kobject *kobj)；
	 ```
	 中初始化
8.   state_initialized：表示 kobject 是否已经初始化。
9.   state_in_sysfs：表示 kobject 是否已经在 sysfs 文件系统中注册。
10.   state_add_uevent_sent：表示 kobject 是否已经发送了 add uevent。
         会在函数
         ```c
         int kobject_uevent_env(struct kobject *kobj, enum kobject_action action, 
	         char *envp_ext[]);
		 ```
		 action为`KOBJECT_ADD`的时候设置为1
11.   state_remove_uevent_sent：表示 kobject 是否已经发送了 remove uevent。
		 会在action为`KOBJ_REMOVE`的时候设置。
12.   uevent_suppress：表示是否需要抑制 uevent。
		设置为1的话，uevent将不会发出。

## 常用符号
### 1. 设置/设置name
```c
int kobject_set_name(struct kobject *kobj, const char *name, ...);
int kobject_set_name_vargs(struct kobject *kobj, const char *fmt, va_list vargs);
static inline const char *kobject_name(const struct kobject *kobj)；
```
### 2. 初始化(kobject_init)
```c
extern void kobject_init(struct kobject *kobj, struct kobj_type *ktype)
{
	kobject_init_internal(kobj);
	kobj->ktype = ktype;
}

static void kobject_init_internal(struct kobject *kobj)
{
	if (!kobj)
		return;
	kref_init(&kobj->kref);
	INIT_LIST_HEAD(&kobj->entry);
	kobj->state_in_sysfs = 0;
	kobj->state_add_uevent_sent = 0;
	kobj->state_remove_uevent_sent = 0;
	kobj->state_initialized = 1;
}
```
 初始化`kobject`结构体，指定`ktype`.
### 3. 添加功能(kobject_add)
 ```c
 int kobject_add(struct kobject *kobj, struct kobject *parent, const char *fmt, ...)
 {
	  va_start(args, fmt);
	  retval = kobject_add_varg(kobj, parent, fmt, args);
	  va_end(args);
 }
 static __printf(3, 0) int kobject_add_varg(struct kobject *kobj,
						 struct kobject *parent, const char *fmt, va_list vargs)
 {
		retval = kobject_set_name_vargs(kobj, fmt, vargs);
		kobj->parent = parent;
		return kobject_add_internal(kobj);
 }
 static int kobject_add_internal(struct kobject *kobj)
 {
	 parent = kobject_get(kobj->parent); // 增加父亲的引用计数
	 if (kobj->kset) {
		 kobj_kset_join(kobj); // 加入到kset列表
		 kobj->parent = parent;
	 }
	 error = create_dir(kobj); // 创建kobj目录和相关属性文件
 }
 static int create_dir(struct kobject *kobj)
 {
	 error = sysfs_create_dir_ns(kobj, kobject_namespace(kobj)); // 创建kobj目录
	 error = populate_dir(kobj); // populate directory with attributes. 创建kobj属性文件
	 if (ktype=get_ktype(kobj)) {
		 error = sysfs_create_groups(kobj, ktype->default_groups); // 创建所属kset属性文件
	 }
 }
 ```
### 4. 初始化和添加
 ```c
 int kobject_init_and_add(struct kobject *kobj, const struct kobj_type *ktype,
			 struct kobject *parent, const char *fmt, ...)
	{
		va_list args;
		int retval;
	
		kobject_init(kobj, ktype);
	
		va_start(args, fmt);
		retval = kobject_add_varg(kobj, parent, fmt, args);
		va_end(args);
	
		return retval;
	}
 ```
### 5. 删除
 ```c
 void kobject_del(struct kobject *kobj)
 {
	 parent = kobj->parent;
	 __kobject_del(kobj);
	 kobject_put(parent);
 }
 ```
### 6. 动态创建
 获取一个初始化了的kobject，并且用完之后无需手动释放，调用`kobject_del`使自动销毁。
 ```c
 struct kobject *kobject_create(void)
 {
	 kobj = kzalloc(sizeof(*kobj), GFP_KERNEL);
	 kobject_init(kobj, &dynamic_kobj_ktype);
	 return kobj;
 }
 # 创建并添加
 struct kobject *kobject_create_and_add(const char *name, struct kobject *parent)；
 ```
### 7. 改变
改变kobject的名字
```c
int kobject_rename(struct kobject *kobj, const char *new_name);

```
改变kobject的父亲
```c
int kobject_move(struct kobject *kobj, struct kobject *new_parent);
```