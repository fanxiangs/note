# Finding a Map

[[null|]]The output starts with a repeated sequence of similar calls, as `bpftool` walks through all the maps looking for any with the name `config`:

bpf(BPF\_MAP\_GET\_NEXT\_ID, {start\_id=0,...}, 12) = 0             [[#list_id_4_7|]]
bpf(BPF\_MAP\_GET\_FD\_BY\_ID, {map\_id=48...}, 12) = 3              [[#list_id_4_8|]]
bpf(BPF\_OBJ\_GET\_INFO\_BY\_FD, {info={bpf\_fd=3, ...}}, 16) = 0    [[#list_id_4_9|]]
  
bpf(BPF\_MAP\_GET\_NEXT\_ID, {start\_id=48, ...}, 12) = 0           [[#list_id_4_10|]]
bpf(BPF\_MAP\_GET\_FD\_BY\_ID, {map\_id=116, ...}, 12) = 3
bpf(BPF\_OBJ\_GET\_INFO\_BY\_FD, {info={bpf\_fd=3...}}, 16) = 0

[[#code_id_4_7|]]

[[null|]]`BPF_MAP_GET_NEXT_ID` gets the ID of the next map after the value specified in `start_id`.

[[#code_id_4_8|]]

[[null|]]`BPF_MAP_GET_FD_BY_ID` returns the file descriptor for the specified map ID.

[[#code_id_4_9|]]

[[null|]]`BPF_OBJ_GET_INFO_BY_FD` retrieves information about the object (in this case, the map) referred to by the file descriptor. This information includes its name so `bpftool` can check whether this is the map it is looking for.

[[#code_id_4_10|]]

The sequence repeats, getting the ID of the next map after the one in step 1.

There’s a group of these three syscalls for each map loaded into the kernel, and you should also see that the values used for `start_id` and `map_id` match the IDs of those maps. The repeated pattern ends when there are no more maps left to look at, which results in `BPF_MAP_GET_NEXT_ID` returning a value of `ENOENT`, like this:

bpf(BPF\_MAP\_GET\_NEXT\_ID, {start\_id=133,...}, 12) = -1 ENOENT (No such file or
directory)

If a matching map has been found, `bpftool` holds its file descriptor so that it can read the elements out of that map.