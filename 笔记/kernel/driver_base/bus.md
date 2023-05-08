

## bus register
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
```

### subsys_system_register
