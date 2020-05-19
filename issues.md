# Issues

## Operator Hub

Currently the Operator Hub is unable to display/populate.  The following error is noted in the system.

```
root@greedo:~# oc get opsrc -n openshift-marketplace
NAME                  TYPE          ENDPOINT              REGISTRY              DISPLAYNAME           PUBLISHER   STATUS        MESSAGE                                                                                                                                        AGE
certified-operators   appregistry   https://quay.io/cnr   certified-operators   Certified Operators   Red Hat     Configuring   Get https://quay.io/cnr/api/v1/packages?namespace=certified-operators: x509: certificate is valid for *.apps.greedo.coc1-ibm.us, not quay.io   5h18m
community-operators   appregistry   https://quay.io/cnr   community-operators   Community Operators   Red Hat     Configuring   Get https://quay.io/cnr/api/v1/packages?namespace=community-operators: x509: certificate is valid for *.apps.greedo.coc1-ibm.us, not quay.io   5h18m
redhat-operators      appregistry   https://quay.io/cnr   redhat-operators      Red Hat Operators     Red Hat     Configuring   Get https://quay.io/cnr/api/v1/packages?namespace=redhat-operators: x509: certificate is valid for *.apps.greedo.coc1-ibm.us, not quay.io      5h18m
root@greedo:~#



```