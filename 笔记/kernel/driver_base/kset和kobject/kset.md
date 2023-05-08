
## 结构体
```c
struct kset {
	struct list_head list;
	spinlock_t list_lock;
	struct kobject kobj;
	const struct kset_uevent_ops *uevent_ops;
} __randomize_layout;

```

## 描述
-   list：表示 kset 对象在内核对象链表中的位置，用于管理和操作 kset 对象。
-   list_lock：表示kset的自旋锁，用于包含kset对象的访问。
-   kobj：表示 kset 对象对应的 kobject，包含了管理 kset 对象的各种属性和方法。
-   uevent_ops：表示 kset 的 uevent 操作，用于在 sysfs 文件系统中更新 kset 对象的属性。

## 常用方法

### 1. 初始化
```c
void kset_init(struct kset *k)
{
	kobject_init_internal(&k->kobj); // 初始化kset的obj对象
	INIT_LIST_HEAD(&k->list);
	spin_lock_init(&k->list_lock);
}
```
### 2. 初始化并注册
```c
int kset_register(struct kset *k)
{
	kset_init(k);
	err = kobject_add_internal(&k->kobj);
	kobject_uevent(&k->kobj, KOBJ_ADD);
	return 0;
}
// 注销
void kset_unregister(struct kset *k)；
```
### 3. 创建并注册
```c
struct kset *kset_create_and_add(const char *name,
				 const struct kset_uevent_ops *uevent_ops,
				 struct kobject *parent_kobj)
{
	struct kset *kset;
	int error;

	kset = kset_create(name, uevent_ops, parent_kobj);
	if (!kset)
		return NULL;
	error = kset_register(kset);
	if (error) {
		kfree(kset);
		return NULL;
	}
	return kset;
}

static struct kset *kset_create(const char *name,
				const struct kset_uevent_ops *uevent_ops,
				struct kobject *parent_kobj)
{
	struct kset *kset;
	int retval;

	kset = kzalloc(sizeof(*kset), GFP_KERNEL);
	if (!kset)
		return NULL;
	retval = kobject_set_name(&kset->kobj, "%s", name);
	if (retval) {
		kfree(kset);
		return NULL;
	}
	kset->uevent_ops = uevent_ops;
	kset->kobj.parent = parent_kobj;

	/*
	 * The kobject of this kset will have a type of kset_ktype and belong to
	 * no kset itself.  That way we can properly free it when it is
	 * finished being used.
	 */
	kset->kobj.ktype = &kset_ktype; // 这个结构体提供了属性读写的操作
	kset->kobj.kset = NULL;

	return kset;
}
```
### 4. to方法
```c
static inline struct kset *to_kset(struct kobject *kobj)
{
	return kobj ? container_of(kobj, struct kset, kobj) : NULL;
}
```
### 5. 查找kobject
用于根据名字查找kset下的kobject
```c
struct kobject *kset_find_obj(struct kset *kset, const char *name)
{
	struct kobject *k;
	struct kobject *ret = NULL;

	spin_lock(&kset->list_lock);

	list_for_each_entry(k, &kset->list, entry) {
		if (kobject_name(k) && !strcmp(kobject_name(k), name)) {
			ret = kobject_get_unless_zero(k);
			break;
		}
	}

	spin_unlock(&kset->list_lock);
	return ret;
}
```

### 6. 全局的kobject
```c
/* The global /sys/kernel/ kobject for people to chain off of */
extern struct kobject *kernel_kobj; // kernel/ksysfs.c

/* The global /sys/kernel/mm/ kobject for people to chain off of */
extern struct kobject *mm_kobj; // mm/mm_init.c

/* The global /sys/hypervisor/ kobject for people to chain off of */
extern struct kobject *hypervisor_kobj; // drivers/base/hypervisor.c

/* The global /sys/power/ kobject for people to chain off of */
extern struct kobject *power_kobj; // kernel/power/main.c

/* The global /sys/firmware/ kobject for people to chain off of */
extern struct kobject *firmware_kobj; // drivers/base/firmware.c
```