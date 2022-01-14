# multicloud-kubernetes
Kubernetes multicloud deployment example using Terraform

# EKS

- Ensure AWS cli is installed and credentials configured
- Navigate to `eks` directory
- Run `terraform init` to initialize the directory and download the necessary providers
- Run `terraform plan` to review the provisioning plan
- Run `terraform apply` to provision the configured resources

# AKS

- Ensure Azure cli is installed
- Navigate to `aks` directory
- Login to Azure with `az login`
- Create active directory service account with `az ad sp create-for-rbac --skip-assignment`
- Rename `terraform.tfvars.example` to `terraform.tfvars` and replace appId and password using service account credentials
- Run `terraform init` to initialize the directory and download the necessary providers
- Run `terraform plan` to review the provisioning plan
- Run `terraform apply` to provision the configured resources

# Consul

- Navigate to the `consul` directory
- Comment out the `proxy_defaults.tf` file or remove the `tf` extension so that Terraform does not use this on the initial apply. We'll need to make sure that main.tf is applied first.
- Run `terraform init` to initialize the directory and download the necessary providers
- Run `terraform plan` to review the provisioning plan
- Run `terraform apply` to provision the configured resources
- Uncomment or add back the `tf` extension to the `proxy_defaults` file
- Run `terraform plan` to review the provisioning plan
- Run `terraform apply` to provision the configured proxy resources

# Configure kubectl

- Backup your existing kube config so you don't have to worry about removing clusters, users and contexts
  `cp ~/.kube/config ~/.kube/config_backup`
- Navigate to the `eks` directory
- Add the `demo-eks` context to your `~/.kube/config` file with the following command
  aws eks --region $(terraform output -raw region) update-kubeconfig --name $(terraform output -raw cluster_name) --alias demo-eks
- Navigate to the `aks` directory
- Add the `demo-aks` context to your `~/.kube/config` file with the following command
  az aks get-credentials --resource-group $(terraform output -raw resource_group_name) --name $(terraform output -raw kubernetes_cluster_name) --context demo-aks

# Validate cluster federation
- Verify EKS consul deployments
  `kubectl --context demo-eks get pods`
- Verify AKS consul deployments
  `kubectl --context demo-aks get pods`
- Verify proxy defaults
  `kubectl --context demo-eks get proxydefaults`
  `kubectl --context demo-aks get proxydefaults`
- Verify cluster federation of dc1 and dc2
  `kubectl exec statefulset/consul-server --context demo-aks -- consul members -wan`

# Deploy Application

- Navigate to the `counting-service` directory
- Run `terraform init` to initialize the directory and download the necessary providers
- Run `terraform plan` to review the provisioning plan
- Run `terraform apply` to provision the configured resources
- Verify that a `dashboard` pod exists in the `eks` cluster, this is the front end application
  `kubectl --context demo-eks get pods | grep dashboard`
- Verify that a `counting` pod exists in the `aks` cluster, this is the server application
  `kubectl --context demo-aks get pods | grep counting`
- Port forward the front end `dashboard` pod
  `kubectl --context demo-eks port-forward dashboard 9002:9002`
- Access the app at http://localhost:9002

# Cleanup application and resources

- For each of the following directories run the `terraform destroy` command in the specified order:
  - `counting-service`
  - `consul`
  - `aks`
  - `eks`

# Learning Resources
- https://learn.hashicorp.com/collections/terraform/kubernetes
- https://learn.hashicorp.com/tutorials/terraform/multicloud-kubernetes
