## Memory management

### Make() vs new()

`new()` Allocates but does not initialize memory. Returns a memory address but zeroed out storage. 

`make()` Allocate and initialized memory. Allocates non-zeroed storage and returns a memory address.

`*` means its a pointer `&` dereferences the pointer.

## Complex types

To print a struct for debbugging purposes use `%+v` for example `fmt.Printf("%+v\n", poodle)` where `poodle` is an instance of a struct.
