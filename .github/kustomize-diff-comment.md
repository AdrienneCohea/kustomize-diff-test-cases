<!-- kustomize-diff -->
# Kubernetes Manifests Changed

Click [here to review your manifest changes]({{JOB_URL}}). Each overlay that changed is shown in its own diff, which is contained in a collapsible section in the GitHub Actions logs linked above.

Note that if you changed `base` or `environments`, the same diff might be shown multiple times.

If the changes match what you expect, you can continue to peer review and merging.

## When Actual Changes Don't Match What You Expected

If expected changes are missing, or a change is affecting more places than you expected, consider the following possibilities:

- Your teammate merged other changes to these Kubernetes manifests, and you may need to rebase or update this branch.
- You made the change in `base` or `environments`, which triggered it to propagate.
- You need to change your approach and iterate locally using `kustomize build` until you achieve what you want.

## Useful Kustomization References

- [Declarative Management of Kubernetes Objects Using Kustomize](https://kubernetes.io/docs/tasks/manage-kubernetes-objects/kustomization/)
- [Kustomize Detailed Reference Pages](https://kubectl.docs.kubernetes.io/references/kustomize/kustomization)
