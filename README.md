# SpoLab Fleet Repository

This repository is a Flux fleet management tool. It allows you to bootstrap the basic infrastructure required for the petclinic demo to work.

## How to configure the local cluster

```bash
flux bootstrap github --owner=spolab --repository spolab/spolab-flux --path clusters/development --personal  
```

If you want to avoid using SSH (e.g. behind a corporate firewall) add the `--token-auth` option

If the corporate proxy (very likely) uses self-signed certificates, configure the certificate authority by adding `--ca-file <filename>` option.

How is the bootstrap process organized:
- 