加载acpi模块时执行`acpi_init`，代码在`drivers/acpi/bus.c`
```c
static int __init acpi_init(void)
{
	int result;

	if (acpi_disabled) {
		pr_info("Interpreter disabled.\n");
		return -ENODEV;
	}

	acpi_kobj = kobject_create_and_add("acpi", firmware_kobj);
	if (!acpi_kobj)
		pr_debug("%s: kset create error\n", __func__);

	init_prmt();
	result = acpi_bus_init();
	if (result) {
		kobject_put(acpi_kobj);
		disable_acpi();
		return result;
	}

	pci_mmcfg_late_init();
	acpi_iort_init();
	acpi_viot_early_init();
	acpi_hest_init();
	ghes_init();
	acpi_scan_init(); // =============================
	acpi_ec_init();
	acpi_debugfs_init();
	acpi_sleep_proc_init();
	acpi_wakeup_device_init();
	acpi_debugger_init();
	acpi_setup_sb_notify_handler();
	acpi_viot_init();
	return 0;
}

subsys_initcall(acpi_init);
```

`acpi_scan_init`函数(在`drivers/acpi/scan.c`)进行枚举设备
```c

int __init acpi_scan_init(void)
{
	int result;
	acpi_status status;
	struct acpi_table_stao *stao_ptr;

.....

	acpi_scan_add_handler(&generic_device_handler);

......

	/*
	 * Although we call __add_memory() that is documented to require the
	 * device_hotplug_lock, it is not necessary here because this is an
	 * early code when userspace or any other code path cannot trigger
	 * hotplug/hotunplug operations.
	 */
	mutex_lock(&acpi_scan_lock);
	/*
	 * Enumerate devices in the ACPI namespace.
	 */
	result = acpi_bus_scan(ACPI_ROOT_OBJECT); //========================
	if (result)
		goto out;
......

 out:
	mutex_unlock(&acpi_scan_lock);
	return result;
}
```
`acpi_bus_scan`这个函数会创建acpi device和platform device
```c
int acpi_bus_scan(acpi_handle handle)
{
	struct acpi_device *device = NULL;

	acpi_bus_scan_second_pass = false;

	/* Pass 1: Avoid enumerating devices with missing dependencies. */

	if (ACPI_SUCCESS(acpi_bus_check_add(handle, true, &device)))
		acpi_walk_namespace(ACPI_TYPE_ANY, handle, ACPI_UINT32_MAX,
				    acpi_bus_check_add_1, NULL, NULL,
				    (void **)&device); // ============== 1

	if (!device)
		return -ENODEV;

	acpi_bus_attach(device, true); // ============== 2

	if (!acpi_bus_scan_second_pass)
		return 0;

	/* Pass 2: Enumerate all of the remaining devices. */

	device = NULL;

	if (ACPI_SUCCESS(acpi_bus_check_add(handle, false, &device)))
		acpi_walk_namespace(ACPI_TYPE_ANY, handle, ACPI_UINT32_MAX,
				    acpi_bus_check_add_2, NULL, NULL,
				    (void **)&device);

	acpi_bus_attach(device, false);

	return 0;
}
EXPORT_SYMBOL(acpi_bus_scan);
```
`acpi_bus_check_add_1`函数创建acpi_device
```c
static acpi_status acpi_bus_check_add(acpi_handle handle, bool check_dep,
				      struct acpi_device **adev_p)
{
	struct acpi_device *device = NULL;
	acpi_object_type acpi_type;
	int type;

......

	/*
	 * If check_dep is true at this point, the device has no dependencies,
	 * or the creation of the device object would have been postponed above.
	 */
	acpi_add_single_object(&device, handle, type, !check_dep); //============
	if (!device)
		return AE_CTRL_DEPTH;

	acpi_scan_init_hotplug(device);

out:
	if (!*adev_p)
		*adev_p = device;

	return AE_OK;
}
```
`acpi_add_single_object`
```c
static int acpi_add_single_object(struct acpi_device **child,
				  acpi_handle handle, int type, bool dep_init)
{
	struct acpi_device *device;
	bool release_dep_lock = false;
	int result;

	device = kzalloc(sizeof(struct acpi_device), GFP_KERNEL);
	if (!device)
		return -ENOMEM;

......
	if (!result)
		result = __acpi_device_add(device, acpi_device_release);//===========

	if (result) {
		acpi_device_release(&device->dev);
		return result;
	}

	acpi_power_add_remove_device(device, true);
	acpi_device_add_finalize(device);

	acpi_handle_debug(handle, "Added as %s, parent %s\n",
			  dev_name(&device->dev), device->parent ?
				dev_name(&device->parent->dev) : "(null)");

	*child = device;
	return 0;
}
```
`__acpi_device_add`
```c
static int __acpi_device_add(struct acpi_device *device,
			     void (*release)(struct device *))
{
	struct acpi_device_bus_id *acpi_device_bus_id;
	int result;
......
	device->dev.bus = &acpi_bus_type;
	device->dev.release = release;
	result = device_add(&device->dev); // ==========================
	if (result) {
		dev_err(&device->dev, "Error registering device\n");
		goto err;
	}
......
	return result;
}
```

