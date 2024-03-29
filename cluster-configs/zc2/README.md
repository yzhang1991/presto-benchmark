# `zc2` cluster
custom (vCPU: 64, Memory: 712 GB) * 28

### Global
* `SysReservedGb = 2`
* `ContainerMemoryGb = MemoryPerNodeGb - ceil(SysReservedGb) = 712 - ceil(2) = 710` [[docker-stack-java.yaml](docker-stack-java.yaml)] and [[docker-stack-native.yaml](docker-stack-native.yaml)]
### For Java clusters:
* `HeapSizeGb = floor(ContainerMemory * HeapSizePercentOfContainerMem) = floor(710 * 0.9) = 639` (`-Xmx` and `-Xms` in [[coordinator jvm.config](coordinator/jvm.config)] and [[worker jvm.config](workers/jvm.config)])
* [[coordinator config.properties](coordinator/config.properties)] and [[worker config.properties](worker/config.properties)]
  * `memory.heap-headroom-per-node = ceil(HeapSizeGb * HeadroomPercentOfHeap) = ceil(639 * 0.2) = 128`
  * `query.max-total-memory-per-node = floor(HeapSizeGb * QueryMaxTotalMemPerNodePercentOfHeap) = floor(639 * 0.8) = 511`
  * `query.max-memory-per-node = floor(query.max-total-memory-per-node * QueryMaxMemPerNodePercentOfTotal) = floor(511 * 0.9) = 459`
  * `query.max-total-memory = query.max-total-memory-per-node * NumberOfWorkers = 511 * 28 = 14308`
  * `query.max-memory = query.max-memory-per-node * NumberOfWorkers = 459 * 28 = 12852` [[documentation](https://prestodb.io/docs/current/admin/properties.html#memory-management-properties)]
### For Prestissimo clusters:
* Coordinator heap setting same as Java cluster
* `NativeProxygenMemGb = ceil(min(ProxygenMemPerWorkerGb * NumberOfWorkers, ProxygenMemCapGb)) = ceil(min(0.125 * 28, 2)) = 2`
* `NonVeloxBufferMemGb = 8`
* `system-memory-gb = ContainerMemory - NativeProxygenMemGb - ceil(NonVeloxBufferMemGb) = 710 - 2 - ceil(8) = 700`
* `query-memory-gb = query.max-memory-per-node = floor(system-memory-gb * NativeQueryMemPercentOfSysMem) = floor(700 * 0.95) = 665`
