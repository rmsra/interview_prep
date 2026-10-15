# Debugging & Profiling Tools

**Courses:** [DSA](../dsa-course/dsa_00_README_start_here.md) · [Networking](../networking-course/networking_00_README_start_here.md) · **Operating Systems** · [C++](../cpp-course/cpp_00_README_start_here.md) · [Python](../python-course/python_00_README_start_here.md)
**Operating Systems modules:** [Index](os_00_README_start_here.md) · [Processes/Threads/Scheduling](os_01_processes_threads_and_scheduling.md) · [Synchronization/Concurrency](os_02_synchronization_and_concurrency.md) · [Deadlocks](os_03_deadlocks.md) · [Memory Management](os_04_memory_management.md) · [File Systems/IO](os_05_file_systems_and_io.md) · [IPC/Virtualization](os_06_ipc_and_virtualization.md) · [Linux Kernel/Real-Time](os_07_linux_kernel_and_real_time_systems.md) · [OS Practice Problems](os_08_os_interview_practice_problems.md) · **Debugging & Profiling Tools**

---

Added after reviewing the SAI job posting: "analytical reasoning and debugging, with practical experience in using tools such as GDB, PDB, etc." is a named preferred qualification, and your resume already lists most of the tools below — this file is about having them at your fingertips for a live conversation, not learning them fresh. This is also squarely "Debugging Focus" interviewer territory per Tanique's breakdown.

## GDB — the interactive C/C++ debugger

**Core commands, the ones to have cold:**

| Command | What it does |
|---|---|
| `break file:line` / `break func` | set a breakpoint |
| `break file:line if cond` | conditional breakpoint — critical for bugs that only manifest on iteration 40,000 |
| `run` / `continue` | start / resume execution |
| `next` / `step` | step over / step into |
| `print expr` | evaluate and print — works on variables, expressions, even function calls |
| `backtrace` (`bt`) | call stack of the current thread |
| `frame N` / `up` / `down` | navigate the call stack |
| `watch expr` | break when a variable/expression's value changes — invaluable for "something is corrupting this memory and I don't know where" |
| `info threads` | list all threads and their state |
| `thread apply all bt` | backtrace of **every thread at once** — the single most useful command for debugging a hung or deadlocked multi-threaded program, since you can see exactly which locks each thread is blocked on |
| `x/Nfu addr` | examine raw memory (N units, format f, unit size u) — e.g. `x/8xw ptr` for 8 hex words |
| `catch signal SIGSEGV` | break automatically when a signal is raised, before the crash handler/default action runs |

**Core dump / post-mortem debugging** (the production-realistic workflow, worth naming explicitly):
```
ulimit -c unlimited          # enable core dumps for the shell session
./my_program                 # crashes, writes core file
gdb ./my_program core        # load the binary + the dump — full state at time of crash, no live reproduction needed
```
- **Attaching to a running/hung process:** `gdb -p <pid>` — for a production service that's stuck rather than crashed, this plus `thread apply all bt` is usually the fastest way to see exactly where every thread is stuck.
- **The optimized-build gotcha:** builds with `-O2`/`-O3` can show `<optimized out>` for variables the compiler eliminated or kept only in registers, and stepping can jump around unintuitively because instructions were reordered. `-Og` (optimize for debugging) or `-g -O0` gives a much cleaner debugging session — but real production bugs sometimes only reproduce under full optimization, so knowing to expect and work around this (rather than being confused by it live) is the actual skill.

## Valgrind — heavyweight, exhaustive instrumentation

