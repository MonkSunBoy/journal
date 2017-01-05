# system info

```
system_information:to_file("si.txt").
recon_alloc:fragmentation(current).
[{K, V/1024/1024} || {K, V} <- recon_alloc:memory(allocated_types)].
elang:memory().
A = [ proplists:get_value(name, erlang:port_info(P))|| P <- erlang:ports(), is_list(erlang:port_info(P))].
```
