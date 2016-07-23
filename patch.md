Content: SO_ORIGINAL_DST, python3 support on listener, dynamic listening port, iptables result check

Future: possibility to run without root nor iptables

```
iptables -F OUTPUT
iptables -A OUTPUT -p tcp --dport 25 -j ACCEPT
iptables -A OUTPUT -p tcp --dport 30 -j ACCEPT
iptables -A OUTPUT -p tcp -d 192.168.1.68 -j REJECT
```

```
root@egress-attacker:~/egressbuster# ./egress_listener.py 192.168.1.68 eth0
[*] Inserting iptables rule to redirect **all TCP ports** to port TCP 1090
[*] Listening on all TCP ports now... Press control-c when finished.
[*] Connected from 192.168.1.70 on port: TCP 25
[*] Connected from 192.168.1.70 on port: TCP 30
```

```
root@egress-target:~/egressbuster# ./egressbuster.py 192.168.1.68 1-100
[i] Sending packets to egress listener (192.168.1.68)...
[i] Starting at: TCP 1, ending at: TCP 100
[*] Connection made to 192.168.1.68 on port: TCP 25
[*] Connection made to 192.168.1.68 on port: TCP 30
[*] All packets have been sent
[i] Remaining threads: 89
[*] Done
```

of course I could do like this:

```
root@egress-target:~/egressbuster# for i in $(seq 24 31); do echo $i | nc -n 192.168.1.68 $i; done
(UNKNOWN) [192.168.1.68] 24 (?) : Connection refused
(UNKNOWN) [192.168.1.68] 26 (?) : Connection refused
(UNKNOWN) [192.168.1.68] 27 (?) : Connection refused
(UNKNOWN) [192.168.1.68] 28 (?) : Connection refused
(UNKNOWN) [192.168.1.68] 29 (?) : Connection refused
(UNKNOWN) [192.168.1.68] 31 (?) : Connection refused
```

```
root@egress-attacker:~/egressbuster# ./egress_listener.py 192.168.1.68 eth0
[*] Inserting iptables rule to redirect **all TCP ports** to port TCP 1090
[*] Listening on all TCP ports now... Press control-c when finished.
[*] Connected from 192.168.1.70 on port: TCP 25
[*] Connected from 192.168.1.70 on port: TCP 30
```

but why, if I could simply update the listener side? :)

```
root@egress-attacker:~/egressbuster# ./egress_listener.py 192.168.1.68 eth0
[*] Inserting iptables rule to redirect **all TCP ports** to port TCP 59950
[*] Listening on all TCP ports now... Press control-c when finished.
[*] Connected from 192.168.1.70 on port: TCP 30 (client reported )
[*] Connected from 192.168.1.70 on port: TCP 25 (client reported )
[*] Connected from 192.168.1.70 on port: TCP 25 (client reported 25)
[*] Connected from 192.168.1.70 on port: TCP 30 (client reported 30)
```

```
root@egress-target:~/egressbuster# nc -nz 192.168.1.68 25-30
root@egress-target:~/egressbuster# ./egressbuster.py 192.168.1.68 25-30
[i] Sending packets to egress listener (192.168.1.68)...
[i] Starting at: TCP 25, ending at: TCP 30
[*] Connection made to 192.168.1.68 on port: TCP 25
[*] Connection made to 192.168.1.68 on port: TCP 30
[*] All packets have been sent
[i] Remaining threads: 4
[*] Done
```