而这个函数`acpi_bus_attach`则会创建platform device
```c
static void acpi_bus_attach(struct acpi_device *device, bool first_pass)
{
	struct acpi_device *child;
	bool skip = !first_pass && device->flags.visited;
	acpi_handle ejd;
	int ret;
......
	if (device->pnp.type.platform_id || device->flags.enumeration_by_parent)
		acpi_default_enumeration(device); // ==================
	else
		acpi_device_set_enumerated(device);
.......
}
```
`acpi_default_enumeration`
```c
static void acpi_default_enumeration(struct acpi_device *device)
{
	/*
	 * Do not enumerate devices with enumeration_by_parent flag set as
	 * they will be enumerated by their respective parents.
	 */
	if (!device->flags.enumeration_by_parent) {
		acpi_create_platform_device(device, NULL); // =====================
		acpi_device_set_enumerated(device);
	} else {
		blocking_notifier_call_chain(&acpi_reconfig_chain,
					     ACPI_RECONFIG_DEVICE_ADD, device);
	}
}
```
`acpi_create_platform_device`函数在`drivers/acpi/acpi_platform.c`中定义
```c
struct platform_device *acpi_create_platform_device(struct acpi_device *adev,
					struct property_entry *properties)
{
	struct platform_device *pdev = NULL;
	struct platform_device_info pdevinfo;
	struct resource_entry *rentry;
	struct list_head resource_list;
	struct resource *resources = NULL;
	int count;

	/* If the ACPI node already has a physical device attached, skip it. */
	if (adev->physical_node_count)
		return NULL;

	if (!acpi_match_device_ids(adev, forbidden_id_list))
		return ERR_PTR(-EINVAL);

	INIT_LIST_HEAD(&resource_list);
	count = acpi_dev_get_resources(adev, &resource_list, NULL, NULL);
	if (count < 0) {
		return NULL;
	} else if (count > 0) {
		resources = kcalloc(count, sizeof(struct resource),
				    GFP_KERNEL);
		if (!resources) {
			dev_err(&adev->dev, "No memory for resources\n");
			acpi_dev_free_resource_list(&resource_list);
			return ERR_PTR(-ENOMEM);
		}
		count = 0;
		list_for_each_entry(rentry, &resource_list, node)
			acpi_platform_fill_resource(adev, rentry->res,
						    &resources[count++]);

		acpi_dev_free_resource_list(&resource_list);
	}

	memset(&pdevinfo, 0, sizeof(pdevinfo));
	/*
	 * If the ACPI node has a parent and that parent has a physical device
	 * attached to it, that physical device should be the parent of the
	 * platform device we are about to create.
	 */
	pdevinfo.parent = adev->parent ?
		acpi_get_first_physical_node(adev->parent) : NULL;
	pdevinfo.name = dev_name(&adev->dev);
	pdevinfo.id = -1;
	pdevinfo.res = resources;
	pdevinfo.num_res = count;
	pdevinfo.fwnode = acpi_fwnode_handle(adev); // ============ 
	pdevinfo.properties = properties; // =====================

	if (acpi_dma_supported(adev))
		pdevinfo.dma_mask = DMA_BIT_MASK(32);
	else
		pdevinfo.dma_mask = 0;

	pdev = platform_device_register_full(&pdevinfo); // =================
	if (IS_ERR(pdev))
		dev_err(&adev->dev, "platform device creation failed: %ld\n",
			PTR_ERR(pdev));
	else {
		set_dev_node(&pdev->dev, acpi_get_node(adev->handle));
		dev_dbg(&adev->dev, "created platform device %s\n",
			dev_name(&pdev->dev));
	}

	kfree(resources);

	return pdev;
}
EXPORT_SYMBOL_GPL(acpi_create_platform_device);

```
`platform_device_register_full`函数完成platfrom device设备的创建