
1. Which resource record would you use to set up resolving for names to IPv6
	addresses?
	**AAAA**
2. Mail servers do not succeed in delivering messages to your domain. Which
	resource record is probably not configured correctly?
	**MX**
3. Which command enables you to verify the current mail server setting for
	example.com?
	**dig MX example.com**
4. Which command enables you to find the name that is configured in DNS for
	IP address 192.168.4.122?
	**dig -x 192.168.4.122**
5. Which setting do you configure in unbound.conf to make sure that all requests
	your caching server receives are forwarded to the DNS server with IP address
	10.0.0.100?
	**forward-zone:**
		**name: "."**
		**forward-addr: 10.0.0.100**
6. What do you need to do to make sure that all clients on the 192.168.122.0/24
	network can use your unbound DNS server?
	**access-control 192.168.122.0./24 allow**
7. Which parameter do you need to use to exclude a zone that is not signed with
	DNSSEC keys?
	**dnssec-insecure**: DOMAIN
8. Which parameter do you configure in unbound.conf to make sure the
	unbound name server is listening on real interfaces and not just the localhost
	interface?
	**interface: 0.0.0.0**
9. Which command enables you to verify that you have not made any typos in
	the unbound configuration?
	**unbound-checkconf**
10. Which command enables you to dump the unbound DNS cache so that it can
	be further analyzed
	**unbound-control dump_cache** > file
