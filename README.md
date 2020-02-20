# gitops-helm-workshop

Workshop:



Sessions:

- [Helm Summit EU 2019](https://events.linuxfoundation.org/events/helm-summit-2019/)

Current state:

- all resources used during workshop for deploying podinfo are disabled
- repo contains active config for Gitlab-runner and Minio Helm charts as well as SealedSecrets.

## issue

I tried to run gitlab-runner with minio s3 using same approach but faced issues with minio resurces. At the beggining it seemed that `minio-make-bucket-job` stopped whole chart installation.

``` log
ts=2020-02-20T09:20:16.946257871Z caller=release.go:335 component=release release=minio targetNamespace=gitlab-runner resource=gitlab-runner:helmrelease/minio helmVersion=v3 info="no existing release" action=install
ts=2020-02-20T09:20:17.048138859Z caller=release.go:360 component=release release=gitlab-runner targetNamespace=gitlab-runner resource=gitlab-runner:helmrelease/gitlab-runner helmVersion=v3 info="performing dry-run upgrade to see if release has diverged"
ts=2020-02-20T09:20:17.050857027Z caller=helm.go:69 component=helm version=v3 info="preparing upgrade for gitlab-runner" targetNamespace=gitlab-runner release=gitlab-runner
ts=2020-02-20T09:20:17.288908949Z caller=helm.go:69 component=helm version=v3 info="creating 4 resource(s)" targetNamespace=gitlab-runner release=minio
ts=2020-02-20T09:20:17.417871988Z caller=helm.go:69 component=helm version=v3 info="creating 1 resource(s)" targetNamespace=gitlab-runner release=minio
ts=2020-02-20T09:20:17.444198192Z caller=helm.go:69 component=helm version=v3 info="performing update for gitlab-runner" targetNamespace=gitlab-runner release=gitlab-runner
ts=2020-02-20T09:20:17.466834408Z caller=helm.go:69 component=helm version=v3 info="dry run for gitlab-runner" targetNamespace=gitlab-runner release=gitlab-runner
ts=2020-02-20T09:20:17.499200139Z caller=release.go:216 component=release release=minio targetNamespace=gitlab-runner resource=gitlab-runner:helmrelease/minio helmVersion=v3 error="Helm release failed" revision=5.0.6 err="failed to upgrade chart for release [minio]: failed post-install: warning: Hook post-install minio/templates/post-install-create-bucket-job.yaml failed: jobs.batch \"minio-make-bucket-job\" already exists"
ts=2020-02-20T09:20:17.499463991Z caller=release.go:221 component=release release=minio targetNamespace=gitlab-runner resource=gitlab-runner:helmrelease/minio helmVersion=v3 info="uninstalling initial failed release so it can be retried"
ts=2020-02-20T09:20:17.533749482Z caller=helm.go:69 component=helm version=v3 info="uninstall: Deleting minio" targetNamespace=gitlab-runner release=minio
ts=2020-02-20T09:20:17.536043122Z caller=release.go:404 component=release release=gitlab-runner targetNamespace=gitlab-runner resource=gitlab-runner:helmrelease/gitlab-runner helmVersion=v3 info="no changes" action=skip
ts=2020-02-20T09:20:17.665992196Z caller=release.go:360 component=release release=nginx-ingress targetNamespace=ingress-nginx resource=ingress-nginx:helmrelease/nginx-ingress helmVersion=v3 info="performing dry-run upgrade to see if release has diverged"
ts=2020-02-20T09:20:17.670530231Z caller=helm.go:69 component=helm version=v3 info="preparing upgrade for nginx-ingress" targetNamespace=ingress-nginx release=nginx-ingress
ts=2020-02-20T09:20:17.731192192Z caller=helm.go:69 component=helm version=v3 info="Starting delete for \"minio\" Service" targetNamespace=gitlab-runner release=minio
ts=2020-02-20T09:20:17.768909807Z caller=helm.go:69 component=helm version=v3 info="Starting delete for \"minio\" Deployment" targetNamespace=gitlab-runner release=minio
ts=2020-02-20T09:20:17.779547137Z caller=helm.go:69 component=helm version=v3 info="Starting delete for \"minio\" ServiceAccount" targetNamespace=gitlab-runner release=minio
ts=2020-02-20T09:20:17.789764173Z caller=helm.go:69 component=helm version=v3 info="Starting delete for \"minio\" ConfigMap" targetNamespace=gitlab-runner release=minio
ts=2020-02-20T09:20:17.808097922Z caller=helm.go:69 component=helm version=v3 info="purge requested for minio" targetNamespace=gitlab-runner release=minio
```

However when I used `helm install` command all things went ok. Minio chart creates temporary pod that is responsinble for doing some preparation tasks. Calling `describe` for that pod revealed that minio configmap was not created when using HelmRelease. However, it turned out that it was created and mounted into minio pod but from unknown reason it dissappered after 2-3 minutes from installation. Manual configmap creation was also not an option as HelmRelease complained about that. Calling `get events` revealed huge number of __Linkerd__ events, so I decided to remove service mesh that was in place after doing tutorial. That solved problem with Minio that was now able to be installed using Helm Release.

However problem is still open. _Why Helm Operator installing Minio experienced such issues while Gitlab-runner didn't in presence of Linkerd?_
