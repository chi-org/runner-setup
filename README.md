# Github self-hosted runner scale set setup for organization
#### 1. Summary

- Runners are in organization level (shared runners)
- Authentication by github app (auto install on current and future repos)
- 3 runner groups correspond to 3 runner sets (ubuntu-dind, robot-dind, ansible-dind), in this guidance we only create robot-dind runner set.
- Each runner set: minRunners: 1, maxRunners: 10

#### 2. Note

- Logs are kept by github action for each run, upload artifact for test result
- Logs and Artifact default retension day: 90(can be customized)

#### 3. Step by step guidance

1. K8S cluster required with helm installed

2. Install Action Runner Controller (arc) 

   - Ensure arc is installed before arc-runner-set 
   - This will create a controller pod in ns `arc-systems`

   ```bash
   helm install arc --namespace "arc-systems" --create-namespace oci://ghcr.io/actions/actions-runner-controller-charts/gha-runner-scale-set-controller
   ```

3. Install Action Runner Scale Set (arc-runner-set)

   - Create kube namespace for runner set
     ```bash
     kubectl create ns arc-runners
     ```

   - We use github app (we will have `github_app_id`, `github_app_installation_id`, `github_app_private_key` after the creation) here so that runner set can authenticate with the github api:

     - Go to **Organization Settings** > **Developer Settings** > **GitHub Apps** and create a new app "self-hosted-runner-authentication"

     - Homepage URL: `https://github.com/<org-name> `

     - Deselect **Webhook** > **Active**

     - Permissions:
       **Repo permissions:** Actions (Read-only), Administration (Read and write), Checks (Read-only), Metadata (Read-only)
       **Org permissions:** Self-hosted runners (Read and write)

       ![image-20250117111000611](./README.assets/image-20250117111000611.png)

     - Where can this GitHub App be installed?: `Any account` (we need to install on our repos in our org)
     - After successfully creating app, we will have `github_app_installation_id` is the **App ID** in **General** tab of our github app page
     - Click Generate a private key. The content of the private key is the `github_app_private_key`
     - Go to **Install App** tab and install the app on your organization (you must have access to the org you want to install app on) for all repos. After installation, you can get the `github_app_installation_id` in the URL (`https://github.com/organizations/<org-name>/settings/installations/<installation-id>`)
     - Create a kube secret for the github app authentication

     ```bash
     kubectl create secret generic github-app-secret --namespace=arc-runners --from-literal=github_app_id=<app-id> --from-literal=github_app_installation_id=<installation-id> --from-literal=github_app_private_key='-----BEGIN RSA PRIVATE KEY-----********-----END RSA PRIVATE KEY-----'
     ```

   - Build a custom docker image for runner based on the default runner image: Modify **custom-runner-image**/**Dockerfile** in this repo to customize your image, build and push it to your container registry

   - Create a runner group so that we can share our runners throughout the organization and manage repos access to our runner:

     - Go to **Org Settings** > **Actions** > **Runners group** and create a new runner group called `robot-dind` (if you change this name, please modify the runner group name in **actions-runner-scale-set/values.yaml** file)
     - Manage repo and workflow access to our runner and create.

   - Install arc-runner-set

     - Modify `<org-name>` and `<your-runner-image>` (ensure that you have right permission to pull this image) in **values.yaml** in **actions-runner-scale-set** folder of this repo and run:

     ```bash
     helm upgrade --install "arc-runner-set" --namespace "arc-runners" --create-namespace -f values.yaml oci://ghcr.io/actions/actions-runner-controller-charts/gha-runner-scale-set
     ```

     - After installation, if authentication is successfully, a listener pod will be created in ns arc-systems and runner pods created in arc-runners
       ![image-20250117134404992](./README.assets/image-20250117134404992.png)		![image-20250117134529782](./README.assets/image-20250117134529782.png)
     - Now the repos and workflows are permitted in our runner group can use our runners.
