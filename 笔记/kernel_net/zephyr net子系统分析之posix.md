
zephyr 支持posix访问网络
`lib/posix/options/net.c`

socket调用过程
socket -->
		zsock_socket -->
`subsys/net/lib/sockets/sockets.c` (`z_impl_zsock_socket`) -->
