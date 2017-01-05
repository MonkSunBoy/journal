# 查看是否丢包或者发生错误

```
➜ monkboy >cat /proc/net/dev
Inter-|   Receive                                                |  Transmit
 face |bytes    packets errs drop fifo frame compressed multicast|bytes    packets errs drop fifo colls carrier compressed
    lo:1027302517064 7047577951    0    0    0     0          0         0 1027302517064 7047577951    0    0    0     0       0          0
   em1:19328344938734 97141607597    0   12    0    27          0 472311234 15368773298976 98277186097    0    0    0     0       0          0
   em2:       0       0    0    0    0     0          0         0        0       0    0    0    0     0       0          0
   em3:       0       0    0    0    0     0          0         0        0       0    0    0    0     0       0          0
   em4:       0       0    0    0    0     0          0         0        0       0    0    0    0     0       0          0
➜ monkboy >sudo ethtool -S em1 | grep rx     
     rx_octets: 19328804280873
     rx_fragments: 0
     rx_ucast_packets: 96632797518
     rx_mcast_packets: 472312819
     rx_bcast_packets: 38804237
     rx_fcs_errors: 0
     rx_align_errors: 0
     rx_xon_pause_rcvd: 0
     rx_xoff_pause_rcvd: 0
     rx_mac_ctrl_rcvd: 0
     rx_xoff_entered: 0
     rx_frame_too_long_errors: 0
     rx_jabbers: 0
     rx_undersize_packets: 0
     rx_in_length_errors: 0
     rx_out_length_errors: 0
     rx_64_or_less_octet_packets: 0
     rx_65_to_127_octet_packets: 0
     rx_128_to_255_octet_packets: 0
     rx_256_to_511_octet_packets: 0
     rx_512_to_1023_octet_packets: 0
     rx_1024_to_1522_octet_packets: 0
     rx_1523_to_2047_octet_packets: 0
     rx_2048_to_4095_octet_packets: 0
     rx_4096_to_8191_octet_packets: 0
     rx_8192_to_9022_octet_packets: 0
     rxbds_empty: 27
     rx_discards: 12
     rx_errors: 0
     rx_threshold_hit: 0
➜ monkboy >sudo ethtool -S em1 | grep tx
     tx_octets: 15369188472483
     tx_collisions: 0
     tx_xon_sent: 12
     tx_xoff_sent: 12
     tx_flow_control: 0
     tx_mac_errors: 0
     tx_single_collisions: 0
     tx_mult_collisions: 0
     tx_deferred: 0
     tx_excessive_collisions: 0
     tx_late_collisions: 0
     tx_collide_2times: 0
     tx_collide_3times: 0
     tx_collide_4times: 0
     tx_collide_5times: 0
     tx_collide_6times: 0
     tx_collide_7times: 0
     tx_collide_8times: 0
     tx_collide_9times: 0
     tx_collide_10times: 0
     tx_collide_11times: 0
     tx_collide_12times: 0
     tx_collide_13times: 0
     tx_collide_14times: 0
     tx_collide_15times: 0
     tx_ucast_packets: 98279682890
     tx_mcast_packets: 0
     tx_bcast_packets: 27025
     tx_carrier_sense_errors: 0
     tx_discards: 0
     tx_errors: 0
     tx_comp_queue_full: 0
     nic_tx_threshold_hit: 0
➜ monkboy >sudo ethtool -S em1               
NIC statistics:
     rx_octets: 19328703791081
     rx_fragments: 0
     rx_ucast_packets: 96632252715
     rx_mcast_packets: 472312441
     rx_bcast_packets: 38804213
     rx_fcs_errors: 0
     rx_align_errors: 0
     rx_xon_pause_rcvd: 0
     rx_xoff_pause_rcvd: 0
     rx_mac_ctrl_rcvd: 0
     rx_xoff_entered: 0
     rx_frame_too_long_errors: 0
     rx_jabbers: 0
     rx_undersize_packets: 0
     rx_in_length_errors: 0
     rx_out_length_errors: 0
     rx_64_or_less_octet_packets: 0
     rx_65_to_127_octet_packets: 0
     rx_128_to_255_octet_packets: 0
     rx_256_to_511_octet_packets: 0
     rx_512_to_1023_octet_packets: 0
     rx_1024_to_1522_octet_packets: 0
     rx_1523_to_2047_octet_packets: 0
     rx_2048_to_4095_octet_packets: 0
     rx_4096_to_8191_octet_packets: 0
     rx_8192_to_9022_octet_packets: 0
     tx_octets: 15369049763867
     tx_collisions: 0
     tx_xon_sent: 12
     tx_xoff_sent: 12
     tx_flow_control: 0
     tx_mac_errors: 0
     tx_single_collisions: 0
     tx_mult_collisions: 0
     tx_deferred: 0
     tx_excessive_collisions: 0
     tx_late_collisions: 0
     tx_collide_2times: 0
     tx_collide_3times: 0
     tx_collide_4times: 0
     tx_collide_5times: 0
     tx_collide_6times: 0
     tx_collide_7times: 0
     tx_collide_8times: 0
     tx_collide_9times: 0
     tx_collide_10times: 0
     tx_collide_11times: 0
     tx_collide_12times: 0
     tx_collide_13times: 0
     tx_collide_14times: 0
     tx_collide_15times: 0
     tx_ucast_packets: 98278810944
     tx_mcast_packets: 0
     tx_bcast_packets: 27025
     tx_carrier_sense_errors: 0
     tx_discards: 0
     tx_errors: 0
     dma_writeq_full: 0
     dma_write_prioq_full: 0
     rxbds_empty: 27
     rx_discards: 12
     rx_errors: 0
     rx_threshold_hit: 0
     dma_readq_full: 0
     dma_read_prioq_full: 0
     tx_comp_queue_full: 0
     ring_set_send_prod_index: 0
     ring_status_update: 0
     nic_irqs: 0
     nic_avoided_irqs: 0
     nic_tx_threshold_hit: 0
     mbuf_lwm_thresh_hit: 12
➜ monkboy >cat /proc/net/softnet_stat
62ce59b3 00000000 00045c26 00000000 00000000 00000000 00000000 00000000 00000000 00000000
82d1c5e6 00000000 00000004 00000000 00000000 00000000 00000000 00000000 00000000 00000000
759efa6c 00000000 0000005d 00000000 00000000 00000000 00000000 00000000 00000000 00000000
5ad49ad6 00000000 00000059 00000000 00000000 00000000 00000000 00000000 00000000 00000000
464485a4 00000000 00000067 00000000 00000000 00000000 00000000 00000000 00000000 00000000
2dac1286 00000000 00000000 00000000 00000000 00000000 00000000 00000000 00000000 00000000
6f7cf584 00000000 00000019 00000000 00000000 00000000 00000000 00000000 00000000 00000000
36ef1e8a 00000000 00000001 00000000 00000000 00000000 00000000 00000000 00000000 00000000
268c27e4 00000000 00000000 00000000 00000000 00000000 00000000 00000000 00000000 00000000
18c9b5ee 00000000 00000000 00000000 00000000 00000000 00000000 00000000 00000000 00000000
0f8d934e 00000000 00000000 00000000 00000000 00000000 00000000 00000000 00000000 00000000
0a4b83ac 00000000 00000000 00000000 00000000 00000000 00000000 00000000 00000000 00000000
```
