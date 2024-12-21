# Connecting a Network Namespace to Host Using a Virtual Ethernet Cable

This guide explains how to connect a network namespace (ns) to the host using a virtual Ethernet (veth) cable. This method can be used to set up isolated network environments while still providing connectivity to the host system.

---

## Prerequisites
1. Root or sudo access.
2. `iproute2` tools installed on the system.

---
![Alt Text](https://github.com/cloudybdone/NetworkNamespace/blob/main/pic1.png)


## Steps

### 1. Create a Custom Network Namespace

First, create a custom namespace using the `ip netns add` command:
```bash
sudo ip netns add red
sudo ip netns list
```
![Alt Text](https://github.com/cloudybdone/NetworkNamespace/blob/main/pic2.png)

This creates a new namespace called `red`.

---

### 2. Create a Virtual Ethernet Cable

Create a veth pair in the root namespace:
```bash
sudo ip link add veth-red type veth peer name veth-host
```
This creates a pair of interconnected virtual Ethernet devices (`veth-red` and `veth-host`) in the root namespace.

To list all links, use:
```bash
ip link list
```

---

### 3. Move One End of the Cable to the Namespace

Move one end of the veth pair (`veth-red`) into the `red` namespace:
```bash
sudo ip link set veth-red netns red
```
Now:
- `veth-red` resides in the `red` namespace.
- `veth-host` remains in the root namespace.

---

### 4. Configure IP Addresses for the Veth Pair

#### In the `red` Namespace:
```bash
sudo ip netns exec red ip addr add 192.168.1.1/24 dev veth-red
sudo ip netns exec red ip link set veth-red up
```

#### In the Root Namespace:
```bash
sudo ip addr add 192.168.1.2/24 dev veth-host
sudo ip link set veth-host up
```

---
![Alt Text](https://github.com/cloudybdone/NetworkNamespace/blob/main/pic3.png)

### 5. Add a Route on the Host

Add a route on the host to direct traffic destined for `192.168.1.1` through the `veth-host` interface:
```bash
sudo ip route add 192.168.1.1 dev veth-host
```

#### Why Add a Route?
Without adding the route, the host won't know how to reach `192.168.1.1`. The routing table must be explicitly updated to use `veth-host` for traffic to this destination.

---

### 6. Test Connectivity

Ping the `red` namespace from the host:
```bash
ping 192.168.1.1 -c 3
```

If everything is configured correctly, you should see successful ping responses.

---

## Summary
By following these steps, you've:
- Created a custom namespace (`red`).
- Connected the namespace to the host using a virtual Ethernet cable.
- Configured IP addresses and routes for communication.

This setup is useful for network simulations, container-like environments, and testing isolated networks.

---

## Notes
- Use `sudo ip netns delete red` to delete the namespace when no longer needed.
- Ensure that IP addresses and routes do not conflict with existing network configurations.

---

## References
- [iproute2 Documentation](https://man7.org/linux/man-pages/man8/ip.8.html)
- [Linux Network Namespaces](https://wiki.archlinux.org/title/Network_namespace)
