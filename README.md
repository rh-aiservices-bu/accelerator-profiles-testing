# Content for testing Accelerator Profiles features in RHODS/ODH

- [Content for testing Accelerator Profiles features in RHODS/ODH](#content-for-testing-accelerator-profiles-features-in-rhodsodh)
  - [Requirements](#requirements)
    - [OC cli and Cluster-Admin](#oc-cli-and-cluster-admin)
    - [Install the live-build cut of RHODS in that cluster](#install-the-live-build-cut-of-rhods-in-that-cluster)
    - [Ensure the Accelerator Profile CRD exists](#ensure-the-accelerator-profile-crd-exists)
    - [Ensure you know in which namespace you are supposed to deploy these extra YAMLs](#ensure-you-know-in-which-namespace-you-are-supposed-to-deploy-these-extra-yamls)
  - [Applying the fix (temporary)](#applying-the-fix-temporary)
  - [Creating the sample accelerator profiles](#creating-the-sample-accelerator-profiles)
  - [Creating some dummy custom images with the profile annotations](#creating-some-dummy-custom-images-with-the-profile-annotations)
  - [Adding a couple NVIDIA examples too](#adding-a-couple-nvidia-examples-too)
  - [Adding an OpenVino runtime that prefers Intel FLEX GPUs](#adding-an-openvino-runtime-that-prefers-intel-flex-gpus)
  - [Confirming that Habana HPUs are truly being used](#confirming-that-habana-hpus-are-truly-being-used)


This repository is a draft of the sort of content you'd need in order to test/experiment with Accelerator Profiles.

Notes:
* for now, the `identifiers` defined in the YAML files are probably not all accurate, and will need to be adjusted. If you know what the right values are, open an issue.
* this does not include the taints that you should apply to your nodes in order to finely target nodes.

## Requirements

### OC cli and Cluster-Admin

Ensure you are cluster-admin and have a working `oc` command.

### Install the live-build cut of RHODS in that cluster

```
placeholder for instructions
```

### Ensure the Accelerator Profile CRD exists

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

### Ensure you know in which namespace you are supposed to deploy these extra YAMLs

* You should set the `NS` parameter to either `opendatahub` or `redhat-ods-applications`, depending on which cluster you are working in.

* execute:

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

## Applying the fix (temporary)

**Note:** This fix is hopefully no longer required. Please do not apply it and confirm that it works without it.

* Apply the YAML directly from the `main` branch of the github repo:

    ```bash
    ## do not apply this. Confirm it works for a regular (non cluster-admin) user
    # GH_ROOT="https://raw.githubusercontent.com/rh-aiservices-bu/accelerator-profiles-testing/main/manifests"
    # oc -n ${NS} apply -f ${GH_ROOT}/fix_acc_profiles.yaml
    ```

## Creating the sample accelerator profiles

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

