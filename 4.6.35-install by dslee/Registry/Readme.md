oc patch configs.imageregistry.operator.openshift.io cluster --type merge --patch '{"spec":{"managementState":"Managed"}}'
oc get pod -n openshift-image-registry
oc edit configs.imageregistry.operator.openshift.io