- **Memcheck** (the default tool): catches memory leaks, invalid reads/writes (buffer overruns), use-after-free, and reads of uninitialized memory. Runs your program on a synthetic CPU it fully controls, which is precisely why it catches so much — and precisely why it's **10-50x slower**, making it a dev/test-time tool, not a production one.
- **Helgrind / DRD:** race-condition detectors — flag memory accessed by multiple threads without proper synchronization, even if the race never actually caused visible corruption in that particular run. Pairs directly with the OS course's synchronization file.
- **Callgrind:** call-graph based profiling (cache simulation, instruction counts) — useful, but `perf` (below) is the lower-overhead, more commonly reached-for choice for pure performance profiling today.
- **Gotcha relevant to your domain:** Valgrind's memory model doesn't always play well with custom allocators/memory pools (like DPDK's `mempool`, or huge-page-backed regions) — expect noisy or reduced-value output on pool-allocated memory, and know to say so if asked "would Valgrind catch a bug in your DPDK mempool" rather than overclaiming it.

## AddressSanitizer / ThreadSanitizer — the modern, CI-friendly alternative

- **ASan** (`-fsanitize=address`) and **TSan** (`-fsanitize=thread`), built into GCC/Clang: compiler-instrumented rather than full emulation, so overhead is much lower (~2x for ASan vs. Valgrind's 10-50x) — practical to run in CI on every build, not just an occasional manual pass.
- **The precise trade-off vs. Valgrind:** sanitizers require recompiling with the flag (Valgrind runs an unmodified binary), but are fast enough to be part of routine automated testing rather than a special debugging session — directly relevant to your own CI/regression-automation background, and a strong answer if asked how you'd prevent a memory bug from reaching production rather than just how you'd find one after the fact.
- **A real, honest anecdote if asked "have you used ASan":** you tried building DPDK 21.11 with `-Db_sanitize=address` to chase down a suspected memory corruption, and hit an unresolved linker error (`asan` library not found) rather than a clean success — worth knowing this is a fine answer to give as-is. "I tried it, hit a build/environment snag, and here's what I'd check next (confirm `libasan` matches the compiler version, or try GCC's built-in sanitizer support directly)" is a more credible debugging answer than a story where every tool works on the first try.

## `perf` — sampling profiler and hardware counters

- **`perf stat`**: high-level counters for a run — instructions, cycles, cache misses, branch mispredictions — the first thing to check when something's "just slow" and you don't know why yet.
- **`perf record` / `perf report`**: sampling profiler — periodically samples the call stack, builds a picture of where time is actually going without the heavy instrumentation overhead of something like Callgrind.
- **`perf c2c`**: cache-to-cache sharing analysis — the specific tool for diagnosing false sharing (see the OS synchronization file's false-sharing section, and directly the tool you'd name for your own false-sharing story if asked "how did you find it").
- **`perf top`**: live, continuously-updating view — good for "something's burning CPU right now, what."

## Flame Graphs

- Built from `perf record` output (or eBPF-collected stack samples) via Brendan Gregg's FlameGraph scripts — visualizes an entire profile as nested boxes, one per stack frame.
- **How to read one, precisely** (a fair thing to be asked directly): **width represents time spent** (wider = more samples landed in that function or its descendants), **not** call order — the x-axis is alphabetical/arbitrary, not chronological. Height is stack depth. A wide, flat box near the top means one function is genuinely hot; a wide *base* with the width persisting up through many child frames means the time is spread through a call chain rather than concentrated in one leaf function.

## eBPF (you've noted this as "learning" — here's the honest, conceptual-depth version)

- In-kernel, **sandboxed, verifier-checked bytecode** that can attach to kernel events (tracepoints, kprobes/uprobes, network hooks) without writing a custom kernel module — safety (the verifier rejects programs that could crash or hang the kernel) and low overhead are the two properties that made it the modern default over older tracing mechanisms (ftrace, SystemTap).
- **`bpftrace`** is the practical entry point — a high-level, `awk`-like scripting frontend over eBPF, good for one-off "trace this syscall/function across the whole system live" investigations without writing raw eBPF.
- **XDP (eXpress Data Path)** — worth knowing specifically given your DPDK background: XDP runs eBPF programs at the earliest possible point in the NIC driver, still **inside the kernel**, for very fast packet filtering/redirection decisions — a different point on the same spectrum as DPDK's full kernel bypass. The honest comparison, if asked: DPDK trades kernel integration entirely for maximum throughput/control; XDP keeps you in-kernel (so existing kernel networking features, tooling, and security boundaries still apply) while still getting a large speedup over the full traditional network stack — a "good default" answer is that XDP is often the right choice when you need a fast filtering/redirect decision but still want to interoperate with the normal kernel network stack for everything else, while DPDK is the right choice when you're building a dedicated data-plane application and want to own the entire packet path yourself, as you did.

## Wireshark / `tcpdump`

- **Capture filters vs. display filters — the precise distinction, a common gotcha:** capture filters (BPF syntax — the same underlying mechanism eBPF's name references) are applied *while capturing*, reducing what's written at all; display filters (Wireshark's own richer syntax) are applied *after* capture, for analysis, and don't reduce what was captured — you can change a display filter freely and still see everything you captured, but a capture filter's exclusions are gone for good.
- **Follow TCP Stream:** reconstructs one side or both sides of a full TCP session into readable order — the fastest way to answer "what did the application actually send/receive" without manually correlating individual packets.
- **Your own project is a good callback here:** you've already built programmatic pcap analysis tooling (NumPy/Pandas over captured traffic) — worth mentioning directly if asked about scaling packet analysis beyond manual Wireshark inspection; `scapy` or `pyshark` are the standard Python libraries for that same task if the topic comes up as "how would you automate this."

## PDB — the Python debugger (Python is an explicit minimum qualification for this role, not just C++)

| Command | What it does |
|---|---|
| `b`(`reak`) `file:line` | set a breakpoint |
| `n`(`ext`) / `s`(`tep`) | step over / into |
| `c`(`ontinue`) | resume |
| `p`(`rint`) `expr` / `pp expr` | print / pretty-print |
| `l`(`ist`) | show source around current line |
| `bt` | backtrace |

- **`breakpoint()`** (Python 3.7+): the modern way to drop into the debugger from inside code — calls `pdb.set_trace()` by default (respecting `$PYTHONBREAKPOINT` if you want a different debugger, e.g. `ipdb`), without needing an explicit `import pdb` at the top of the file.
- **Post-mortem debugging:** `python -m pdb -c continue myscript.py` runs the script and drops into `pdb` automatically at an unhandled exception, landing you at the exact frame that failed — the Python equivalent of GDB's core-dump workflow above.

## Practice signal

The strongest answer to "how would you debug X" is usually **naming the right tool for the specific failure class**, not deep syntax recall of any single tool:

| Symptom | Reach for |
|---|---|
| Memory corruption / leak | ASan (fast, CI-friendly) or Valgrind Memcheck (exhaustive, slow) |
| Data race | TSan or Helgrind |
| Deadlock / hung process | GDB, `thread apply all bt` |
| "It's just slow," unknown cause | `perf stat` → `perf record`/`report` → flame graph |
| Scaling doesn't improve with more cores | `perf c2c` (false sharing) |
| Production crash, can't reproduce live | core dump + GDB |
| Live production issue, can't stop the service | eBPF/`bpftrace` (near-zero overhead, no restart needed) |
| Packet-level network behavior | Wireshark/`tcpdump`, capture vs. display filter |
| Python-side logic bug | `pdb` / `breakpoint()` |

Matching symptom to tool, out loud, before diving into any one of them, is exactly what a "Debugging Focus" interviewer is listening for.

---

## Related Modules

- [OS 02 — Synchronization (races, false sharing these tools catch)](os_02_synchronization_and_concurrency.md)
- [OS 03 — Deadlocks (GDB's thread apply all bt)](os_03_deadlocks.md)
- [Networking 07 — DPDK (eBPF/XDP vs. kernel bypass compared)](../networking-course/networking_07_dpdk_kernel_bypass_and_your_story.md)
- [Networking 08 — Troubleshooting (tying tools to symptoms)](../networking-course/networking_08_troubleshooting_and_design_questions.md)
