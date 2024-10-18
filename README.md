# Gloo-SP-6994 Reproducer

Issue: https://github.com/solo-io/solo-projects/issues/6994


## Installation

Add Gloo EE Helm repo:
```
helm repo add glooe https://storage.googleapis.com/gloo-ee-helm
```

Add Gloo Portal Helm repo:
```
helm repo add gloo-portal https://storage.googleapis.com/dev-portal-helm
```

Export your Gloo Edge License Key to an environment variable:
```
export GLOO_EDGE_LICENSE_KEY={your license key}
```

Install Gloo Edge and Gloo Portal:
```
cd install
./install-gloo-edge-enterprise-portal-with-helm.sh
```

> NOTE
> The Gloo Edge and Gloo Portal versions that will be installed are set in two variable at the top of the `install/install-gloo-edge-enterprise-portal-with-helm.sh` installation script.

## Deploy the APIDoc with invalid OpenAPI Spec

Deploy the APIDoc:

```
kubectl apply -f apidocs/oas-3_0_1-only-path-wrong-indentation-apidoc.yaml
```

Observe that, after deploying this `APIDoc` with an invalid OpenAPI-Specification defintion, the `gloo-portal-controller` in the `gloo-portal` namespace will crash and print errors in its logs:

```
kubectl -n gloo-portal logs -f deploy/gloo-portal-controller
```

