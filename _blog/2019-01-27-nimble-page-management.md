---
title: "Tutorial: Nimble Page Management for Tiered Memory Systems"
collection: blog
date: 2019-04-20

---

- The kernel of "Nimble Page Management for Tiered Memory Systems" is [here](https://github.com/ysarch-lab/nimble_page_management_asplos_2019).
- Its companion userspace applications and microbenchmarks can be find [here](https://github.com/ysarch-lab/nimble_page_management_userspace).

## Kernel compilation

Use `make menuconfig` and select `NIMBLE_PAGE_MANAGEMENT` to make sure the
kernel can be compiled correctly. (Use `/` to search for that option.)

Make sure you have `CONFIG_PAGE_MIGRATION_PROFILE=y` in your .config if you want
to run microbenchmarks. (Use `make menuconfig` to search and enable this option.)

## Microbenchmarks for page migration mechanisms

You need a two socket NUMA machine to run microbenchmarks properly. *numactl* and *libnuma-dev* packages are required. `numactl -H` will tell your machine's NUMA topology:

```
available: 2 nodes (0-1)
node 0 cpus: 0 2 4 6 8 10 12 14 16 18 20 22 24 26 28 30
node 0 size: 31374 MB
node 0 free: 203 MB
node 1 cpus: 1 3 5 7 9 11 13 15 17 19 21 23 25 27 29 31
node 1 size: 32231 MB
node 1 free: 5535 MB
node distances:
node   0   1
  0:  10  20
  1:  20  10
```

This machine has two NUMA nodes (0-1), each has 32GB memory (**Note**: it is not necessarily the same machine as the one described in the paper).

1. Clone microbenchmarks from GitHub repository [here](https://github.com/ysarch-lab/nimble_page_management_userspace).

2. In **microbenchmarks** folder, you can find microbenchmarks for all four page migration optimizations in different subfolders.

3. In each subfolder, `make thp_move_pages` generates page migrations using THPs, while `make non_thp_move_pages` generates page migration using non-THPs, namely 4KB base pages.

4. In **thp_page_migration_and_parallel** subfolder, there are four shell scripts migrating pages between NUMA node 0 and NUMA node 1.

   1. run_non_thp_test.sh: it migrates 2<sup>0</sup> to 2<sup>9</sup> 4KB pages between node 0 and node 1.
   2. run_non_thp_2mb_page_test.sh: it migrates 2<sup>0</sup> to 2<sup>9</sup> 512 4KB pages (each set of 512 4KB pages are equivalent to 1 2MB page size) between node 0 and node 1.
   3. run_split_thp_test.sh: it migrates 2<sup>0</sup> to 2<sup>9</sup> 2MB THPs between node 0 and node 1 but each THP will be split before migration.
   4. run_thp_test.sh: it migrates 2<sup>0</sup> to 2<sup>9</sup> 2MB THPs natively between node 0 and node 1.

   In each of these scripts: `MULTI="1 2 4 8 16"` means 1, 2, 4, 8, 16 threads will be used to migrate pages.

5. In **concurrent_page_migration** and **exchange_page_migration** subfolders, you will see similar scripts: run_non_thp_test.sh for 4KB page migration and run_thp_test.sh for 2MB THP migration.

After running the scripts, a `stats_*` folder will be created, stats files have the names like `[mt|seq]_[thp|non_thp]_[2mb|4kb]_page_order_[0-9]`, meaning sequential or multithreaded migration method for THP or non-THP (namely 4KB base pages) with page sizes 2MB or 4KB and number of 2<sup>0</sup> to 2<sup>9</sup> pages. In each file, you will see results like:


```
Total_cycles    Begin_timestamp End_timestamp
6005843 24045933446997535       24045933453003378
syscall_timestamp       check_rights_cycles     migrate_prep_cycles     form_page_node_info_cycles    form_physical_page_list_cycles   enter_unmap_and_move_cycles     split_thp_page_cycles   get_new_page_cycles    lock_page_cycles        unmap_page_cycles       change_page_mapping_cycles      copy_page_cycles       remove_migration_ptes_cycles    putback_old_page_cycles putback_new_page_cycles migrate_pages_cleanup_cycles   store_page_status_cycles        return_to_syscall_cycles        last_timestamp
24045933447007103       1716    199242  66342   461837  33925   0       433761  55648   1315997 59190 2274685  391541  679706  0       299     20532   506     24045933453002030
--
Test successful.
```

The numbers are all in CPU cycles, read from x86_64 `rtdsc` instructions. Here are the meaning of these items, where all numbers from Row 3 and 4 are recorded inside the kernel and read via `/proc/<pid>/move_pages_breakdown`. The listed items below from 4 onwards are all kernel activities.

1. Total_cycles: total CPU cycles spent on this migration.
2. Begin_timestamp: a number returned by `rtdsc` instruction at the beginning of the migration. `rtdsc` always gives an increasing number in your machine, so it can be used as a unique timestamp.
3. End_timestamp: a number returned by `rtdsc` at the end of the migration.
4. syscall_timestamp: `rtdsc` number recorded inside the kernel at the beginning of `migrate_pages()` system call.
5. check_rights_cycles: cycles spent on checking process permissions in the kernel.
6. migrate_prep_cycles: cycles spent on preparing page migrations.
7. form_page_node_info_cycles: cycles spent on reading page node information.
8. form_physical_page_list_cycles: cycles spent on making a page list for migration.
9. enter_unmap_and_move_cycles: cycles spent before entering `unmap_and_move` kernel function.
10. split_thp_page_cycles: cycles spent on splitting a THP (will be 0 if migration 4KB pages or 2MB THPs without splitting).
11. get_new_page_cycles: cycles spent on allocating new pages on remote node.
12. lock_page_cycles: cycles spent on locking pages before migration.
13. unmap_page_cycles: cycles spent on unmapping to-be-migrated pages.
14. change_page_mapping_cycles: cycles spent on copying page mapping data structure from old pages to new pages.
15. copy_page_cycles: cycles spent on copying page data from old pages to new pages.
16. remove_migration_ptes_cycles: cycles spent on removing migration page table entries and mapping new pages.
17. putback_old_page_cycles: cycles spent on putting back old pages.
18. putback_new_page_cycles: cycles spent on putting back new pages on LRU page lists.
19. migrate_pages_cleanup_cycles: cycles spent on cleaning up page migration process.
20. store_page_status_cycles: cycles spent on storing page migration results to the status array passed via `migrate_pages()` system call.
21. return_to_syscall_cycles: cycles spent before exiting the system call.
22. last_timestamp: `rtdsc` number recorded inside the kernel at the end of `migrate_pages()` system call.

With these numbers, you should be able to get detail breakdown of Linux page migrations.

## User application launcher

To launch benchmarks for end-to-end results, you will need the scripts and the launcher from **end_to_end_launcher** folder. Also you need to enable cgroup v2 support by adding `systemd.unified_cgroup_hierarchy=1` to your kernel boot parameters. What I did is adding `systemd.unified_cgroup_hierarchy=1` to `GRUB_CMDLINE_LINUX` in `/etc/default/grub` file, like `GRUB_CMDLINE_LINUX="systemd.unified_cgroup_hierarchy=1"` and run `sudo update_grub2`.

In **end_to_end_launcher** folder, run `make` to create launcher binary from *launch.c* file.

1. create_die_stack_mem.sh can help you create a cgroup limiting your fast memory size.
2. run_bench.sh uses launcher to run your applications in cgroup created by create_die_stack_mem.sh. You should change `PROJECT_LOC` to point to the folder of launcher binary.
3. Each of your applications should be in a separate folder like graph500-omp from the GitHub repository. You also need to create a `bench_run.sh` file to tell `run_bench.sh` the name of your application (`BENCH_NAME`) and how to run it (`BENCH_RUN`). **Note**: you should adjust your benchmark size, so that it can fit in your slow memory. For example, in this tutorial, it is better to use 30GB as your benchmark footprint, considering kernel might use some memory in each 32GB memory node.
4. run_all.sh will loop through `BENCHMARK_LIST` and run each of them for end-to-end performance evaluation.
   1. `SHRINK_PAGE_LISTS` means page migration policy will be used.
   2. `NR_RUNS` means how many times you want repeat the experiments.
   3. `MIGRATION_BATCH_SIZE` means how many pages you want to migrate at the same time when concurrent page migration is used.
   4. `MIGRATION_MT` means how many CPU threads you want to use for page migration.
   5.  `MEM_SIZE_LIST` allows you to specify a list of fast memory sizes.
   6. `PAGE_REPLACEMENT_SCHEMES`: you can provide different migration options showing in my paper: all-remote-access and all-local-access are All Slow and All Fast policies. non-thp-migration, thp-migration, exchange-pages are using different ways of migrating pages in end-to-end evaluation with the same Linux page replacement policy.
   7. `THREAD_LIST` allows you to specify how many hardware threads your applications want to use.

For slow memory, you can use a real NVM, slowing down one of your NUMA nodes with memhog, or use HP's [quartz](https://github.com/HewlettPackard/quartz) tool to slow down one of your NUMA nodes.