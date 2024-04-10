- It is a means to structure the export surface of the in-kernel API.  
- It allows subsystem maintainers to partition their exported symbols into separate namespaces.   
- `EXPORT_SYMBOL` and `EXPORT_SYMBOL_GPL` allow exporting of kernel symbols to the kernel symbol table (`ksymtab`).  
- `EXPORT_SYMBOL_NS` and `EXPORT_SYMBOL_NS_GPL` take one additional argument: **the namespace**. Eg : `EXPORT_SYMBOL_NS(usb_stor_suspend, USB_STORAGE);`  
- A symbol that is exported without a namespace will refer to `NULL`. There is no default namespace if none is defined.  
- `DEFAULT_SYMBOL_NAMESPACE` if set, will become the default for all EXPORT_SYMBOL() and EXPORT_SYMBOL_GPL() macro expansions that do not specify a namespace. Eg : `ccflags-y += -DDEFAULT_SYMBOL_NAMESPACE=USB_COMMON`  
- The module code is required to use the macro `MODULE_IMPORT_NS` for the namespaces it uses symbols from. Eg : `MODULE_IMPORT_NS(USB_STORAGE);`  
- This will create a modinfo tag in the module for each imported namespace. This has the side effect, that the imported namespaces of a module can be inspected with modinfo:  
```shell  
$ modinfo drivers/usb/storage/ums-karma.ko  
[...]  
import_ns:      USB_STORAGE  
[...]  
```  
 - Fixing missing imports can be done with: `$ make nsdeps`  
 - You can also run `nsdeps` for external module builds. A typical usage is: `$ make -C <path_to_kernel_src> M=$PWD nsdeps`  
  
# Assembler Notations  
- to make assembly debugging easier, the assembler provides notations like `SYM_FUNC_START`, `SYM_FUNC_END`, `SYM_CODE_START`  
  
These macros can be divided into three main groups :  
  
1. `SYM_FUNC_*` - to annotate C-like functions. This means functions with standard C calling conventions. For example, on x86, this means that the stack contains a return address at the predefined place and a return from the function can happen in a standard way. For example, on x86, this means that the stack contains a return address at the predefined place and a return from the function can happen in a standard way.   
  
2. `SYM_CODE_*` - special functions called with special stack. Be it interrupt handlers with special stack content, trampolines, or startup functions.  
  
3. `SYM_DATA_*` - data belonging to `.data` sections and not to `.text`. Data do not contain instructions, so they have to be treated specially by the tools: they should not treat the bytes as instructions, nor assign any debug information to them.  
  
- Architecture can also override any of the macros in their own `asm/linkage.h`, including macros specifying the type of a symbol (`SYM_T_FUNC, SYM_T_OBJECT`, and `SYM_T_NONE`).   
  
For complete list of annotations, checkout : [Assembler annotations - Kernel Docs](https://docs.kernel.org/core-api/asm-annotations.html)