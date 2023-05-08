# 驱动框架的初始化
## driver_init
```c
void __init driver_init(void)
{
	/* These are the core pieces */
	bdi_init(&noop_backing_dev_info);
	devtmpfs_init();
	devices_init();
	buses_init();
	classes_init();
	firmware_init();
	hypervisor_init();

	/* These are also core pieces, but must come after the
	 * core core pieces.
	 */
	of_core_init();
	platform_bus_init();
	auxiliary_bus_init();
	cpu_dev_init();
	memory_dev_init();
	node_dev_init();
	container_dev_init();
}
```
调用流程
`start_kernel()` -> `arch_call_rest_init()` -> `rest_init()` -> `kernel_init()` -> `kernel_init_freeable()` ->`do_basic_setup()` -> `driver_init()`;
#### 1. devtmpfs_init

### 2. device_init
device init的操作是
1. 在sysfs的根目录下创建`devices_ket`，对应目录是(/sys/devices), 这是所有devices的祖宗，所有的devices创建后都会在这里
2. 在sysfs的根目录下创建`dev_kobj`，对应目录是(/sys/dev),然后在这个目录下创建字符设备(char)和块设备目录(block)
device相关详情参考[[device]]
### 3. buses_init
1. 在sysfs的根目录下创建bus目录`bus_kset`(/sys/bus), `bus_kset`会在[[bus#bus register]]的时候使用到
2. 在devices目录下创建system目录`system_kset`(/sys/devices/system), 在[[bus#subsys_system_register]]的时候使用到
### 4. classes_init
 在sysfs的根目录下创建class目录`class_kset`(/sys/class),  `class_kset`会在[[class#class_register]]的时候使用到
 更多相关请参考[[class]]

### 5. firmware_init

### 6. hypervisor_init

### 7. of_core_init
### 8. platform_bus_init

