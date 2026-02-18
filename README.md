# 📦 Making Waves

This is a visualization of a systolic array $NxN$ that multiplies two $NxN$ square matrices in $3N-2$ steps. Details are described in [Making Waves].

## 🌟 Highlights

- Uses a indexed Job to implement systolic array processing elements 
- A systolic orchestrator implemented as a Deployment communicates with Job pods and visualizes the progression of the matrix calculation 

## ℹ️ Overview

Please refer to [Making Waves] for the specific blog and the relevant repo(s). If you don't have time TL;DR:

This was an attempt to learn more about how to use an indexed Job in place of a Kubernetes stateful set. Using the Job indices it is possible to calculate the specific pod that implements the systolic array processing element $PE_{ij}$. PEs communicate with each other using a FastAPI and an external deployment called systolic orchestrator feeds input matrices to the systolic array, provides clock signal to emulate the systolic movement, it collects the results from the PEs and it visualizes them on a UI. 

### ✍️ Authors

All repos shared in [CurioCopia] are shared under Creative Commons license for others to adopt and use it as they wish.

## 🚀 Usage

1. Generate your own Docker image for [systolic-pe], place it in your favorite registry.
2. Generate your own Docker image for [systolic-orchestrator], place it in your favorite registry.
3. Clone this repo, adjust the essential parameters in [demo], run kustomize to generate the resources.
4. Access the UI via browser and enjoy.

## ⬇️ Installation

Let's follow the typical Kustomize installation process.

Define a place to work:
```bash
TEST_HOME=$(mktemp -d)
```
### Establish the Base

```bash
BASE=$TEST_HOME/base
mkdir -p $BASE

CONTENT="https://raw.githubusercontent.com/Curiocopia/blog-making-waves/refs/heads/main"

curl -s -o "$BASE/#1" "$CONTENT/base\
/{kustomization.yaml,making-waves.env,systolic-orchestrator-deployment.yaml,systolic-orchestrator-service.yaml,systolic-pe-job.yaml,systolic-pe-service.yaml}"
```
Look at the directory:
```bash
tree $TEST_HOME
```
Expect something like:
```bash
/tmp/tmp.rKw7Zf2Cti
└── base
    ├── kustomization.yaml
    ├── making-waves.env
    ├── systolic-orchestrator-deployment.yaml
    ├── systolic-orchestrator-service.yaml
    ├── systolic-pe-job.yaml
    └── systolic-pe-service.yaml
```
### The Base Customization

The base directory has a kustomization file:
```bash
more $BASE/kustomization.yaml
```
You can run kustomize on the base to emit customized resources to stdout and inspect:
```bash
kustomize build $BASE
```
If you are satisfied:
```bash
kubectl apply -k $BASE 
```
Once the resources are running, access the user interface via the NodePort that you selected, `http://<node-IP>:<nodeport>/ui`

## Create Overlay

Create a `demo` overlay.
```bash
OVERLAYS=$TEST_HOME/overlays
mkdir -p $OVERLAYS/demo
```
## Demo Customization

```bash
curl -s -o "$OVERLAYS/demo/#1" "$CONTENT/overlays/demo\
/{kustomization.yaml,making-waves-demo.env,systolic-orchestrator-service-patch.yaml,systolic-pe-job-patch.yaml}"
```
Adjust the parameters as you need. Set `namespace` for all resources and `systolic-pe` and `systolic-orchestrator` `image` in `kustomization.yaml`:
```yaml
namespace: demo

images:
- name: systolic-pe
  newName: my-registry/systolic-pe
  newTag: v1
- name: systolic-orchestrator
  newName: my-registry/systolic-orchestrator
  newTag: v1
```
Adjust `making-waves-demo.env` values for ConfigMap creation to use in various reources.

Adjust the values for the `spec.completions` and the `spec.parallelism` in the `systolic-pe-job-patch.yaml` to be $N^2$ where $N$ is set in `making-waves-demo.env` as `ARRAY_N`.

Add the value for the `spec.ports[0].nodePort` in the `systolic-orchestrator-service-patch.yaml`:
```yaml
spec:
  ports:
    - name: http
      port: 8080
      targetPort: 8080
      nodePort: 30100
```
Inspect the values. If you are satisfied, apply the kustomization after you create the `demo` namespace.
```bash
kubectl apply -k $OVERLAYS/demo 
```
Once the resources are running, access the user interface via the NodePort that you selected, `http://<node-IP>:<nodeport>/ui`

## 💭 Feedback and Contributing

If you have any other suggestions for improvements or corrections, please drop a note in Discussions.

[Making Waves]: https://curiocopia.com/blog/making-waves
[Curiocopia]: https://curiocopia.com
[systolic-pe]: https://github.com/Curiocopia/systolic-pe
[systolic-orchestrator]: https://github.com/Curiocopia/systolic-orchestrator
[demo]: overlays/demo/