在 Linux 内核中，class 结构体是用于表示一组相关设备的抽象结构体。class 结构体中包含了一系列指向函数的指针，这些函数指针可以被用于管理和访问该设备类中的各种设备。

class 结构体通常被用于实现设备驱动程序。设备驱动程序可以将自己注册到某个设备类中，以表示自己属于该设备类。例如，一个 USB 设备驱动程序可以将自己注册到 USB 设备类中，以表示自己是一个 USB 设备。

## 结构体
```c
struct class {
	const char		*name;
	struct module		*owner;

	const struct attribute_group	**class_groups;
	const struct attribute_group	**dev_groups;
	struct kobject			*dev_kobj;

	int (*dev_uevent)(struct device *dev, struct kobj_uevent_env *env);
	char *(*devnode)(struct device *dev, umode_t *mode);

	void (*class_release)(struct class *class);
	void (*dev_release)(struct device *dev);

	int (*shutdown_pre)(struct device *dev);

	const struct kobj_ns_type_operations *ns_type;
	const void *(*namespace)(struct device *dev);

	void (*get_ownership)(struct device *dev, kuid_t *uid, kgid_t *gid);

	const struct dev_pm_ops *pm;

	struct subsys_private *p;
};
```
## 参数解释
- `name` 表示该设备类的名称
- `owner` 表示该设备类所属的模块
- `class_attrs`和`dev_attrs` 分别表示该设备类、该设备类中的某个设备和该设备类中某个设备的属性；
	1. `class_attrs`会在`class_register`的时候添加
	2. `dev_attrs`会在`device_add_attrs`的时候添加
- `dev_uevent` 函数指针表示发生uevent时的回调函数；
- `devnode`函数指针表示可以提供获取设备节点名字的回调函数
- `class_release`函数指针表示在`class_release`的时候调用
- `dev_release`函数指针表示在`device_release`的时候调用
- `namespace` 函数指针表示该设备类的命名空间；
- `pm` 表示该设备类的电源管理相关操作；
- `iommu_ops` 表示该设备类的 IOMMU 相关操作；`p` 表示该设备类的私有数据。

## class_register
完成class的注册，添加class 属性组
```c
#define class_register(class)			\
({						\
	static struct lock_class_key __key;	\
	__class_register(class, &__key);	\
})
int __class_register(struct class *cls, struct lock_class_key *key)
{
	struct subsys_private *cp;
	int error;

	pr_debug("device class '%s': registering\n", cls->name);

	cp = kzalloc(sizeof(*cp), GFP_KERNEL);
	klist_init(&cp->klist_devices, klist_class_dev_get, klist_class_dev_put);
	INIT_LIST_HEAD(&cp->interfaces);
	kset_init(&cp->glue_dirs);
	
	__mutex_init(&cp->mutex, "subsys mutex", key);
	error = kobject_set_name(&cp->subsys.kobj, "%s", cls->name);
	/* set the default /sys/dev directory for devices of this class */
	if (!cls->dev_kobj)
		cls->dev_kobj = sysfs_dev_char_kobj;

	cp->subsys.kobj.kset = class_kset;
	cp->subsys.kobj.ktype = &class_ktype;
	cp->class = cls;
	cls->p = cp;

	error = kset_register(&cp->subsys);
	error = class_add_groups(class_get(cls), cls->class_groups);
	class_put(cls);
	return error;
}
// class的注销操作
void class_unregister(struct class *class);
```

## 迭代查找
### 初始化迭代器
```c
void class_dev_iter_init(struct class_dev_iter *iter, struct class *class,
			 struct device *start, const struct device_type *type)
{
	struct klist_node *start_knode = NULL;

	if (start)
		start_knode = &start->p->knode_class;
	klist_iter_init_node(&class->p->klist_devices, &iter->ki, start_knode);
	iter->type = type;
}
```
### 迭代下一个设备
```c
struct device *class_dev_iter_next(struct class_dev_iter *iter)
{
	struct klist_node *knode;
	struct device *dev;

	while (1) {
		knode = klist_next(&iter->ki);
		if (!knode)
			return NULL;
		dev = klist_class_to_dev(knode);
		if (!iter->type || iter->type == dev->type)
			return dev;
	}
}
```

### 完成迭代
```c
void class_dev_iter_exit(struct class_dev_iter *iter)
{
	klist_iter_exit(&iter->ki);
}
```

