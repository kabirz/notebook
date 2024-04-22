## 设备插件有三种方法
### 1. 在core实例化时不指定xml参数则加载编译时生成的插件库

```cpp
inline const std::map<Key, Value> get_compiled_plugins_registry() {
    static const std::map<Key, Value> plugins_hpp = {
        { "CPU", Value { "libopenvino_riscv_cpu_plugin.so", {} } },
    };

    return plugins_hpp;
}
```
这段code会在编译的时候由cmake自动生成
调用流程[code](https://github.com/openvinotoolkit/openvino/blob/master/src/inference/src/cpp/core.cpp#L68)
```CPP
Core::Core(const std::string& xml_config_file) {
    _impl = std::make_shared<Impl>();

    std::string xmlConfigFile = find_plugins_xml(xml_config_file);
    if (!xmlConfigFile.empty())
        OV_CORE_CALL_STATEMENT(
            // If XML is default, load default plugins by absolute paths
            _impl->register_plugins_in_registry(xmlConfigFile, xml_config_file.empty());)
    // Load plugins from the pre-compiled list
    OV_CORE_CALL_STATEMENT(_impl->register_compile_time_plugins();) // <<<<<<<<<
}
```
`register_compile_time_plugins`   -> `get_compiled_plugins_registry`[code](https://github.com/openvinotoolkit/openvino/blob/master/src/inference/src/dev/core_impl.cpp#L488)
                             -> `register_plugin_in_registry_unsafe`
### 2. 在core实例化时指定xml参数则加载xml文件
xml示例如下
```xml
<ie>
    <plugins>
        <plugin name="GNA" location="libGNAPlugin.so">
        </plugin>
        <plugin name="HETERO" location="libHeteroPlugin.so">
        </plugin>
        <plugin name="CPU" location="libMKLDNNPlugin.so">
        </plugin>
        <plugin name="MULTI" location="libMultiDevicePlugin.so">
        </plugin>
        <plugin name="GPU" location="libclDNNPlugin.so">
        </plugin>
        <plugin name="MYRIAD" location="libmyriadPlugin.so">
        </plugin>
        <plugin name="HDDL" location="libHDDLPlugin.so">
        </plugin>
        <plugin name="FPGA" location="libdliaPlugin.so">
        </plugin>
    </plugins>
</ie>
```
`register_plugins_in_registry`     -> `pugixml::parse_xml`
                             -> `register_plugin_in_registry_unsafe`
### 3. 调用 core的register_plugin方法
```cpp
void Core::register_plugin(const std::string& plugin, const std::string& device_name, const ov::AnyMap& properties) {
	OV_CORE_CALL_STATEMENT(_impl->register_plugin(plugin, device_name, properties););
}

void Core::register_plugins(const std::string& xml_config_file) {
	OV_CORE_CALL_STATEMENT(_impl->register_plugins_in_registry(xml_config_file););
}
```
### 调用过程
 `compile_model` --> `get_plugin`
 `import_model` --> `get_plugin`
 `query_model` --> `get_plugin`
 `create_context` --> `get_plugin`

 ```cpp
 get_plugin() {
 so = ov::util::load_shared_object(desc.libraryLocation.c_str());
 std::shared_ptr<ov::IPlugin> plugin_impl;
 reinterpret_cast<ov::CreatePluginFunc*>(ov::util::get_symbol(so, ov::create_plugin_function))(plugin_impl); # symbol "create_plugin_engine"
 plugin = Plugin{plugin_impl, so};
 }
```
在这里如果创建插件需要使用`OV_DEFINE_PLUGIN_CREATE_FUNCTION`宏
```cpp
#define OV_DEFINE_PLUGIN_CREATE_FUNCTION(PluginType, version, ...) \
    OPENVINO_PLUGIN_API void OV_CREATE_PLUGIN(::std::shared_ptr<::ov::IPlugin>& plugin) noexcept(false); \
    void OV_CREATE_PLUGIN(::std::shared_ptr<::ov::IPlugin>& plugin) noexcept(false) {\
        try {\
            plu gin = ::std::make_shared<PluginType>(__VA_ARGS__); \
            plugin->set_version(version); \
        } catch (const std::exception& ex) { \
            OPENVINO_THROW(ex.what()); \
        } \
    }
```
## 前端插件(pytorch/TF/TFLITE/ONNX)

前端插件在调用covert_model的时候加载插件动态库