
以i2c-gpio为例
设备数是这样的
```dts
  hdmi_ddc: i2c-ddc {
    compatible = "i2c-gpio";
    sda-gpios = <&gpe4 2 (GPIO_ACTIVE_HIGH | GPIO_OPEN_DRAIN)>;
    scl-gpios = <&gpe4 3 (GPIO_ACTIVE_HIGH | GPIO_OPEN_DRAIN)>;
    i2c-gpio,delay-us = <100>;
    #address-cells = <1>;
    #size-cells = <0>;

    pinctrl-0 = <&i2c_ddc_bus>;
    pinctrl-names = "default";
    status = "okay";
  };
```
这里用gpio来模拟i2c通讯，sda和scl分别为gpio2 和gpio3
在来看源码：drivers/i2c/busses/i2c-gpio.c
```c
static const struct of_device_id i2c_gpio_dt_ids[] = {
        { .compatible = "i2c-gpio", },
        { /* sentinel */ }
};

static int i2c_gpio_probe(struct platform_device *pdev)
{
	......
	priv->sda = i2c_gpio_get_desc(dev, "sda", 0, gflags);
	priv->scl = i2c_gpio_get_desc(dev, "scl", 1, gflags);
	......
}
static struct gpio_desc *i2c_gpio_get_desc(struct device *dev,
                                           const char *con_id,
                                           unsigned int index,
                                           enum gpiod_flags gflags)
{
        struct gpio_desc *retdesc;
        int ret;

        retdesc = devm_gpiod_get(dev, con_id, gflags);
        retdesc = devm_gpiod_get_index(dev, NULL, index, gflags);

        return retdesc;
}
/* 这里拿到了gpio_desc */

```