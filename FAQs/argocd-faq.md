### FAQ on Argo CD

1. What is the difference between Infrastrucutre as Code and GitOps?

--> While Infrastructure as Code tools provide a way to manage infrastructure, they don’t manage the entire Cloud Native stack. GitOps on the other hand, allows you to manage the whole platform.

With GitOps you can use Infrastructure as Code tools to perform any infrastructure updates through Git.  GitOps extends the tradition of Infrastructure as Code tools. You can read more on this [here](https://queue.acm.org/detail.cfm?id=3237207).

“GitOps is to Infrastructure as Code as Microservices are to SOA or XP is to Agile.” -- Alexis Richardson, CEO of Weaveworks

2. What permissions does Argo CD need in the remote managed cluster?

--> `argocd cluster add <context>` command creates:
- serviceaccount named **argocd-manager** in kube-system
- clusterrole named **argocd-manager-role**
- clusterrolebinding named **argocd-manager-role-binding**

3. How is Flux CD similar or different from Argo CD?

--> [This](https://luktom.net/en/e1683-argocd-vs-flux) article does an excellent job at answeing the same question.

4. How is Tekton or Argo Workflow different from Argo CD?

--> Tekton and Argo Workflow provide ways to declare workflow pipelines for execution on Kubernetes. Tekton focuses on source based workflows, while Argo Workflow is more general purpose.

Argo CD is specifically built to address application delivery/deployment on Kubernetes following GitOps principles. That's why, it offers more sophistication and powerful features which cannot be expected from tools like Tekton or Argo Workflow. 

5. How to add private repository to Argo CD?

--> You can connect a private repository either using the [CLI](https://argo-cd.readthedocs.io/en/release-1.8/user-guide/commands/argocd_repo_add/), the web UI or by [creating a Kubernetes secret](https://argo-cd.readthedocs.io/en/stable/operator-manual/declarative-setup/#repositories). 

 How do you handle dev work when you're testing manifests in a namespace? Do you keep creating PRs or temporarily disable the sync?

--> You can disable *auto-sync* under the Sync Policy but there is no concept of disabling sync. For dev work, I'll assume that you're using your own feature branch and also deploying to a dev cluster. In that case, it shouldn't matter if you play around as you can specify your own feature branch for the git repo and you won't need to create PRs (i.e. the webhook event will react to every push on your feature branch).

6. When we have self-heal enabled, does ArgoCD prevent the resources from deletion or recreate them if they are deleted?

--> Argo CD recreates the resources, it doesn't prevent the deletion.

7. Can we measure the latency of the self-healing functionality for Argo CD?

--> If you mean the latency to perform self-heal, I think it will perform the action immediately. The latency to finish is just the same as a normal sync operation. Argo CD is purely Kubernetes event driven, so the timing to detect the drift is almost instantaneous. 

You might also want to check-out retry option that can be configured per Argo CD application. Refer to *syncPolicy* --> *retry* [here](https://argoproj.github.io/argo-cd/operator-manual/application.yaml). 

8. How many Argo CD instances do we need? Do we need one for each environment such as dev, prod etc?

--> This really depends IMHO on how the organization is structured. In a DevOps-mature organization, just one Argo CD instance for all the environments work. For organizations that are very silo'ed, it might be desirable to have a separate one for prod than from non-prod environments.

I do see people use two ArgoCDs for the good purpose, one for the cluster addon management like coredns, cluster autoscaler, the other one for the applications. I think either way should work, as long as you understand the problem.

Argo CD should be managed in a relative raw way without many dependencies. I recall that one community member shared his circular dependency problem recently (eg. Argo CD is deploying Ingress controller, but also using it configure its endpoint. So, when the Ingress controller is experiencing some issues. You cannot access ArgoCD to get the ingress controller fixed). That’s one of the (rare) cases I am thinking another Argo CD might be making sense.

9. Is there a concept of extensions in Argo CD?

--> Argo CD allows integrating additional config management tools beyond the ones Argo CD is shipped with using [config management plugins](https://argoproj.github.io/argo-cd/user-guide/config-management-plugins/). 

10. Does Argo CD integrate with Terraform?

--> While there is no official integration for Argo CD with Terraform, you can certainly use Terraform for your infrastructure deployment and Argo CD for the application deployment. These two tools can work together.

11. Does ArgoCD work with terraform for non-k8s workloads?

--> [Crossplane](https://crossplane.io/) would be a project for you to check-out. Most folks using Argo CD are probably using Argo CD to manage standard K8s resources (Deployments/ConfigMaps/etc), but I don't see why you couldn't also use Argo CD with Crossplane to manage external resources.

12. Any suggestions for automating the update of the container image version as part of a deployment pipeline?

--> Three options I can think of:

a. [Argo CD Image Updater](https://github.com/argoproj-labs/argocd-image-updater)
b. [kpt](https://github.com/GoogleContainerTools/kpt) to replace the image value. It can be used for other fields as well so you have to replace image tag plus environment variable that drives the tracing sidecar you might use. On the backend, you can see trace information based on the image version as well.
c. `sed` replacing a helm values file for your pipeline

13.  I’m adding Linkerd to my deployment managed with Argo CD, and using CM as a cluster CA for mTLS - but because I’m using sealed secrets, the trust anchor issuer isn’t ready for a bit, and the certificate request fails and does not get retried because the issuer is not in a ready state. What can I do?

--> Although not covered in this demo, please have your SealedSecret in one sync wave and Linkerd in a later wave.

14. How would one prevent a PVC from being deleted in order to preserve data and then be able to bring it back to the app? Every time I delete an application via the argocd-ui, it always deletes the PVC and I lose data.

--> You can choose 'Non cascading' delete option as a first try.

15. Do you know if Argo CD is capable to control K8s jobs? I mean can I use Argo CD as to trigger Jobs with certain set of env_variables as parameters?

--> Maybe you want to have a look at Argo Workflows for that. Argo CD is good for managing applications, and not so for one-off things like Jobs are. Argo Workflows is exactly for that - run any kind of jobs in a coordinated way.

Follow-up question: Workflows and rollouts seems very heavy for my usecase. There're only two jobs I want to run via Argo-CD, maybe I'm wrong but it seems easier to implement this case using ArgoCD then, move everything else to workflows\rollouts.

--> You can run jobs as hooks. This way, they will not affect sync status and if you have a deletion policy on them (e.g. delete before run), you can run them on each sync. 

16. Suppose I have YAML files for each of the namespace, secret, configmap, pvc, deployment, service, route. How does Argo CD know which YAML to apply first (Namespace, secret, CM, PVC ...etc)?

--> Argo CD has a built-in order as can be seen [here](https://github.com/argoproj/gitops-engine/blob/7495c633c378ca446d166bd5c9e2a46c7e40b476/pkg/sync/sync_tasks.go#L27). If you need more fine grained control, you have to use [sync waves]([ as described here](https://argo-cd.readthedocs.io/en/stable/user-guide/sync-waves/#how-do-i-configure-waves)).

17. Suppose I have a Git repo example-repo and under this I have 5 microservices directories `db-app` `notification` `events` etc. How can I instruct Argo to deploy the microservices in a specific order?

--> [Sync Waves](https://argo-cd.readthedocs.io/en/stable/user-guide/sync-waves/) + [Sync Hooks](https://argo-cd.readthedocs.io/en/stable/operator-manual/upgrading/1.3-1.4/#sync-hooks). For example, you'll have a presync hook that waits for your application to finish before the next resource starts. However, I'd recommend implementing your services so they are not order dependent, if at all possible.
