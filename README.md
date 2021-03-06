# Map Guard

Map Guard is a proof of concept memory safety exploit mitigation that protects mmap based page allocations. It enforces a simple set of allocation security policies configurable via environment variables:

```
MG_DISALLOW_RWX - Disallows PROT_READ, PROT_WRITE, PROT_EXEC mappings
MG_DISALLOW_X_TRANSITION - Disallows RW allocations to ever transition to PROT_EXEC
MG_DISALLOW_TRANSITION_FROM_X - Disallows R-X allocations to ever transition to PROT_WRITE
MG_DISALLOW_STATIC_ADDRESS - Disallows page allocations at a set address (enforces ASLR)
MG_ENABLE_GUARD_PAGES - Force top and bottom guard page allocations
MG_PANIC_ON_VIOLATION - Instructs Map Guard to abort the process when these policies are violated
MG_POISON_ON_ALLOCATION - Fill all allocated pages with a byte pattern 0xde
MG_USE_MAPPING_CACHE - Enable the mapping cache, required for guard page allocation
```

This library requires hooking `mmap`, `munmap`, `mprotect`, and `mremap`. There are still corner cases that need support. This library introduces some performance overhead, especially if guard pages are enabled. Map Guard has only been tested on 64 bit Linux but should work on 32 bit programs and Mac OS with minor modifications. 

## Testing

You can test Map Guard by running `./run_tests.sh`:

```
# ./run_tests.sh 
rm -rf ../build/
mkdir -p ../build/
clang -Wall -fPIC -shared -ldl -DDEBUG -ggdb mapguard.c vector.c -o ../build/mapguard.so
mkdir -p ../build/
clang -Wall -fPIE -fPIC  -DDEBUG -ggdb mapguard_test.c vector.c -o ../build/mapguard_test
[LOG][67059](map_guard_pages) Successfully allocated bottom guard page: [(guard) 0x7fb9e2516000 | 0x7fb9e2517000]
[LOG][67059](map_guard_pages) Failed to map top guard page @ 0x7fb9e2518000. Mapped @ 0x7fb9e2515000
[LOG][67059](map_memory) Successfully mmapped RW memory @ 0x7fb9e2517000
[LOG][67059](map_rw_memory) Test passed
[LOG][67059](munmap) Found mapguard cache entry for mapping 0x7fb9e2517000
[LOG][67059](munmap) Unmapped guard pages 0x7fb9e2516000 and 0x7fb9e2515000
[LOG][67059](munmap) Deleting cache entry for 0x7fb9e2517000
[LOG][67059](unmap_memory) Successfully munmapped memory @ 0x7fb9e2517000
[LOG][67059](mmap) Disallowing RWX memory allocation
[LOG][67059](map_memory) Failed to map RWX memory
[LOG][67059](map_rwx_memory) Test passed
[LOG][67059](map_guard_pages) Successfully allocated bottom guard page: [(guard) 0x7fb9e2516000 | 0x7fb9e2517000]
[LOG][67059](map_guard_pages) Failed to map top guard page @ 0x7fb9e2518000. Mapped @ 0x7fb9e2515000
[LOG][67059](map_memory) Successfully mmapped RW memory @ 0x7fb9e2517000
[LOG][67059](mprotect) Cannot allow mapping 0x7fb9e2517000 to be set PROT_EXEC
[LOG][67059](map_rw_then_x_memory) Test passed
[LOG][67059](munmap) Found mapguard cache entry for mapping 0x7fb9e2517000
[LOG][67059](munmap) Unmapped guard pages 0x7fb9e2516000 and 0x7fb9e2515000
[LOG][67059](munmap) Deleting cache entry for 0x7fb9e2517000
[LOG][67059](unmap_memory) Successfully munmapped memory @ 0x7fb9e2517000
[LOG][67059](map_guard_pages) Successfully allocated bottom guard page: [(guard) 0x7fb9e2516000 | 0x7fb9e2517000]
[LOG][67059](map_guard_pages) Failed to map top guard page @ 0x7fb9e2518000. Mapped @ 0x7fb9e2515000
[LOG][67059](map_memory) Successfully mmapped RW memory @ 0x7fb9e2517000
[LOG][67059](map_guard_pages) Successfully allocated bottom guard page: [(guard) 0x7fb9e2512000 | 0x7fb9e2513000]
[LOG][67059](map_guard_pages) Failed to map top guard page @ 0x7fb9e2515000. Mapped @ 0x7fb9e2515000
[LOG][67059](remap_memory) Successfully remapped Remap memory 0x7fb9e2517000 @ 0x7fb9e2513000
[LOG][67059](map_then_mremap) Test passed
[LOG][67059](munmap) Found mapguard cache entry for mapping 0x7fb9e2513000
[LOG][67059](munmap) Unmapped guard pages 0x7fb9e2512000 and 0x7fb9e2515000
[LOG][67059](munmap) Deleting cache entry for 0x7fb9e2513000
[LOG][67059](unmap_remapped_memory) Successfully munmapped remapped memory @ 0x7fb9e2513000
[LOG][67059](mmap) Disallowing memory allocation at static address 0x7f3bffaaa000
[LOG][67059](map_static_address) Test passed
[LOG][67059](map_guard_pages) Successfully allocated bottom guard page: [(guard) 0x7fb9e2516000 | 0x7fb9e2517000]
[LOG][67059](map_guard_pages) Failed to map top guard page @ 0x7fb9e2518000. Mapped @ 0x7fb9e2515000
[LOG][67059](map_memory) Successfully mmapped Poison Bytes memory @ 0x7fb9e2517000
[LOG][67059](check_poison_bytes) Test passed
[LOG][67059](munmap) Found mapguard cache entry for mapping 0x7fb9e2517000
[LOG][67059](munmap) Unmapped guard pages 0x7fb9e2516000 and 0x7fb9e2515000
[LOG][67059](munmap) Deleting cache entry for 0x7fb9e2517000
[LOG][67059](unmap_memory) Successfully munmapped memory @ 0x7fb9e2517000
[LOG][67059](map_guard_pages) Successfully allocated bottom guard page: [(guard) 0x7fb9e2516000 | 0x7fb9e2517000]
[LOG][67059](map_guard_pages) Failed to map top guard page @ 0x7fb9e2518000. Mapped @ 0x7fb9e2515000
[LOG][67059](map_memory) Successfully mmapped R-X memory @ 0x7fb9e2517000
[LOG][67059](mprotect) Cannot allow mapping 0x7fb9e2517000 to transition from PROT_EXEC to PROT_WRITE
[LOG][67059](check_x_to_w) Test passed
[LOG][67059](munmap) Found mapguard cache entry for mapping 0x7fb9e2517000
[LOG][67059](munmap) Unmapped guard pages 0x7fb9e2516000 and 0x7fb9e2515000
[LOG][67059](munmap) Deleting cache entry for 0x7fb9e2517000
[LOG][67059](unmap_memory) Successfully munmapped memory @ 0x7fb9e2517000
```

Or run your own program with the library:

```
MG_PANIC_ON_VIOLATION=0         \
MG_USE_MAPPING_CACHE=1          \
MG_DISALLOW_RWX=1               \
MG_DISALLOW_STATIC_ADDRESS=1    \
MG_ENABLE_GUARD_PAGES=1         \
MG_DISALLOW_X_TRANSITION=1      \
MG_POISON_ON_ALLOCATION=1       \
LD_PRELOAD=build/mapguard.so ./your_program
```

## Who

Copyright Chris Rohlf - 2019

chris.rohlf@gmail.com
