
源文件在：drivers/base/driver.c
提供了一些驱动的方法供其他驱动调用
## 迭代处理
查看驱动下的每一设备并执行`fn`操作
```c
int driver_for_each_device(struct device_driver *drv, struct device *start,
			   void *data, int (*fn)(struct device *, void *))
{
	struct klist_iter i;
	struct device *dev;
	int error = 0;

	klist_iter_init_node(&drv->p->klist_devices, &i,  start ? &start->p->knode_driver : NULL);
	while (!error && (dev = next_device(&i)))
		error = fn(dev, data);
	klist_iter_exit(&i);
	return error;
}
```

## 查找并匹配设备
```c
struct device *driver_find_device(struct device_driver *drv,
				  struct device *start, const void *data,
				  int (*match)(struct device *dev, const void *data))
{
	struct klist_iter i;
	struct device *dev;

	klist_iter_init_node(&drv->p->klist_devices, &i,(start ? &start->p->knode_driver : NULL));
	while ((dev = next_device(&i)))
		if (match(dev, data) && get_device(dev))
			break;
	klist_iter_exit(&i);
	return dev;
}
```

## driver属性(组)创建销毁
```c
int driver_create_file(struct device_driver *drv, const struct driver_attribute *attr)
{
	int error;

	if (drv)
		error = sysfs_create_file(&drv->p->kobj, &attr->attr);
	else
		error = -EINVAL;
	return error;
}

void driver_remove_file(struct device_driver *drv, const struct driver_attribute *attr)
{
	if (drv)
		sysfs_remove_file(&drv->p->kobj, &attr->attr);
}

int driver_add_groups(struct device_driver *drv, const struct attribute_group **groups)
{
	return sysfs_create_groups(&drv->p->kobj, groups);
}

void driver_remove_groups(struct device_driver *drv, const struct attribute_group **groups)
{
	sysfs_remove_groups(&drv->p->kobj, groups);
}
```

## 驱动注册/注销
```c
int driver_register(struct device_driver *drv)
{
	int ret;

	ret = bus_add_driver(drv);
	ret = driver_add_groups(drv, drv->groups);
	kobject_uevent(&drv->p->kobj, KOBJ_ADD);
	deferred_probe_extend_timeout();

	return ret;
}
void driver_unregister(struct device_driver *drv)；
```

## 找到驱动
```c
struct device_driver *driver_find(const char *name, struct bus_type *bus)
{
	struct kobject *k = kset_find_obj(bus->p->drivers_kset, name);
	struct driver_private *priv;

	if (k) {
		/* Drop reference added by kset_find_obj() */
		kobject_put(k);
		priv = to_driver(k);
		return priv->driver;
	}
	return NULL;
}
```