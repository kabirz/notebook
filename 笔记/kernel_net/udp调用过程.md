
## 用户层

```c
#include <stdlib.h>
#include <string.h>
#include <stdio.h>
#include <arpa/inet.h>
#include <netinet/in.h>
#include <sys/socket.h>
#include <unistd.h>

int main(int argc, char *argv[])
{
	char buf[64] = {0};
	int fd = socket(AF_INET, SOCK_DGRAM, 0);
	if (fd < 0) return -1;

	char ip_str[16] = "10.239.71.23";
	if (argc > 1)
		strncpy(ip_str, argv[1], strlen(argv[1]));
	struct sockaddr_in ssi = {
		.sin_addr.s_addr = inet_addr(ip_str),
		.sin_port = htons(5678),
		.sin_family = AF_INET,
	}; // 目标ip和port

	for (int i = 0; i < 10; i++) {
		snprintf(buf, sizeof(buf), "%d. UDP client", i+1);
		sendto(fd, buf, sizeof((buf)), 0, (struct sockaddr *)&ssi, sizeof(ssi)); // 发送
		printf("Send(%s:%d)%s\n", inet_ntoa(ssi.sin_addr), ntohs(ssi.sin_port), buf);
	}

	close(fd);

	return EXIT_SUCCESS;
}
```

用户层调用的是
```c
ssize_t sendto(int sockfd, const void *buf, size_t len, int flags,
			   const struct sockaddr *dest_addr, socklen_t addrlen);
```

