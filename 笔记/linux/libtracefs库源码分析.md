tracefs_instance_create函数
```c
struct tracefs_instance *tracefs_instance_create(const char *name) 
{
        struct tracefs_instance *inst = NULL;
        char *path = NULL;
        const char *tdir;
        struct stat st;
        mode_t mode;
        int ret;

        tdir = tracefs_tracing_dir();
        if (!tdir)
                return NULL;
        inst = instance_alloc(tdir, name);
        if (!inst)
                return NULL;

        path = tracefs_instance_get_dir(inst);
        ret = stat(path, &st);
        if (ret < 0) {
                /* Cannot create the top instance, if it does not exist! */
                if (!name)
                        goto error;
                mode = get_trace_file_permissions("instances");
                if (mkdir(path, mode))
                        goto error;
                inst->flags |= FLAG_INSTANCE_NEWLY_CREATED;
        }
        tracefs_put_tracing_file(path);
        return inst;

error:
        tracefs_instance_free(inst);
        return NULL;
}
```
1.  这个函数tdir是`tracefs`的root path: `/sys/kernel/tracing`.
2.  `instance_alloc`函数为`inst`分配空间。
3. `tracefs_instance_get_dir`函数获取到指定`name`的instance路径为: `/sys/kernel/tracing/instances/name`, name为参数设置的name， 如果文件不存在将会创建
以cxl为例：
![[cxl-monitor.png]]

