---
layout: default
title: Azure Kubernetes Services
permalink: /docs/integrations-guide/azure-kubernetes-services/
parent: Integrations Guide
nav_order: 11
---

# Azure Kubernetes Services

Deploy and manage containerized applications with Azure managed Kubernetes clusters via the Azure Management API.

**Category:** Asset Management

**Description:** List, create, update, and delete AKS managed clusters. Start and stop clusters, run kubectl commands directly against a cluster, and retrieve command results — all using Azure AD OAuth authentication.

---

## Table of Contents
- [Configuration](#configuration)
- [Required API Permissions](#required-api-permissions)
- [Setup Instructions](#setup-instructions)
- [Supported Actions](#supported-actions)
- [FAQ](#faq)
- [Troubleshooting](#troubleshooting)

---

## Configuration

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| **Tenant ID** | String | Yes | Azure AD Directory (tenant) ID |
| **Client ID** | String | Yes | Application (client) ID from Azure AD |
| **Client Secret** | Password | Yes | Client secret value from Azure AD |
| **Subscription ID** | String | Yes | Default Azure subscription ID (used as fallback when not provided per-call) |
| **Redirect URI** | String | Yes | Redirect URI registered in your Azure AD app (must match exactly) |
| **Authorization Code** | Password | Yes* | OAuth authorization code from Azure AD (used for initial setup only; replaced by refresh token after first use) |
| **API Version** | String | No | Azure Management API version (default: `2021-05-01`) |

\* Only required on first setup. Once the refresh token is obtained it is used automatically.

---

## Required API Permissions

Your Azure AD application must have **Azure Service Management** listed in API Permissions with the following delegated permission:

- `user_impersonation` — Allows the app to act on behalf of the signed-in user against the Azure Management API

**Permission Type:** Delegated

**OAuth Scope:** `https://management.azure.com/user_impersonation offline_access user.read`

---

## Setup Instructions

### 1. Register Application in Azure AD

Register your client application with Azure AD. See [Register your client application](https://docs.microsoft.com/en-us/azure/active-directory/develop/quickstart-register-app) for more information.

After registering, Azure AD provides you with:
- **Client ID** — Application (client) ID from the app's Overview page
- **Tenant ID** — Directory (tenant) ID from the app's Overview page
- **Redirect URI** — the URI you registered (default: `https://localhost/myapp`)
- **Client Secret** — generated under "Certificates & secrets" → "New client secret" (copy the value immediately)

### 2. Add API Permission

- In your registered app, navigate to **API permissions** → **Add a permission**
- Select **Azure Service Management**
- Under Delegated permissions, select **user_impersonation**
- Click "Add permissions"

### 3. Generate Authorization Code

Copy the following URL into a browser and replace `TENANT_ID`, `CLIENT_ID`, and `REDIRECT_URI` with your values:

```
https://login.microsoftonline.com/TENANT_ID/oauth2/v2.0/authorize?response_type=code&scope=https://management.azure.com/user_impersonation offline_access user.read&client_id=CLIENT_ID&redirect_uri=REDIRECT_URI
```

If you have not yet granted the required permissions, this link will prompt you to grant Delegated permissions for `user_impersonation` in the Azure Service Management API. If permissions were already granted, you will not be prompted.

After authenticating, you will be redirected to a URL with the following structure:

```
REDIRECT_URI?code=AUTH_CODE&session_state=SESSION_STATE
```

Copy the `AUTH_CODE` value. Authorization codes expire quickly (~10 minutes) so use it immediately.

### 4. Configure in Arambh AI

- Navigate to Integrations → Azure Kubernetes Services
- In the 'Configurations' tab, click on 'Add Configuration'
- Enter **Client ID**, **Client Secret**, **Tenant ID**, and **Subscription ID**
- Enter **Redirect URI** (must match exactly what was registered in Azure AD)
- Paste the **Authorization Code** copied in the previous step
- Click "Save" to save the configuration
- In the 'Actions' tab, enable the actions you want the platform to use

---

## Supported Actions

### 1. List Managed Clusters

List all managed Kubernetes clusters in an Azure subscription.

**Action:** `azure_kubernetes_services--get_managed_clusters_list`

#### Parameters

- `subscription_id` (required): Azure subscription ID to list clusters from
- `config_name` (optional): Configuration profile name

#### Returns

- Array of cluster objects with names, locations, provisioning states, and properties

---

### 2. Get Managed Cluster

Get details of a specific managed Kubernetes cluster.

**Action:** `azure_kubernetes_services--get_managed_cluster`

#### Parameters

- `subscription_id` (required): Azure subscription ID
- `resource_group_name` (required): Resource group containing the cluster
- `resource_name` (required): Name of the managed cluster
- `config_name` (optional): Configuration profile name

#### Returns

- Full cluster object including Kubernetes version, node pools, network profile, provisioning state, and FQDN

---

### 3. Create Managed Cluster

Create a new managed Kubernetes cluster in Azure Kubernetes Services.

**Action:** `azure_kubernetes_services--create_managed_cluster`

#### Parameters

- `subscription_id` (required): Azure subscription ID
- `resource_group_name` (required): Resource group in which to create the cluster
- `resource_name` (required): Name for the new managed cluster
- `location` (required): Azure region where the cluster will be created (e.g. `eastus`)
- `dns_prefix` (optional): DNS prefix for the cluster API server FQDN. Defaults to `resource_name`
- `node_count` (optional): Number of nodes in the default node pool (default: `1`)
- `vm_size` (optional): VM size for nodes (default: `Standard_DS2_v2`)
- `additional_fields` (optional): Override or extend any cluster properties (e.g. `networkProfile`, `tags`)
- `config_name` (optional): Configuration profile name

#### Example

```json
{
  "subscription_id": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "resource_group_name": "production-rg",
  "resource_name": "my-aks-cluster",
  "location": "eastus",
  "node_count": 3,
  "vm_size": "Standard_DS2_v2"
}
```

#### Returns

- Cluster object with provisioning state (`Creating` → `Succeeded`)

**Note:** Cluster creation is an asynchronous long-running operation. The response returns immediately with `provisioningState: Creating`. Use Get Managed Cluster to poll until the state reaches `Succeeded`.

---

### 4. Update Managed Cluster

Update an existing managed Kubernetes cluster.

**Action:** `azure_kubernetes_services--update_managed_cluster`

#### Parameters

- `subscription_id` (required): Azure subscription ID
- `resource_group_name` (required): Resource group containing the cluster
- `resource_name` (required): Name of the managed cluster to update
- `location` (required): Azure region of the cluster
- `tags` (optional): Comma-separated key=value tags to apply (e.g. `env=prod,team=infra`)
- `kubernetes_version` (optional): Kubernetes version to upgrade to (e.g. `1.29.0`)
- `node_count` (optional): New node count for the default agent pool
- `sku_tier` (optional): Cluster pricing tier — `Free`, `Standard`, or `Premium`
- `additional_fields` (optional): Any other cluster properties to update
- `config_name` (optional): Configuration profile name

#### Example

```json
{
  "subscription_id": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "resource_group_name": "production-rg",
  "resource_name": "my-aks-cluster",
  "location": "eastus",
  "tags": "env=prod,team=platform",
  "node_count": 5
}
```

#### Returns

- Updated cluster object with new provisioning state

---

### 5. Delete Managed Cluster

Delete a managed Kubernetes cluster from Azure Kubernetes Services.

**Action:** `azure_kubernetes_services--delete_managed_cluster`

#### Parameters

- `subscription_id` (required): Azure subscription ID
- `resource_group_name` (required): Resource group containing the cluster
- `resource_name` (required): Name of the managed cluster to delete
- `config_name` (optional): Configuration profile name

#### Returns

- Success confirmation message

---

### 6. Enable Cluster

Start (enable) a stopped managed Kubernetes cluster.

**Action:** `azure_kubernetes_services--enable_cluster`

#### Parameters

- `subscription_id` (required): Azure subscription ID
- `resource_group_name` (required): Resource group containing the cluster
- `resource_name` (required): Name of the managed cluster to start
- `config_name` (optional): Configuration profile name

#### Returns

- Success confirmation that the start operation was initiated

**Note:** Starting a cluster is asynchronous. The cluster will transition through `Starting` before reaching `Running`. Use Get Managed Cluster to check the current state.

---

### 7. Disable Cluster

Stop (disable) a running managed Kubernetes cluster.

**Action:** `azure_kubernetes_services--disable_cluster`

#### Parameters

- `subscription_id` (required): Azure subscription ID
- `resource_group_name` (required): Resource group containing the cluster
- `resource_name` (required): Name of the managed cluster to stop
- `config_name` (optional): Configuration profile name

#### Returns

- Success confirmation that the stop operation was initiated

**Note:** Stopping a cluster deallocates its nodes and reduces costs while preserving all cluster configuration and workloads. The cluster cannot serve traffic while stopped.

---

### 8. Run Command

Run a kubectl command directly against a managed Kubernetes cluster.

**Action:** `azure_kubernetes_services--run_command`

#### Parameters

- `subscription_id` (required): Azure subscription ID
- `resource_group_name` (required): Resource group containing the cluster
- `resource_name` (required): Name of the managed cluster
- `command` (required): The kubectl command to run (e.g. `kubectl get nodes`)
- `additional_fields` (optional): Additional fields for the run command request
- `config_name` (optional): Configuration profile name

#### Example

```json
{
  "subscription_id": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "resource_group_name": "production-rg",
  "resource_name": "my-aks-cluster",
  "command": "kubectl get pods --all-namespaces"
}
```

#### Returns

- Command result including exit code and stdout/stderr logs

**Note:** The action automatically polls every 15 seconds until the command completes and returns the final result. The cluster must be in a `Running` state and have sufficient node capacity to schedule the command runner pod.

---

### 9. Get Command Details

Retrieve the result of a previously submitted run_command operation.

**Action:** `azure_kubernetes_services--get_command_details`

#### Parameters

- `subscription_id` (required): Azure subscription ID
- `resource_group_name` (required): Resource group containing the cluster
- `resource_name` (required): Name of the managed cluster
- `command_id` (required): The command ID returned by the run_command operation
- `config_name` (optional): Configuration profile name

#### Returns

- Command result with provisioning state, exit code, and output logs

---

## FAQ

### Do I need admin consent for this integration?

No admin consent is required for Azure Resource Manager permissions. You only need to assign the appropriate IAM role (e.g. Contributor) to the registered application on the target subscription or resource group.

### How often are tokens refreshed?

Access tokens are automatically refreshed every 55 minutes using the stored refresh token. You only need to provide the authorization code once during initial setup.

### What happens if my authorization code expires?

Authorization codes expire within ~10 minutes. Once the initial setup is complete and a refresh token is obtained, you don't need the authorization code again. If you need to re-authorize (e.g. after the refresh token is revoked), generate a new authorization code using the OAuth URL in the setup instructions.

### Can I use multiple configurations?

Yes, use the `config_name` parameter to maintain separate configurations (e.g. `"production"`, `"staging"`). Each configuration stores its own credentials and refresh token independently.

### Why does run_command fail with "Unschedulable"?

Azure runs kubectl commands by scheduling a runner pod on the cluster. If all nodes are fully utilized (no spare CPU/memory capacity), the pod cannot be scheduled and the command fails. Ensure the cluster has at least one node with available resources before running commands.

### Can I run commands on a stopped cluster?

No. The cluster must be in `Running` state to accept kubectl commands. Use the Enable Cluster action first, then run your command once the cluster is running.

### What is the difference between node count in create vs update?

In `create_managed_cluster`, `node_count` sets the initial size of the default agent pool named `agentpool`. In `update_managed_cluster`, `node_count` updates the count of that same `agentpool`. If your cluster uses a differently named pool, use `additional_fields` to target it explicitly.

---

## Troubleshooting

### "AuthorizationFailed" error

**Solution:**
1. Verify the application has a role assignment on the subscription or resource group
2. Check that the assigned role includes the required `Microsoft.ContainerService` actions
3. Wait 1-2 minutes after assigning the role for propagation
4. Ensure the correct subscription ID is used

### "Invalid client secret" or "AADSTS7000215" error

**Solution:**
1. Verify the client secret hasn't expired (Azure Portal → App registrations → Certificates & secrets)
2. Ensure you copied the secret **value** (not the Secret ID)
3. Regenerate the secret if expired and update the configuration in Arambh AI
4. Check that Client ID and Tenant ID are correct with no extra spaces

### "Cluster not found" error

**Solution:**
1. Verify `resource_name` matches the cluster name exactly (case-sensitive)
2. Ensure `resource_group_name` is correct
3. Confirm the cluster exists in the Azure Portal
4. Check that `subscription_id` matches the subscription containing the cluster

### Run command fails with "PowerState=Stopped"

**Solution:**
1. The cluster is stopped — use the Enable Cluster action to start it first
2. Wait for the cluster to reach `Running` state before retrying the command

### Run command fails with "Unschedulable - Insufficient cpu/memory"

**Solution:**
1. The cluster nodes are fully utilized — scale up the node count using Update Managed Cluster
2. Or free up capacity by removing unused workloads from the cluster
3. Then retry the run_command operation

### Authorization code errors

**Solution:**
1. Ensure the authorization code is copied completely from the redirect URL
2. Use the code immediately — it expires within ~10 minutes
3. Verify the redirect URI in the request exactly matches what is registered in Azure AD
4. Generate a new authorization code if it has expired

### Cluster creation stuck in "Creating" state

**Solution:**
1. Large clusters can take 5-15 minutes to provision — check again after waiting
2. Check the Azure Portal for any provisioning errors under the cluster's Activity Log
3. Verify the subscription has sufficient quota for the requested VM size and node count
4. Check that the resource group exists in the specified location

---

**Related Integrations:**
- [Microsoft Azure Active Directory](microsoft-azure-ad) - User and identity management

**Back to:** [Integrations Overview](../)

**Last Updated:** June 2026