### 迭代查找设备
对每一个设备调用指定回调函数, `fn`是回调函数，当返回值不为0时为止
```c
int class_for_each_device(struct class *class, struct device *start,
			  void *data, int (*fn)(struct device *, void *))
{
	struct class_dev_iter iter;
	struct device *dev;
	int error = 0;

	class_dev_iter_init(&iter, class, start, NULL);
	while ((dev = class_dev_iter_next(&iter))) {
		error = fn(dev, data);
		if (error)
			break;
	}
	class_dev_iter_exit(&iter);

	return error;
}
```

### 查找设备
#### 通用方法
```c
struct device *class_find_device(struct class *class, struct device *start,
				 const void *data,
				 int (*match)(struct device *, const void *))
{
	struct class_dev_iter iter;
	struct device *dev;

	class_dev_iter_init(&iter, class, start, NULL);
	while ((dev = class_dev_iter_next(&iter))) {
		if (match(dev, data)) {
			get_device(dev);
			break;
		}
	}
	class_dev_iter_exit(&iter);

	return dev;
}
```
#### 指定match方法
##### 设备名查找
```c
static inline struct device *class_find_device_by_name(struct class *class,
						       const char *name)
{
	return class_find_device(class, NULL, name, device_match_name);
}
int device_match_name(struct device *dev, const void *name)
{
	return sysfs_streq(dev_name(dev), name);
}
```
##### 设备树节点查找
```c
static inline struct device *
class_find_device_by_of_node(struct class *class, const struct device_node *np)
{
	return class_find_device(class, NULL, np, device_match_of_node);
}
int device_match_of_node(struct device *dev, const void *np)
{
	return dev->of_node == np;
}
```
##### fwnode查找
```c
static inline struct device *
class_find_device_by_fwnode(struct class *class,
			    const struct fwnode_handle *fwnode)
{
	return class_find_device(class, NULL, fwnode, device_match_fwnode);
}
int device_match_fwnode(struct device *dev, const void *fwnode)
{
	return dev_fwnode(dev) == fwnode;
}
```
##### 设备号查找
```c
static inline struct device *class_find_device_by_devt(struct class *class,
						       dev_t devt)
{
	return class_find_device(class, NULL, &devt, device_match_devt);
}
int device_match_devt(struct device *dev, const void *pdevt)
{
	return dev->devt == *(dev_t *)pdevt;
}
```
##### acpi 设备查找
```c
static inline struct device *
class_find_device_by_acpi_dev(struct class *class, const struct acpi_device *adev)
{
	return class_find_device(class, NULL, adev, device_match_acpi_dev);
}
int device_match_acpi_dev(struct device *dev, const void *adev)
{
	return ACPI_COMPANION(dev) == adev;
}
```

## 属性和属性文件创建
```c
struct class_attribute {
	struct attribute attr;
	ssize_t (*show)(struct class *class, struct class_attribute *attr,
			char *buf);
	ssize_t (*store)(struct class *class, struct class_attribute *attr,
			const char *buf, size_t count);
};

#define CLASS_ATTR_RW(_name) struct class_attribute class_attr_##_name = __ATTR_RW(_name)
#define CLASS_ATTR_RO(_name) struct class_attribute class_attr_##_name = __ATTR_RO(_name)
#define CLASS_ATTR_WO(_name) struct class_attribute class_attr_##_name = __ATTR_WO(_name)

int class_create_file_ns(struct class *class, const struct class_attribute *attr,
				const void *ns);
void class_remove_file_ns(struct class *class, const struct class_attribute *attr,
				const void *ns);

int class_create_file(struct class *class,const struct class_attribute *attr)
{
	return class_create_file_ns(class, attr, NULL);
}

void class_remove_file(struct class *class, const struct class_attribute *attr)
{
	return class_remove_file_ns(class, attr, NULL);
}

```

## 动态创建和销毁
```c
struct class *__class_create(struct module *owner, const char *name,
			     struct lock_class_key *key)
{
	struct class *cls;
	int retval;

	cls = kzalloc(sizeof(*cls), GFP_KERNEL);

	cls->name = name;
	cls->owner = owner;
	cls->class_release = class_create_release;

	retval = __class_register(cls, key);
	return cls;

}

void class_destroy(struct class *cls);

#define class_create(owner, name)		\
({						\
	static struct lock_class_key __key;	\
	__class_create(owner, name, &__key);	\
})
```