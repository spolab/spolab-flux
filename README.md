# SpoLab Fleet Repository

## How to configure the local cluster

```bash
flux bootstrap github --owner=spolab --repository spolab/spolab-flux --path clusters/development --personal  
```
Note that if you add applications with an error in this repository the whole bootstrap process will fail. Looks bizarre, though.