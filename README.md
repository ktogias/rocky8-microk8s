# rocky8-microk8s

```./ansible-host-configuration.sh```  
```ansible-playbook -i hosts.ini -e @vars.yaml hosts_setup.yaml -K```

If pods are failing with Permission denied on PV run ```chmod ugo+rwX /var/snap/microk8s/common/default-storage/*```
