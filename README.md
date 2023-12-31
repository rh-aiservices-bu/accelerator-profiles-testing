# Content for testing Accelerator Profiles features in RHODS/ODH

- [Content for testing Accelerator Profiles features in RHODS/ODH](#content-for-testing-accelerator-profiles-features-in-rhodsodh)
  - [Requirements](#requirements)
    - [not already having RHODS](#not-already-having-rhods)
    - [OC cli and Cluster-Admin](#oc-cli-and-cluster-admin)
    - [Install the live-build cut of RHODS in that cluster](#install-the-live-build-cut-of-rhods-in-that-cluster)
  - [Pick the right target namespace](#pick-the-right-target-namespace)
  - [Applying the fixes (temporary)](#applying-the-fixes-temporary)
    - [Create CRD:](#create-crd)
    - [this piece is likely needed for things to work for non-admin users](#this-piece-is-likely-needed-for-things-to-work-for-non-admin-users)
  - [Ensure the Accelerator Profile CRD exists](#ensure-the-accelerator-profile-crd-exists)
  - [Creating some sample accelerator profiles](#creating-some-sample-accelerator-profiles)
  - [Creating some dummy custom images with the profile annotations](#creating-some-dummy-custom-images-with-the-profile-annotations)
  - [Adding a couple NVIDIA examples too](#adding-a-couple-nvidia-examples-too)
  - [Adding an OpenVino runtime that prefers Intel FLEX GPUs](#adding-an-openvino-runtime-that-prefers-intel-flex-gpus)
  - [Confirming that Habana HPUs are truly being used](#confirming-that-habana-hpus-are-truly-being-used)
  - [notes for later:](#notes-for-later)
  - [uninstall live build](#uninstall-live-build)


This repository is a draft of the sort of content you'd need in order to test/experiment with Accelerator Profiles.

Notes:
* for now, the `identifiers` defined in the YAML files are probably not all accurate, and will need to be adjusted. If you know what the right values are, open an issue.
* this does not include the taints that you should apply to your nodes in order to finely target nodes.

## Requirements

### not already having RHODS

Ensure your cluster does not already have RHODS running in it. These instructions will setup a pre-release cut of RHODS and assume the cluster has neither RHODS nor ODH in it.

### OC cli and Cluster-Admin

Ensure you are cluster-admin and have a working `oc` command.

### Install the live-build cut of RHODS in that cluster

* The command below will kick off the RHODS live build install.
* Allow 5-10 minutes for it to complete

    ```bash
    GH_ROOT="https://raw.githubusercontent.com/rh-aiservices-bu/accelerator-profiles-testing/main/manifests"
    oc apply -f ${GH_ROOT}/rhods-1.33-livebuild.yaml
    ```

## Pick the right target namespace

The rest of these instructions will walk you through setting up extra resources in your cluster, and this has to be done in a specific namespace.

* You should set the `NS` environment variable to either `opendatahub` or `redhat-ods-applications`, depending on which cluster you are working in, and where RHODS is running.
* If you used the above-mentioned RHODS live build, it's likely to be `redhat-ods-applications`.
* If you're not sure, you can execute:

    ```bash
    if oc get namespace opendatahub >/dev/null 2>&1 && oc get namespace redhat-ods-applications >/dev/null 2>&1; then
        echo "Error: Both 'opendatahub' and 'redhat-ods-applications' namespaces exist."
    elif ! oc get namespace opendatahub >/dev/null 2>&1 && ! oc get namespace redhat-ods-applications >/dev/null 2>&1; then
        echo "Error: Neither 'opendatahub' nor 'redhat-ods-applications' namespaces exist."
    elif oc get namespace opendatahub >/dev/null 2>&1; then
        NS="opendatahub"
    elif oc get namespace redhat-ods-applications >/dev/null 2>&1; then
        NS="redhat-ods-applications"
    else
        NS=""
        echo "Error: not sure what happened"
    fi

    # Print the selected namespace
    echo "Selected namespace: $NS"
    ```

From this point on, make sure the `NS` environment variable has the right value.

## Applying the fixes (temporary)

### Create CRD:

* apply this to get the CRD

    ```bash
    oc apply -f https://raw.githubusercontent.com/opendatahub-io/odh-dashboard/f/accelerator-support/manifests/crd/acceleratorprofiles.opendatahub.io.crd.yaml
    ```

### this piece is likely needed for things to work for non-admin users

* so far, seems to be required still:
* Apply this:

    ```bash
    ## do not apply this. Confirm it works for a regular (non cluster-admin) user
    GH_ROOT="https://raw.githubusercontent.com/rh-aiservices-bu/accelerator-profiles-testing/main/manifests"
    oc -n ${NS} apply -f ${GH_ROOT}/fix_acc_profiles.yaml
    ```

## Ensure the Accelerator Profile CRD exists

* execute:

    ```bash
    oc get crd | grep -i acceleratorprofile
    ```

* should return

    ```bash
    bash-4.4 ~ $ oc get crd | grep -i acceleratorprofile
    acceleratorprofiles.dashboard.opendatahub.io                                       2023-08-25T18:44:23Z
    bash-4.4 ~ $
    ```

* if you don't see the line above, probably not worth continuing

## Creating some sample accelerator profiles

* Apply the YAML directly from the `main` branch of the github repo:

    ```bash
    GH_ROOT="https://raw.githubusercontent.com/rh-aiservices-bu/accelerator-profiles-testing/main/manifests"
    oc -n ${NS} apply -f ${GH_ROOT}/acc_profiles.yaml
    ```

* If you need to delete them:

    ```bash
    # oc -n ${NS} delete acceleratorprofile <actual_names>
    ```

## Creating some dummy custom images with the profile annotations

* Apply the YAML directly from the `main` branch of the github repo:

    ```bash
    GH_ROOT="https://raw.githubusercontent.com/rh-aiservices-bu/accelerator-profiles-testing/main/manifests"
    oc  -n ${NS}  apply -f ${GH_ROOT}/custom_workbench_images.yaml
    ```

## Adding a couple NVIDIA examples too

* Apply the YAML directly from the `main` branch of the github repo:

    ```bash
    GH_ROOT="https://raw.githubusercontent.com/rh-aiservices-bu/accelerator-profiles-testing/main/manifests"
    oc  -n ${NS}  apply -f ${GH_ROOT}/nvidia_examples.yaml
    ```

## Adding an OpenVino runtime that prefers Intel FLEX GPUs

* Apply the YAML directly from the `main` branch of the github repo:

    ```bash
    GH_ROOT="https://raw.githubusercontent.com/rh-aiservices-bu/accelerator-profiles-testing/main/manifests"
    oc  -n ${NS}  apply -f ${GH_ROOT}/serving_runtime.yaml
    ```

## Confirming that Habana HPUs are truly being used

* not sure here. maybe these repos can help
  * <https://github.com/HabanaAI/Gaudi2-Workshop>
  * <https://github.com/HabanaAI/Gaudi-tutorials>

## notes for later:

look into NVIDIA GPU simulator: <https://issues.redhat.com/browse/NVIDIA-50>

## uninstall live build

This will remove RHODS from the cluster.

draft instructions (probably slightly incomplete):

```bash
## acc profiles deletion
oc -n redhat-ods-applications delete acceleratorprofiles --all

# rhods uninstall
oc create configmap delete-self-managed-odh -n redhat-ods-operator
oc label configmap/delete-self-managed-odh api.openshift.com/addon-managed-odh-delete=true -n redhat-ods-operator
PROJECT_NAME=redhat-ods-applications
while oc get project $PROJECT_NAME &> /dev/null; do
  echo "$PROJECT_NAME still exists"
  sleep 1
done
echo "$PROJECT_NAME no longer exists"
oc delete namespace redhat-ods-operator

```