# Examining DAWG Cache Partitioning Performance through CPU simulation

This proof-of-concept implements and evaluates the proposed dynamically allocated way guard (DAWG) strategy for mitigating cache-based side-channel attacks using gem5's syscall emulation (SE) mode.


## Implementation Flow
1. CPU assigns a domain id to each memory access depending on access type (ifetch, load, store) and writes it into the request.
2. The request is wrapped into a packet that carries the domain id to the cache.  
4. On a tag hit, the cache queries the WayGuardTable for the allowed-way bitmask for that set and the packet’s domain.  
5. If the hit way is allowed by the mask, the access completes as a normal hit.
6. If the hit way is disallowed, the cache treats the hit as a miss (a masked-hit), records a hit-masked event, and proceeds to miss handling.  
7. During miss handling, the replacement logic fetches the allowed-way mask and filters replacement candidates by intersecting with allowed ways.  
8. If the filtered candidate list is non-empty, the replacement policy selects a victim from those allowed ways, records a filter event, and the eviction/installation proceeds
9. Currently, if the filtered candidate list is empty, the system falls back to the original victim selection
10. The chosen way is filled and the installed block’s metadata is set to the originating domain so future accesses are enforced consistently.  
11. MSHR entries and forwarded requests preserve and copy the domain id so downstream fills and installs use the same domain for enforcement.  

### Build:
`scons build/X86/gem5.opt -j$(nproc)`

### Run & Benchmark:
 both baseline & dawg:<br><br> 
`python3 util/run_dawg_experiment.py` <br><br> 
 run only dawg:<br><br> 
`python3 util/run_dawg_experiment.py --no-baseline` <br><br> 
 run only baseline:<br><br> 
`python3 util/run_dawg_experiment.py --no-dawg`


### References:
[DAWG: A Defense Against Cache Timing Attacks in Speculative Execution Processors*](https://eprint.iacr.org/2018/418.pdf)

[Microbenchmark](https://github.com/tgrogers/gem5)
