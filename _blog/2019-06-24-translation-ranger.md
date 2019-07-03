---
title: "Tutorial: Translation Ranger: Operating System Support for
Contiguity-Aware TLBs"
collection: blog
date: 2019-06-24

---

- The kernel of "Translation Ranger" is [here](https://github.com/ysarch-lab/translation_ranger_isca_2019).
- A newer version of "Translation Ranger" rebased on Linux kenrel v5.0-rc5 is [here](https://lwn.net/Articles/779979/) and [patches](https://patchwork.kernel.org/series/81075/mbox/). You will need to apply the patch (`git am <patch file>`) on top of Linux v5.0-rc5.
- Its companion userspace applications can be find [here](https://github.com/ysarch-lab/translation_ranger_userspace).



##Kernel compilation

Use `make menuconfig` and select `TRANSLATION_RANGER` option to make sure the kernel can be compiled correctly. (Use `/` to search for the option)

To get an initial kernel configuration file, you can copy a config file like `config-4.4.0-154-generic` from your `/boot` folder to the kernel source folder and rename it to `.config`.

To reduce kernel compilation time, you could run `make localmodconfig`, which only selects the modules currently loaded in your system.

##Run your application with Translation Ranger

To launch benchmarks in Translation Ranger, you need the scripts and the launcher from the userspace GitHub repo. The application will always run on Node 1 in the system, which can be changed by assigning the NUMA node id to `FAST_NUMA_NODE` in run_bench.sh.

In the repo, run `make` to create launcher binary from _launcher.c_ file.
1. setup_thp.sh changes kernel khugepaged running configurations. The script will disable khugepaged except for Linux Default. In Enhanced khugepaged case, khugepaged is invoked via my customized system call from launcher.
2. run_bench.sh uses launcher to run your applications and performs different memory defragmentation actions, like Linux Default, Large Max Order, and Translation Ranger.
3. Each of your applications should be in a separate folder like 503.postencil in the GitHub repo. You need a `bench_run.sh` file in the folder to tell `run_bench.sh` the name of your appilcation (`BENCH_NAME`) and how to run it (`BENCH_RUN`). `BENCH_SIZE` will be exported in `run_all.sh` (see next bullet point), so you can launch your appilcations with different memory footprints by just changing `BENCH_SIZE` in `run_all.sh`.
4. run_all.sh loops through `CONFIG_LIST` for each benchmark in `BENCH_LIST` with the number of threads used in `THREAD_NUM_LIST` and memory footprint from `WORKLOAD_SIZE`(exported as `BENCH_SIZE` and used by `bench_run.sh` above). Based on the values from `DEFRAG_FRAQ_FACTOR_LIST`, like N, Translation Ranger will run every `STATS_PERIOD * N` seconds (`STATS_PERIOD` is explained below). Other MACRO explanations:
    1. `PREFER_MEM_MODE`: if an application footprint is bigger than the assigned NUMA node, the kernel will spill the extra data to other NUMA nodes or not. Yes means extra data will be allocated on other NUMA nodes; no means swapping will happen.
    2. `STATS_PERIOD`: how frequenct the launcher will scan application memory and generate contiguity stats. Every 5 seconds is the default value.
    3. `CONTIG_STATS`: dumping contiguity stats or not.
    4. `ONLINE_STATS`: debug information, do not change.
    5. `PROMOTE_1GB_MAP`: if yes, Translation Ranger will try to do in-place 1GB THP promotion.
    6. `NO_REPEAT_DEFRAG`: each VMA is augmented with timestamps telling when the VMA is changed and defragmented. If yes, Translation Ranger will compare the two timestamps and performs a memory defragmentation when the VMA is changed after last memory defragmentation. (This configuration is not presented in the paper)
    7. `CHILD_PROC_STAT`: debug information, do not change.

The scripts use different configuration names (the ones showing in `CONFIG_LIST`) than the ones (like Linux Default, Large Max Order, Translation Ranger) from the paper. Here is the one-to-one mapping:

1. Linux Default: `compact_default` and `THP_SIZE_LIST` is "2mb".
2. Large Max Order: `compact_default` and `THP_SIZE_LIST` is "1gb".
3. Enhanced khugepaged: `compact_freq` and `THP_SIZE_LIST` is "2mb".
4. Translation Ranger: `no_compact_no_daemon_defrag` and `THP_SIZE_LIST` is "2mb".

###Contiguity stats format
After running, a result folder will be created for each benchmark with a distinct configuration, like `result-<benchmark name>-<thread number>-cpu-<memory footprint size>-mem-defrag-scan-period-<the value of STATS_PERIOD>-defrag-period-<the value of DEFRAG_FRAQ_FACTOR*STATS_PERIOD>-<configuration>-<date and time of the run>`. In the folder, `<benchmark name>_mem_frag_contig_stats_0` stores the memory contiguity stats. It contains multiple memory scans, which are separated by `----`. Each scan contains contiguity stats for all VMAs in the application. Here is a sample for two VMAs:
```
00000000a8b6e356, 0xffff9b36d7c7cb0b, 0x7f1325294000, 0x7f1325295000, 1, -1
00000000a8b6e356, 0xffff9b35d7817550, 0x7ffc42439000, 0x7ffc4245a000, -29, 1, 1, 1, 1, -1
```
The fields are explained below:

1. `00000000a8b6e356`: kernel mm_struct virtual address, use it only if you have multiple application running. 
2. `0xffff9b36d7c7cb0b`: kernel vm_area virtual address, distinguish different VMAs.
3. `0x7f1325294000`: the start virutal address of the VMA.
4. `0x7f1325295000`: the end virtual address of the VMA.
5. all following fields represent all contiguous regions in the VMA, beginning from the start virtual address of the VMA. Positive number `N` means `N` 4KB contiguous physical pages are in this contiguous region and negative number `-M` means `M` 4KB non-present physical hole.

By scanning the stats, you should be able to get contiguity statistics for each scan.
