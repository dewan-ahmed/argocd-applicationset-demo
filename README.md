# argocd-applicationset-demo

### Purpose

This repository contains slides, Kubernetes manifest files and instructions for my talk *Deploy N Applications to N Clusters using Argo CD ApplicationSet*. This is not a stand-alone workshop repository. You will need to refer to the talk slides/video to make sense of the exercises provided below. The resource section contains suggested readings/resources on GitOps, Argo CD and ApplicationSet.

### Pre-requisites

- An understanding of Kubernetes, GitOps and Argo CD
- Access to at least two managed Kubernetes clusters (recent versions preferred). For ease of reference, we'll call the cluster where Argo CD is installed `cluster-manager` and the other cluster(s) as `managed-cluster`. 
- [kubectl](https://kubernetes.io/docs/tasks/tools/) & [argocd](https://argoproj.github.io/argo-cd/cli_installation/) CLI tools installed  (recent versions required)
- [kubectx](https://github.com/ahmetb/kubectx) for easy Kubernetes cluster context switching (optional)

### Argo CD and ApplicationSet installation

```
kubectl create namespace argocd

kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj-labs/applicationset/master/manifests/install.yaml
```
:construction: Notice that _ApplicationSet_ is installed from *master* rather than *v0.2.0*. This is because at the time of creating this lab, matrix-generator is not available on *v0.2.0*. 

### Accessing the Argo CD API Server

1. Change the argocd-server service type to _LoadBalancer_

```
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
```

2. Wait a few minutes for the loadbalancer to be available:

```
kubectl get svc -A | grep argocd-server
```

Right next to the private IP column, you will see a public IP for the `argocd-server` service.

3. From a web browser, navigate to that public IP. Use `admin` as the username and find the default password from the following command (Argo CD v1.9 and later):

```
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

4. Change the default admin password for Argo CD server from the CLI:

```
argocd login <public IP found in step2>
```

Use the same credentials from Step3.

Once succesfully logged in, use the following command to update the password:

```
argocd account update-password --current-password <default admin password> --new-password <set a password of your choice>
```
---

### Argo CD Demo

```
kubectl apply -f files/argocd-guestbook.yaml -n argocd
```

---

### ApplicationSet Demo

**List Generator example**:

Argo CD is able to create a namespace on the cluster it's installed on. However, it is unable to do so on the remote cluster. For this example, you have to create the namespace on the `managed-cluster` if it does not exist already.

```
# Change the Kubernetes context first
kubectx <context for the managed-cluster>

# Create the namespace on the managed-cluster
kubectl create namespace guestbook-ns

```

```
kubectl apply -f files/list-generator.yaml -n argocd
```

**Git Generator (directory) example**:

```
kubectl apply -f files/git-generator-directory.yaml -n argocd
```

:construction: guestbook-jsonnet application will take longer to sync and appear healthy. 


**Matrix Generator (list & git directory) example**:

```
kubectl apply -f files/matrix-generator.yaml -n argocd
```

:construction: Similar to list generator example, you might need to create all three namespaces in the `managed-cluster` prior to executing the above command (for the same reason).

---

### FAQ

1. What permissions does Argo CD need in the remote managed cluster?

--> `argocd cluster add <context>` command creates:
- serviceaccount named **argocd-manager** in kube-system
- clusterrole named **argocd-manager-role**
- clusterrolebinding named **argocd-manager-role-binding**


### Resources

- [The history of GitOps](https://www.weave.works/blog/the-history-of-gitops)
- [Continuous Lifecycle London 2018 Keynote on GitOps](https://www.slideshare.net/weaveworks/continuous-lifecycle-london-2018-event-keynote-97418556)
- [What is GitOps Really](https://www.weave.works/blog/what-is-gitops-really)
- [Alexis Richardson: GitOps - Git push all the things](https://www.youtube.com/watch?v=uWzgmmCzdF4)
- [Everything You Need To Become a GitOps Ninja - Alex C. & Alexander M.](https://www.youtube.com/watch?v=r50tRQjisxw)
- [GitOps in 1 slide](https://twitter.com/vitorsilva/status/999978906903080961/photo/1)
- [Argo CD - GitHub](https://github.com/argoproj/argo-cd)
- [Argo CD - ReadTheDocs](https://argo-cd.readthedocs.io/en/stable/)
- [ApplicationSet - GitHub](https://github.com/argoproj-labs/applicationset)
- [ApplicationSet - ReadTheDocs](https://argocd-applicationset.readthedocs.io/en/stable)
- [ApplicationSet Introductory Blog by Jonathan West](https://blog.argoproj.io/introducing-the-applicationset-controller-for-argo-cd-982e28b62dc5)
- [ApplicationSet Blog by Christian Hernandez](https://cloud.redhat.com/blog/getting-started-with-applicationsets)
