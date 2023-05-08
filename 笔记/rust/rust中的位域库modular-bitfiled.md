[Robbepop/modular-bitfield](https://github.com/Robbepop/modular-bitfield)
[zhp-rs/bitfield (github.com)](https://github.com/zhp-rs/bitfield)
在C中是这样使用的
```c
#include <stdio.h>

struct Pa {
    unsigned char a1: 1;
    unsigned char a2: 2;
    unsigned char a5: 5;
    unsigned char b1: 1;
    unsigned char b2: 1;
    unsigned char b6: 6;
    short s;
};

int main(int argc, char const *argv[])
{
	struct Pa da = {1, 3, 9, 1, 0, 0x2d, 0x1122};
	unsigned int *pint = (unsigned int *)&da;

    printf("{ a1: %u, a2: %u, a5: %u, b1: %u, b2: %u, b6: %u, s: %d }\n", 
    da.a1, da.a2, da.a5, da.b1, da.b2, da.b6, da.s
    );
	printf("int: %#x\n", *pint);
	return 0;
}
```
结果：
```log
{ a1: 1, a2: 3, a5: 9, b1: 1, b2: 0, b6: 45, s: 4386 }
int: 0x1122b54f
```

rust中的实现
```rust
use modular_bitfield::prelude::*;

#[bitfield]
#[repr(u32)]
#[derive(Debug)]
pub struct Pa {
    pub a1: B1,
    pub a2: B2,
    pub a5: B5,
    pub b1: bool,
    pub b2: bool,
    pub b6: B6,
    pub s: u16
}

fn main() {
    let da = Pa::new()
    .with_a1(0b1)
    .with_a2(0b11)
    .with_a5(0b01001)
    .with_b1(true)
    .with_b2(false)
    .with_b6(0b101101)
    .with_s(0x1122);
    println!("{:?}", da);
    let bs = da.into_bytes();
    let d_u32 = u32::from_ne_bytes(bs);
    println!("{:#b}, {:#x}", d_u32, d_u32);

    let d_u16 = u16::from_ne_bytes(bs[..2].try_into().unwrap());
    println!("{:#b}, {:#x}", d_u16, d_u16);

    let dd = Pa::from(0x12345678);
    let di = u32::from(dd);
    println!("{:?}", di);
}
```
结果：
```log
Pa { a1: 1, a2: 3, a5: 9, b1: true, b2: false, b6: 45, s: 4386 }
0b10001001000101011010101001111, 0x1122b54f
0b1011010101001111, 0xb54f
305419896
```
