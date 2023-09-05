# accelerator-profiles-testing
A repository to help with Accelerator Profiles

Ensure you are cluster-admin and have a working `oc` command.

## Ensure the Acc Profile CRD exists

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

## Creating the sample accelerator profiles

* Apply the YAML directly from the `main` branch of the github repo:

    ```bash
    GH_ROOT="https://raw.githubusercontent.com/rh-aiservices-bu/accelerator-profiles-testing/main/manifests"
    oc apply -f ${GH_ROOT}/acc_profiles.yaml
    ```

* If you need to delete them:

    ```bash
    oc -n redhat-ods-applications delete acceleratorprofile
    ```