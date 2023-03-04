# CognICA product
This is the helm chart to run CognICA product in the k8s cluster with or without ArgoCD

_Currently, it is not the complete product can be deployed to k8s, but only the 'containerized' applications (services),
including following:_
* _asp_

## Deploy to local k8s
0. Ensure your `kubectl` configured to use local cluster by default
   ```shell
   kubectl config current-context
   ```
   _or using `kubectx`:_
   ```shell
   kubectx -c
   ```
   
1. Choose your local environment name (recommended is `cognica-local-YOUR-NAME`) and set it to variable
   ```shell
   export ICA_ENV_NAME=local-vogol
   ```
2. Create namespace for your local environment
   ```shell
   kubectl create ns "cognica-$ICA_ENV_NAME"
   ```
3. Fill `local-cluster-values.yaml` with valid credentials
4. Install `cognica` chart (run from the `apps` directory)
   ```shell
   helm install -n "cognica-$ICA_ENV_NAME" \
   -f cognica/values.yaml \
   -f cognica/local-cluster-values.yaml \
   --set envName=$ICA_ENV_NAME \
   cognica cognica
   ```

## Deploy to EKS
### ArgoCD
CognICA should be deployed to EKS via ArgoCD application. eparate ArgoCD application is created by GitLab CI per every branch named according with the template: `app/cognica/<env_name>` (see more [here](https://git.cognetivity.com/devops/kubernetes/non-production/-/tree/master/apps))

### Manual (not recommended)
0. Ensure your `kubectl` configured to use required EKS cluster by default
   ```shell
   kubectl config current-context
   ```
   _or using `kubectx`:_
   ```shell
   kubectx -c
   ```
1. Set required variables: 
   - `ICA_ENV_NAME` - name of the environment (e.g. `dev`, `qa`, etc)
   - `ICA_DOCKERREGISTRY_PASSWORD` - access token to `git.cognetivity.com` with **read_registry** Scope
   - `ICA_AWS_ACCOUNT_ID` - id of the AWS account EKS is running in (**non-production** in the code bellow)
   - `ASP_AML_ACCESS_KEY_ID`, `ASP_AML_SECRET_KEY` - credentials of the **asp-aml** IAM User from the Cognetivity's **root** AWS account (required because AML is deprecated and available only in the **root** account) 
   ```shell
   export ICA_ENV_NAME=dev-test
   export ICA_DOCKERREGISTRY_PASSWORD=4p5bL91Ln97c3_DH9NHh
   export ICA_AWS_ACCOUNT_ID=356131978015
   export ASP_AML_ACCESS_KEY_ID=AKIA2LTETHYLDGDRW66T
   export ASP_AML_SECRET_KEY=7SAHcqNGDb3334n5o+hD7mHVWf21uLaCGs5r87Fo
   ```
2. Create namespace for your environment
   ```shell
   kubectl create ns "cognica-$ICA_ENV_NAME"
   ```
3. Install `cognica` chart (run from the `apps` directory)
   ```shell
   helm install -n "cognica-$ICA_ENV_NAME" \
   --set envName="$ICA_ENV_NAME" \
   --set dockerRegistryCfg.cognetivity.password="$ICA_DOCKERREGISTRY_PASSWORD" \
   --set aws.accountId="$ICA_AWS_ACCOUNT_ID" \
   --set asp.aml.accessKeyId="$ASP_AML_ACCESS_KEY_ID" \
   --set asp.aml.secretKey="$ASP_AML_SECRET_KEY" \ 
   cognica cognica
   ```
