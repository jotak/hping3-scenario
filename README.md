## hping3 / SYN-flood scenario with netobserv

1. Install netobserv, with agent privileged, sampling=1
2. Deploy hping3

```bash
oc apply -f ./hping3.yaml
# (I'm reusing here netobserv agent service account by laziness because it's already privileged - hping3 needs to be privileged)
```

3. Install FlowMetrics and Alert

```bash
oc apply -f flows_with_flags_per_source.yaml 
oc apply -f flows_with_flags_per_destination.yaml 
oc apply -f syn-alert.yaml 
```

4. Run SYN flood

```bash
# replace hping3-9dfc449d4-f7p4h with actual pod name, and 172.30.31.170 with the IP you target
# E.g:
oc -n netobserv-privileged exec -it hping3-9dfc449d4-f7p4h -- hping3 -c 1500 -d 10 -S -w 64 -p 80 --flood --rand-source 172.30.31.170
```

5. Check OCP console / alerts: you should have NetObserv-SYNFlood-in triggered. You can also check metrics with promQL:

```
topk(5,rate(netobserv_flows_with_flags_per_destination_total{DstK8S_Namespace!="",Flags="2"}[1m]))
```