对应linux内核中`net/socket.c`的系统调用 [sendto](https://github.com/torvalds/linux/blob/ffd2cb6b718e189e7e2d5d0c19c25611f92e061a/net/socket.c#L2199)
```c
SYSCALL_DEFINE6(sendto, int, fd, void __user *, buff, size_t, len,
		unsigned int, flags, struct sockaddr __user *, addr,
		int, addr_len)
{
	return __sys_sendto(fd, buff, len, flags, addr, addr_len);
}
```

调用`__sys_sendto`
```c
int __sys_sendto(int fd, void __user *buff, size_t len, unsigned int flags,
		 struct sockaddr __user *addr,  int addr_len)
{
	struct socket *sock;
	struct sockaddr_storage address;
	int err;
	struct msghdr msg;
	int fput_needed;
	......
	sock = sockfd_lookup_light(fd, &err, &fput_needed); // 根据文件描述符找到socket
	if (!sock)
		goto out;

	msg.msg_name = NULL;
	msg.msg_control = NULL;
	msg.msg_controllen = 0;
	msg.msg_namelen = 0;
	msg.msg_ubuf = NULL;
	if (addr) {
		err = move_addr_to_kernel(addr, addr_len, &address); // 把接收方地址信息copy到内核
		if (err < 0)
			goto out_put;
		msg.msg_name = (struct sockaddr *)&address;
		msg.msg_namelen = addr_len;
	}
	flags &= ~MSG_INTERNAL_SENDMSG_FLAGS;
	if (sock->file->f_flags & O_NONBLOCK)
		flags |= MSG_DONTWAIT;
	msg.msg_flags = flags;
	err = __sock_sendmsg(sock, &msg); // 发送message

out_put:
	fput_light(sock->file, fput_needed);
out:
	return err;
}
```

`__sock_sendmsg`调用`sock_sendmsg_nosec`
```c
static inline int sock_sendmsg_nosec(struct socket *sock, struct msghdr *msg)
{
	int ret = INDIRECT_CALL_INET(READ_ONCE(sock->ops)->sendmsg, inet6_sendmsg,
				     inet_sendmsg, sock, msg,
				     msg_data_left(msg));
	.....
}
```
ipv4 调用`inet_sendmsg`, `inet_sendmsg`根据协议udp调用函数`udp_sendmsg`
```c
int udp_sendmsg(struct sock *sk, struct msghdr *msg, size_t len)
{
	struct inet_sock *inet = inet_sk(sk);
	struct udp_sock *up = udp_sk(sk);
	DECLARE_SOCKADDR(struct sockaddr_in *, usin, msg->msg_name);
	struct flowi4 fl4_stack;
	struct flowi4 *fl4;
	int ulen = len;
	......

	fl4 = &inet->cork.fl.u.ip4;
	ulen += sizeof(struct udphdr); // 添加udp头长度

	......
	daddr = usin->sin_addr.s_addr;
	dport = usin->sin_port;
	......

	saddr = ipc.addr;
	ipc.addr = faddr = daddr;
	.....
	saddr = fl4->saddr;
	if (!ipc.addr)
		daddr = ipc.addr = fl4->daddr;

	/* Lockless fast path for the non-corking case. */
	if (!corkreq) {
		struct inet_cork cork;

		skb = ip_make_skb(sk, fl4, getfrag, msg, ulen,
				  sizeof(struct udphdr), &ipc, &rt,
				  &cork, msg->msg_flags);
		err = PTR_ERR(skb);
		if (!IS_ERR_OR_NULL(skb))
			err = udp_send_skb(skb, fl4, &cork); // 发送udp包
		goto out;
	}
	......
}
```
`udp_send_skb`: udp发送skb
```c
static int udp_send_skb(struct sk_buff *skb, struct flowi4 *fl4,
			struct inet_cork *cork)
{
	struct sock *sk = skb->sk;
	struct inet_sock *inet = inet_sk(sk);
	struct udphdr *uh;
	int err;
	int is_udplite = IS_UDPLITE(sk);
	int offset = skb_transport_offset(skb);
	int len = skb->len - offset;
	int datalen = len - sizeof(*uh);
	__wsum csum = 0;

	/*
	 * Create a UDP header
	 */
	uh = udp_hdr(skb); // 获取udp头的位置
	uh->source = inet->inet_sport; // 设置源端口
	uh->dest = fl4->fl4_dport; // 设置目的端口
	uh->len = htons(len); // 设置udp包的长度， 包含udp头
	uh->check = 0; // 校验值在后续计算
	......
	csum = udp_csum(skb); // 计算checksum值

	/* add protocol-dependent pseudo-header */
	uh->check = csum_tcpudp_magic(fl4->saddr, fl4->daddr, len,
				      sk->sk_protocol, csum); // 计算check
	if (uh->check == 0)
		uh->check = CSUM_MANGLED_0;

send:
	err = ip_send_skb(sock_net(sk), skb); // 数据包到ip层(网络层)处理
	......
}
```
网络层调用`ip_local_out`函数
```c
int ip_local_out(struct net *net, struct sock *sk, struct sk_buff *skb)
{
	int err;

	err = __ip_local_out(net, sk, skb);
	if (likely(err == 1))
		err = dst_output(net, sk, skb);

	return err;
}

int __ip_local_out(struct net *net, struct sock *sk, struct sk_buff *skb)
{
	struct iphdr *iph = ip_hdr(skb); // ip头

	IP_INC_STATS(net, IPSTATS_MIB_OUTREQUESTS);

	iph_set_totlen(iph, skb->len);
	ip_send_check(iph);
	......

	skb->protocol = htons(ETH_P_IP); // 设置为ip协议包

	return nf_hook(NFPROTO_IPV4, NF_INET_LOCAL_OUT,
		       net, sk, skb, NULL, skb_dst(skb)->dev,
		       dst_output);
}

static inline int dst_output(struct net *net, struct sock *sk, struct sk_buff *skb)
{
	return INDIRECT_CALL_INET(skb_dst(skb)->output, ip6_output, ip_output, net, sk, skb);
}
```
调用`ip_output`
```c
int ip_output(struct net *net, struct sock *sk, struct sk_buff *skb)
{
	struct net_device *dev = skb_dst(skb)->dev, *indev = skb->dev;

	skb->dev = dev;
	skb->protocol = htons(ETH_P_IP);

	return NF_HOOK_COND(NFPROTO_IPV4, NF_INET_POST_ROUTING,
			    net, sk, skb, indev, dev,
			    ip_finish_output,
			    !(IPCB(skb)->flags & IPSKB_REROUTED));
}
```
`ip_finish_output`调用`__ip_finish_output`
```c
static int __ip_finish_output(struct net *net, struct sock *sk, struct sk_buff *skb)
{
	unsigned int mtu;
	
	mtu = ip_skb_dst_mtu(sk, skb);
	if (skb_is_gso(skb))
		return ip_finish_output_gso(net, sk, skb, mtu);

	if (skb->len > mtu || IPCB(skb)->frag_max_size)
		return ip_fragment(net, sk, skb, mtu, ip_finish_output2);

	return ip_finish_output2(net, sk, skb);
}
```

`ip_finish_output2`
```c
static int ip_finish_output2(struct net *net, struct sock *sk, struct sk_buff *skb)
{
	......

	rcu_read_lock();
	neigh = ip_neigh_for_gw(rt, skb, &is_v6gw); // arp
	if (!IS_ERR(neigh)) {
		int res;

		sock_confirm_neigh(skb, neigh);
		/* if crossing protocols, can not use the cached header */
		res = neigh_output(neigh, skb, is_v6gw); // 
		rcu_read_unlock();
		return res;
	}
	rcu_read_unlock();

	net_dbg_ratelimited("%s: No header cache and no neighbour!\n",
			    __func__);
	kfree_skb_reason(skb, SKB_DROP_REASON_NEIGH_CREATEFAIL);
	return PTR_ERR(neigh);
}

```

`neigh_output`
```c
static inline int neigh_output(struct neighbour *n, struct sk_buff *skb,
			       bool skip_cache)
{
	const struct hh_cache *hh = &n->hh;

	/* n->nud_state and hh->hh_len could be changed under us.
	 * neigh_hh_output() is taking care of the race later.
	 */
	if (!skip_cache &&
	    (READ_ONCE(n->nud_state) & NUD_CONNECTED) &&
	    READ_ONCE(hh->hh_len))
		return neigh_hh_output(hh, skb); // 有缓存时

	return READ_ONCE(n->output)(n, skb); // 调用neigh_resolve_output
}
```

`neigh_resolve_output`:
```c
int neigh_resolve_output(struct neighbour *neigh, struct sk_buff *skb)
{
	int rc = 0;

	if (!neigh_event_send(neigh, skb)) {
		......
		do {
			__skb_pull(skb, skb_network_offset(skb));
			seq = read_seqbegin(&neigh->ha_lock);
			err = dev_hard_header(skb, dev, ntohs(skb->protocol),
					      neigh->ha, NULL, skb->len); // 设置以太网头
		} while (read_seqretry(&neigh->ha_lock, seq));

		if (err >= 0)
			rc = dev_queue_xmit(skb); // 发送
		else
			goto out_kfree_skb;
	}
	......
}
```
`dev_hard_header`调用`dev->header_ops->create`是 `eth_header`(net/ethernet/eth.c)
```c
int eth_header(struct sk_buff *skb, struct net_device *dev,
	       unsigned short type,
	       const void *daddr, const void *saddr, unsigned int len)
{
	struct ethhdr *eth = skb_push(skb, ETH_HLEN);

	if (type != ETH_P_802_3 && type != ETH_P_802_2)
		eth->h_proto = htons(type);
	else
		eth->h_proto = htons(len);

	if (!saddr)
		saddr = dev->dev_addr;
	memcpy(eth->h_source, saddr, ETH_ALEN);

	if (daddr) {
		memcpy(eth->h_dest, daddr, ETH_ALEN);
		return ETH_HLEN;
	}

	/*
	 *      Anyway, the loopback-device should never use this function...
	 */

	if (dev->flags & (IFF_LOOPBACK | IFF_NOARP)) {
		eth_zero_addr(eth->h_dest);
		return ETH_HLEN;
	}

	return -ETH_HLEN;
}
```
`dev_queue_xmit` ->`__dev_queue_xmit` ->
`__dev_xmit_skb` -> `dev_hard_start_xmit` -> `xmit_one` -> `netdev_start_xmit` ->
`__netdev_start_xmit` -> `ops->ndo_start_xmit(skb, dev);`
如驱动dm9000

```c
static const struct net_device_ops dm9000_netdev_ops = {
        .ndo_open               = dm9000_open,
        .ndo_stop               = dm9000_stop,
        .ndo_start_xmit         = dm9000_start_xmit, // 发送
        .ndo_tx_timeout         = dm9000_timeout,
        .ndo_set_rx_mode        = dm9000_hash_table,
        .ndo_eth_ioctl          = dm9000_ioctl,
        .ndo_set_features       = dm9000_set_features,
        .ndo_validate_addr      = eth_validate_addr,
        .ndo_set_mac_address    = eth_mac_addr,
};
```
参考
[【驱动】DM9000网卡驱动分析 - Leo.cheng - 博客园 (cnblogs.com)](https://www.cnblogs.com/lcw/p/3159378.html)
[图解 Linux 网络包发送过程 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/373060740)
