# CAVEAT EMPTOR

Like all other things in this repository, the goal is to provide proof-of-concept material. IT IS NOT PRODUCTION READY. USE AT YOUR OWN RISK.

# What problem is this repository solving?

This repository follows the classic `apps`, `infrastructure` and `clusters` structure recommended [here](https://fluxcd.io/flux/guides/repository-structure/), specifically the one repo per team model. In addition to that, we want to address how (more or less) to elegantly solve the problem of atomic updates when a resource depends on the CRDs generated or deployed by another one. This problem is described in this [FluxCD discussion](https://github.com/fluxcd/flux2/discussions/1311). 

Assume you have two `HelmRelease`s and one generates `CRD`s that the other one depends from. At present, even a `dependsOn` clause will not work. Currently there are a number of ways you can 'work around' the problem:

 ## Explicitly deploy the CRDS

 Kustomizations allow the user to specify where the CRDs are located and what are the creation/update policies. This is not always optimal, because many tools nowadays dynamically deploy the `CustomResourceDefinition`. It also requires the team to check any new release and see if there are CRDs that have to be extracted.
 
 ## Separate provider and consumer in different Kustomizations 
 
 Imagine you have a `HelmRelease`, **A**, that provides the CRDs, and a `HelmRelease`, **B**, that depends from those `CRD`. The team can install A in a Kustomization and B in a separate one, making sure that the latter Kustomization having a dependency on the former (`dependsOn` clause). Because deployment atomicity is guaranteed at the single Kustomization level, each Kustomization will run a separate dry-run process. The problem with this approach is that the number of separate Kustomizations may grow very quickly, making performance and maintainability an issue.

## Solution presented

This repository proposes a variant to the latter approach that keeps the number of required kustomizations under control.

Bootstrap happens in the `clusters/development` folder. As you can see, there are three Kustomizations, `infra-0`, `infra-1`, and `apps`. If you open the files, you will see that `infra-1` and `apps` have a `dependsOn` clause that generate the follwoing chain of dependencies among Kustomizations:

```
infra-0 -> infra-1 -> apps
```

Think of `-0` and `-1` as something similar to runlevels. To reach a certain runlevel, you must have executed all the previous ones.

The `infra-0` and `infra-1` Kustomizations point, respectively, to the `infra/0` and `infra/1` folders. These folders contain a file, `kustomization.yaml`, that tells FluxCD which HelmReleases need to be deployed in that "runlevel". For example, you can see that crossplane gets deployed in runlevel-0 but the AWS `Provider`, which depends on crossplane itself (`Provider` is a crossplane `CRD`), gets deployed in runlevel 1. 

With the approach just described, you can limit the number of separate Kustomizations to the maximum depth of transitive dependencies between resources.

# Installation

Make sure that you have the `flux` command installed, and an empty Kubernetes cluster (EKS, AKS, or a local K3S node). Flux installation instructions can be found [here](https://fluxcd.io/flux/installation/).

Once the CLI is installed, make sure that you are pointing to the right context with a 
 bash
 ```
 kubectl config current-context
 ```
 
Once you are sure you are pointing at the right cluster, bootstrap Flux by issuing the following command: 

```bash
flux bootstrap github --owner=spolab --repository spolab/spolab-flux --path clusters/development --personal  
```

If you want to avoid using SSH (e.g. behind a corporate firewall) add the `--token-auth` option

If the corporate proxy (very likely) uses self-signed certificates, configure the certificate authority by adding `--ca-file <filename>` option.

# Questions, suggestions, corrections

I am always happy to receive PRs. If you want to open a discussion or signal an issue, please use GitHub for that.

Hope this helps!