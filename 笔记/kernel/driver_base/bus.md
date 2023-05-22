## 结构体
```c
struct bus_type {
	const char		*name;
	const char		*dev_name;
	struct device		*dev_root;
	const struct attribute_group **bus_groups;
	const struct attribute_group **dev_groups;
	const struct attribute_group **drv_groups;

	int (*match)(struct device *dev, struct device_driver *drv);
	int (*uevent)(struct device *dev, struct kobj_uevent_env *env);
	int (*probe)(struct device *dev);
	void (*sync_state)(struct device *dev);
	void (*remove)(struct device *dev);
	void (*shutdown)(struct device *dev);

	int (*online)(struct device *dev);
	int (*offline)(struct device *dev);

	int (*suspend)(struct device *dev, pm_message_t state);
	int (*resume)(struct device *dev);

	int (*num_vf)(struct device *dev);

	int (*dma_configure)(struct device *dev);
	void (*dma_cleanup)(struct device *dev);

	const struct dev_pm_ops *pm;

	const struct iommu_ops *iommu_ops;

	struct subsys_private *p;
	struct lock_class_key lock_key;

	bool need_parent_lock;
};

```
1. `name`是总线名字
2. `dev_name` device的父亲
3. `bus_groups`总线属性组，在总线注册([[#bus register]])时添加
4. `dev_gropus`设备默认属性组， 在总线添加设备[[#bus_add_device]]时添加
5. `drv_groups`总线所属驱动默认属性组，在总线添加驱动[[#bus_add_driver]]时添加
6. `macth`回调函数，在总线bind时调用
7. `uevent`回调函数，在事件改变时调用
8. `probe`回调函数，在设备驱动probe的时候调用
9. `remove`回调函数，在驱动remove的时候调用


## bus register
注册总线
```c
int bus_register(struct bus_type *bus)
{
	......
	priv = kzalloc(sizeof(struct subsys_private), GFP_KERNEL);
	priv->bus = bus;
	bus->p = priv;

	BLOCKING_INIT_NOTIFIER_HEAD(&priv->bus_notifier);

	retval = kobject_set_name(&priv->subsys.kobj, "%s", bus->name);

	priv->subsys.kobj.kset = bus_kset;
	priv->subsys.kobj.ktype = &bus_ktype;
	priv->drivers_autoprobe = 1;

	retval = kset_register(&priv->subsys);
	if (retval)
		goto out;

	retval = bus_create_file(bus, &bus_attr_uevent);
	if (retval)
		goto bus_uevent_fail;

	priv->devices_kset = kset_create_and_add("devices", NULL, &priv->subsys.kobj);
	priv->drivers_kset = kset_create_and_add("drivers", NULL, &priv->subsys.kobj);

	INIT_LIST_HEAD(&priv->interfaces);
	__mutex_init(&priv->mutex, "subsys mutex", key);
	klist_init(&priv->klist_devices, klist_devices_get, klist_devices_put);
	klist_init(&priv->klist_drivers, NULL, NULL);

	retval = add_probe_files(bus);
	retval = bus_add_groups(bus, bus->bus_groups);

	pr_debug("bus: '%s': registered\n", bus->name);
	return 0;

}
// 总线注销
void bus_unregister(struct bus_type *bus);
```
例子：
```c
struct bus_type i2c_bus_type = {
	.name		= "i2c",
	.match		= i2c_device_match,
	.probe		= i2c_device_probe,
	.remove		= i2c_device_remove,
	.shutdown	= i2c_device_shutdown,
};
static int __init i2c_init(void)
{
	int retval;

	retval = of_alias_get_highest_id("i2c");
	retval = bus_register(&i2c_bus_type);

	is_registered = true;
	retval = i2c_add_driver(&dummy_driver);

	return 0;
}

```

## 迭代查找和创建属性文件和class基本相同

## subsys_system_register


## bus_add_device
```c
int bus_add_device(struct device *dev)
{
	struct bus_type *bus = bus_get(dev->bus);
	int error = 0;

	if (bus) {
		pr_debug("bus: '%s': add device %s\n", bus->name, dev_name(dev));
		error = device_add_groups(dev, bus->dev_groups);
		error = sysfs_create_link(&bus->p->devices_kset->kobj,&dev->kobj, dev_name(dev));
		error = sysfs_create_link(&dev->kobj,&dev->bus->p->subsys.kobj, "subsystem");
		klist_add_tail(&dev->p->knode_bus, &bus->p->klist_devices);
	}
	return 0;
}
```

## bus_add_driver
```c
int bus_add_driver(struct device_driver *drv)
{
	struct bus_type *bus;
	struct driver_private *priv;
	int error = 0;

	bus = bus_get(drv->bus);

	pr_debug("bus: '%s': add driver %s\n", bus->name, drv->name);

	priv = kzalloc(sizeof(*priv), GFP_KERNEL);
	klist_init(&priv->klist_devices, NULL, NULL);
	priv->driver = drv;
	drv->p = priv;
	priv->kobj.kset = bus->p->drivers_kset;
	error = kobject_init_and_add(&priv->kobj, &driver_ktype, NULL, "%s", drv->name);

	klist_add_tail(&priv->knode_bus, &bus->p->klist_drivers);
	if (drv->bus->p->drivers_autoprobe) {
		error = driver_attach(drv);
		if (error)
			goto out_del_list;
	}
	module_add_driver(drv->owner, drv);

	error = driver_create_file(drv, &driver_attr_uevent);
	error = driver_add_groups(drv, bus->drv_groups);

	if (!drv->suppress_bind_attrs) {
		error = add_bind_files(drv);
	}

	return 0;
}
```