```
{"level":"info","ts":"2024-10-18T20:35:46.757Z","caller":"controller/controller.go:115","msg":"Observed a panic in reconciler: runtime error: invalid memory address or nil pointer dereference","controll │
│ panic: runtime error: invalid memory address or nil pointer dereference [recovered]                                                                                                                        │
│     panic: runtime error: invalid memory address or nil pointer dereference                                                                                                                                │
│ [signal SIGSEGV: segmentation violation code=0x1 addr=0x0 pc=0x3306b7c]                                                                                                                                    │
│                                                                                                                                                                                                            │
│ goroutine 434 [running]:                                                                                                                                                                                   │
│ sigs.k8s.io/controller-runtime/pkg/internal/controller.(*Controller).Reconcile.func1()                                                                                                                     │
│     /home/runner/go/pkg/mod/sigs.k8s.io/controller-runtime@v0.16.3/pkg/internal/controller/controller.go:116 +0x30a                                                                                        │
│ panic({0x3978260?, 0x5802e40?})                                                                                                                                                                            │
│     /opt/hostedtoolcache/go/1.21.10/x64/src/runtime/panic.go:920 +0x290                                                                                                                                    │
│ github.com/solo-io/dev-portal/pkg/routing/parser/openapi.parseOperations({0xc000b29040, 0xc000b29040}, 0xc000b29040, 0xc000f784f8)                                                                         │
│     /home/runner/work/dev-portal/dev-portal/pkg/routing/parser/openapi/openapi_parser.go:138 +0x3dc                                                                                                        │
│ github.com/solo-io/dev-portal/pkg/routing/parser/openapi.(*openApiParser).ParseOperations(0xc0009008e0, {0x4174348, 0xc000758870}, 0xc000b29040)                                                           │
│     /home/runner/work/dev-portal/dev-portal/pkg/routing/parser/openapi/openapi_parser.go:100 +0xe5                                                                                                         │
│ github.com/solo-io/dev-portal/pkg/controllers/portal/reconcilers/apidoc.(*apiDocSyncer).addOpenAPIDocInfoToStatus(0xc000a48380, 0xc000f789c8, 0xc000a40960, {0xc000f7e400, 0x81f, 0x900}, 0x0)             │
│     /home/runner/work/dev-portal/dev-portal/pkg/controllers/portal/reconcilers/apidoc/apidoc_syncer.go:159 +0x1ce                                                                                          │
│ github.com/solo-io/dev-portal/pkg/controllers/portal/reconcilers/apidoc.(*apiDocSyncer).makeAPIDocStatus(0xc000a48380, 0xc000a40960)                                                                       │
│     /home/runner/work/dev-portal/dev-portal/pkg/controllers/portal/reconcilers/apidoc/apidoc_syncer.go:140 +0xaf1                                                                                          │
│ github.com/solo-io/dev-portal/pkg/controllers/portal/reconcilers/apidoc.(*apiDocSyncer).SyncAPIDoc(0xc000a48380, 0xc000a40960)                                                                             │
│     /home/runner/work/dev-portal/dev-portal/pkg/controllers/portal/reconcilers/apidoc/apidoc_syncer.go:48 +0x65                                                                                            │
│ github.com/solo-io/dev-portal/pkg/api/portal.gloo.solo.io/v1beta1/controller.genericAPIDocSyncer.Sync({{0x4174348, 0xc000758870}, {0x4154170, 0xc000a48380}}, {0x418b2e8, 0xc000a40960})                   │
│     /home/runner/work/dev-portal/dev-portal/pkg/api/portal.gloo.solo.io/v1beta1/controller/reconciler_impl.go:330 +0xe6                                                                                    │
│ github.com/solo-io/dev-portal/codegen/lib/reconciler/base.(*reconciler).Reconcile(0xc000a48400, {0x418b2e8, 0xc000a40960})                                                                                 │
│     /home/runner/work/dev-portal/dev-portal/codegen/lib/reconciler/base/reconciler.go:192 +0x424                                                                                                           │
│ github.com/solo-io/skv2/pkg/reconcile.(*runnerReconciler).Reconcile(0xc000916d80, {0x4174310, 0xc000e916b0}, {{{0xc000c9d336, 0x7}, {0xc000c9d340, 0xf}}})                                                 │
│     /home/runner/go/pkg/mod/github.com/solo-io/skv2@v0.36.3/pkg/reconcile/runner.go:206 +0xd52                                                                                                             │
│ sigs.k8s.io/controller-runtime/pkg/internal/controller.(*Controller).Reconcile(0xc000a52820, {0x4174310, 0xc000e916b0}, {{{0xc000c9d336, 0x7}, {0xc000c9d340, 0xf}}})                                      │
│     /home/runner/go/pkg/mod/sigs.k8s.io/controller-runtime@v0.16.3/pkg/internal/controller/controller.go:119 +0x1be                                                                                        │
│ sigs.k8s.io/controller-runtime/pkg/internal/controller.(*Controller).reconcileHandler(0xc000a52820, {0x4174310, 0xc000e916b0}, {0x3acdd00, 0xc000a93500})                                                  │
│     /home/runner/go/pkg/mod/sigs.k8s.io/controller-runtime@v0.16.3/pkg/internal/controller/controller.go:316 +0x4b9                                                                                        │
│ sigs.k8s.io/controller-runtime/pkg/internal/controller.(*Controller).processNextWorkItem(0xc000a52820, {0x4174348, 0xc0005ab590})                                                                          │
│     /home/runner/go/pkg/mod/sigs.k8s.io/controller-runtime@v0.16.3/pkg/internal/controller/controller.go:266 +0x3fc                                                                                        │
│ sigs.k8s.io/controller-runtime/pkg/internal/controller.(*Controller).Start.func2.2()                                                                                                                       │
│     /home/runner/go/pkg/mod/sigs.k8s.io/controller-runtime@v0.16.3/pkg/internal/controller/controller.go:227 +0xdf                                                                                         │
│ created by sigs.k8s.io/controller-runtime/pkg/internal/controller.(*Controller).Start.func2 in goroutine 129                                                                                               │
│     /home/runner/go/pkg/mod/sigs.k8s.io/controller-runtime@v0.16.3/pkg/internal/controller/controller.go:223 +0x98a
```

When we delete the `APIDoc`, the Gloo Portal controller will return to a normal state.

```
kubectl delete -f apidocs/oas-3_0_1-only-path-wrong-indentation-apidoc.yaml
```