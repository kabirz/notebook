入口函数:
`x86_64_start_kernel` (*arch/x86/kernel/head64.c)

```c
asmlinkage __visible void __init x86_64_start_kernel(char * real_mode_data)
{
        cr4_init_shadow();
        reset_early_page_tables();
        clear_bss(); 
        clear_page(init_top_pgt);
        /*
         * SME support may update early_pmd_flags to include the memory
         * encryption mask, so it needs to be called before anything
         * that may generate a page fault.
         */
        sme_early_init();
        kasan_early_init();
        idt_setup_early_handler();
        copy_bootdata(__va(real_mode_data));
        /*
         * Load microcode early on BSP.
         */
        load_ucode_bsp();
        /* set init_top_pgt kernel high mapping*/
        init_top_pgt[511] = early_top_pgt[511];
        x86_64_start_reservations(real_mode_data);
}
```

