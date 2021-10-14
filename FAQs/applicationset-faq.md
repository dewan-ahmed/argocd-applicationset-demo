### FAQ on ApplicationSet

1. What's the difference between app of apps pattern and ApplicationSet?

--> These are two different ways to solve the same problem; ApplicationSets is less work up front. ApplicationSets would allow for dynamic updates (for example, respond to new clusters added within Argo CD), and potentially less duplication (DRY). A common pattern with app of apps is to build a script file (or tool) that just regenerates the Application CRs in the Git repository (from a template) whenever you need to refresh them. You can think of ApplicationSet as being a similar idea, but built as a Kubernetes controller, and thus being a bit more declarative/dynamic. But it would definitely be less flexible than a fully custom solution (but also less initial work versus a custom solution). So apps of apps pattern is fine if it fits your use case. But if you prefer a more declarative approach, less initial work and fewer CRs to manage; try ApplicationSet.

2. How to migrate Argo CD applications to ApplicationSet?

--> Build an ApplicationSet CR using the application as a [template](https://argocd-applicationset.readthedocs.io/en/stable/Template/).


3. Does ApplicationSet work with Argo CD Image Updater?

--> No. Image Updater only works with applications that are built using Kustomize or Helm tooling. ApplicationSet creates ArgoCD applications from template so it doesn’t fit the requirement.

4. Does ApplicationSet support zero downtime?

--> As best as it can, but it's up to the user to ensure they are handling this in their Kubernetes deployments.

5. With regards to Argo CD ApplicationSets,  how would we ensure there’s testing performed before pushing to prod if a config has multiple clusters configured?

--> Users would probably want to test their application from a CI service before doing a full push, but for Argo CD/ApplicationSets you can create ApplicationSets that target clusters based on how they are categorized. For example, if you have 3 clusters in Argo CD labelled with `staging`, and 2 with `production`, ApplicationSet controller would target only the ones you specify in the Applicationset CR (and automatically detect when new clusters are added with this label).

6. Can you push changes to the ApplicationSet yaml to Git and have that auto update i.e dogfood for GitOps?

--> You can! You would just need to setup an Argo CD Application CR to point to the Git repository containing your ApplicationSet CR.

7. Does using the cluster operator require a central Argo instance or does Argo need to be on each cluster?

--> Argo CD itself works in both scenarios: you can have one Argo CD instance manage a bunch of clusters, or you can have one Argo CD per cluster (or even one Argo CD per namespace :zany_face: ).
No matter which route you go, the ApplicationSet controller would need to be installed alongside it (so on the central Argo CD instance, or on the per cluster Argo CD instances).


8. When do we use different generators like the *Matrix Generator*?

--> In this demo, the *list-generator* example deploys from a single repository to multiple clusters and the *git-generator-directory* example deploys from a monorepo to a single cluster. If you were to deploy from a monorepo to multiple clusters, you can use *Matrix Generator* and use both *list-generator* and *git-generator-directory* together. This is just an example as you can mix and match other generators together. 

9. What happens when an ApplicationSet is deleted?

--> When an ApplicationSet is deleted, the following occurs (in rough order):

- The ApplicationSet resource itself is deleted
- Any Application resources that were created from this ApplicationSet (as identified by owner reference)
- Any deployed resources (Deployments, Services, ConfigMaps, etc) on the managed cluster, that were created from that Application resource (by Argo CD), will be deleted.
    - Argo CD is responsible for handling this deletion, via the deletion finalizer.
    - To preserve deployed resources, set .syncPolicy.preserveResourcesOnDeletion to true in the ApplicationSet.

Thus the lifecycle of the ApplicationSet, the Application, and the Application's resources, are equivalent.

Read more [here](https://argocd-applicationset.readthedocs.io/en/stable/Application-Deletion/).

10. Can we create custom generators?

--> You can either contribute generators via PRs (which is probably the easiest way, but you need to ensure your generator would be beneficial to the community at large), or you can use the [Cluster Decision Resource Generator](https://argocd-applicationset.readthedocs.io/en/stable/Generators-Cluster-Decision-Resource/) if you want to write a generator that targets clusters using a custom CR (which you would write a controller to manage).