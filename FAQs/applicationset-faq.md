### FAQ on ApplicationSet

1. When do we use different generators like the *Matrix Generator*?

--> In this demo, the *list-generator* example deploys from a single repository to multiple clusters and the *git-generator-directory* example deploys from a monorepo to a single cluster. If you were to deploy from a monorepo to multiple clusters, you can use *Matrix Generator* and use both *list-generator* and *git-generator-directory* together. This is just an example as you can mix and match other generators together. 

2. What happens when an ApplicationSet is deleted?

--> When an ApplicationSet is deleted, the following occurs (in rough order):

- The ApplicationSet resource itself is deleted
- Any Application resources that were created from this ApplicationSet (as identified by owner reference)
- Any deployed resources (Deployments, Services, ConfigMaps, etc) on the managed cluster, that were created from that Application resource (by Argo CD), will be deleted.
    - Argo CD is responsible for handling this deletion, via the deletion finalizer.
    - To preserve deployed resources, set .syncPolicy.preserveResourcesOnDeletion to true in the ApplicationSet.

Thus the lifecycle of the ApplicationSet, the Application, and the Application's resources, are equivalent.

Read more [here](https://argocd-applicationset.readthedocs.io/en/stable/Application-Deletion/).
