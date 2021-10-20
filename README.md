# rocky8-microk8s

```./ansible-host-configuration.sh```  
```ansible-playbook -i hosts.ini -e @vars.yaml hosts_setup.yaml -K```


Known issues:
1. If pods are failing with Permission denied on PV run ```chmod ugo+rwX /var/snap/microk8s/common/default-storage/*```
2. If gitlab runner pod is failling with error message: ```Post https://gitlab.<your-host-name>/api/v4/runners: x509: certificate signed by unknown authority``` then edit gitlab-tls-secret in gitlab namespace and copy ```ca.crt`` to ```gitlab.<your-host-name>.crt```
