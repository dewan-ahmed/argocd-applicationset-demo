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

kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj-labs/applicationset/v0.2.0/manifests/install.yaml
```

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

:construction: For the following examples, make sure that the namespace already exists on the cluster(s) prior to creating the Application/ApplicationSet custom resource(s).

### Argo CD Demo

```
kubectl apply -f files/argocd-guestbook.yaml -n argocd
```

---

### ApplicationSet Demo

**List Generator example**:

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

2. What happens when an ApplicationSet is deleted?

--> When an ApplicationSet is deleted, the following occurs (in rough order):

- The ApplicationSet resource itself is deleted
- Any Application resources that were created from this ApplicationSet (as identified by owner reference)
- Any deployed resources (Deployments, Services, ConfigMaps, etc) on the managed cluster, that were created from that Application resource (by Argo CD), will be deleted.
    - Argo CD is responsible for handling this deletion, via the deletion finalizer.
    - To preserve deployed resources, set .syncPolicy.preserveResourcesOnDeletion to true in the ApplicationSet.

Thus the lifecycle of the ApplicationSet, the Application, and the Application's resources, are equivalent.

Read more [here](https://argocd-applicationset.readthedocs.io/en/stable/Application-Deletion/).

### Resources

- [The history of GitOps](https://www.weave.works/blog/the-history-of-gitops)
- [Continuous Lifecycle London 2018 Keynote on GitOps](https://www.slideshare.net/weaveworks/continuous-lifecycle-london-2018-event-keynote-97418556)
- [What is GitOps Really](https://www.weave.works/blog/what-is-gitops-really)
- [Alexis Richardson: GitOps - Git push all the things](https://www.youtube.com/watch?v=uWzgmmCzdF4)
- [Everything You Need To Become a GitOps Ninja - Alex C. & Alexander M.](https://www.youtube.com/watch?v=r50tRQjisxw)
- [GitOps in 1 slide](https://twitter.com/vitorsilva/status/999978906903080961/photo/1)
- [Kubernetes anti-patterns: Let's do GitOps, not CIOps!](https://www.weave.works/blog/kubernetes-anti-patterns-let-s-do-gitops-not-ciops)
- [Argo CD - GitHub](https://github.com/argoproj/argo-cd)
- [Argo CD - ReadTheDocs](https://argo-cd.readthedocs.io/en/stable/)
- [Companies using Argo CD](https://github.com/argoproj/argo-cd/blob/master/USERS.md)
- [Some GitOps best practices using Argo CD](https://argo-cd.readthedocs.io/en/stable/user-guide/best_practices/)
- [ApplicationSet Original Proposal](https://docs.google.com/document/d/1juWGr20FQaJmuuTIS8mBFmWWDU422M_FQMuhp5c1jt4)
- [ApplicationSet - GitHub](https://github.com/argoproj-labs/applicationset)
- [ApplicationSet - ReadTheDocs](https://argocd-applicationset.readthedocs.io/en/stable)
- [ApplicationSet Introductory Blog by Jonathan West](https://blog.argoproj.io/introducing-the-applicationset-controller-for-argo-cd-982e28b62dc5)
- [ApplicationSet Blog by Christian Hernandez](https://cloud.redhat.com/blog/getting-started-with-applicationsets)
