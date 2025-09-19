# OpenShift Bootstrap

**A reusable GitOps framework for multi-cluster management.** Clone this repository to instantly deploy a complete OpenShift hub cluster that can provision and manage regional clusters at scale.

## Quick Reuse

```bash
# 1. Clone and bootstrap
git clone https://github.com/openshift-online/bootstrap.git
cd bootstrap
oc login https://api.your-hub-cluster.example.com:6443
oc apply -k operators/openshift-gitops/global
oc apply -k gitops-applications/

# 2. Add your first cluster
./bin/cluster-create

# 3. Done - GitOps handles the rest
```

## What You Get

This repository provides a **complete reusable infrastructure**:
- **Self-Managing Hub** - ArgoCD, ACM, Vault, Pipelines all configured automatically
- **Two-Phase Reuse** - GitHub for bootstrap, internal Gitea for cluster-specific configs  
- **Automatic Provisioning** - OpenShift (via Hive) and EKS (via CAPI) cluster creation
- **Zero-Config GitOps** - ApplicationSets with proper dependency ordering
- **Semantic Organization** - Intuitive directory structure for easy navigation

**Key benefit:** From zero to production-ready multi-cluster environment in minutes, not days.

## 🏗️ Architecture Overview

**Hub-Spoke Model**: One OpenShift hub cluster manages multiple regional clusters (OpenShift or EKS) using GitOps automation.

- **Hub Cluster**: Runs ArgoCD, ACM, and all cluster management operators
- **Managed Clusters**: Regional OpenShift (OCP) or EKS clusters provisioned and managed automatically
- **Automated Provisioning**: Single command creates complete cluster overlays with proper GitOps integration

## 📁 Directory Structure & Navigation

The repository uses **semantic directory organization** designed for intuitive navigation. Each directory level follows consistent patterns that clearly indicate purpose and scope.

### 🔍 Semantic Organization Patterns

**Top-level directories represent "things":**
- `clusters/` - Cluster provisioning configurations
- `operators/` - Application/operator deployments  
- `pipelines/` - Pipeline configurations
- `deployments/` - Service deployments
- `regions/` - Regional cluster specifications
- `bases/` - Reusable template components

**Nested patterns follow logical hierarchy:**
- **operators/{operator-name}/{deployment-target}/** - Service-first, then location
- **pipelines/{pipeline-name}/{cluster-name}/** - Pipeline type, then target cluster  
- **deployments/{service-name}/{cluster-name}/** - Service type, then deployment location
- **regions/{aws-region}/{cluster-name}/** - Geographic organization

**Deployment targets are consistent:**
- `global/` - Hub cluster deployments (shared infrastructure)
- `{cluster-name}/` - Managed cluster-specific deployments (e.g., `ocp-02/`, `eks-01/`)

The repository is designed for **intuitive navigation** with each directory level showing your next options:

### 🔧 Generation Input (where you start)
```bash
regions/                          # Available AWS regions
├── us-east-1/                   # Region-specific clusters
│   ├── ocp-02/              # Individual cluster specifications
│   │   └── region.yaml          # ← START HERE: cluster configuration
│   └── ocp-03/
└── us-west-2/
    └── eks-02/
```

### 🏭 Base Templates (shared components)
```bash
bases/
├── clusters/                    # Common cluster templates
├── pipelines/                   # Reusable Tekton pipelines
└── ocm/                        # OCM service templates
```

### 🎯 Generated Overlays (automated output)
```bash
clusters/                        # Cluster provisioning (auto-generated)
├── ocp-02/                 # OCP cluster (Hive resources)
├── ocp-03/                 # OCP cluster (Hive resources)  
└── eks-02/                 # EKS cluster (CAPI resources)

pipelines/                       # Pipeline deployments (auto-generated)
├── hello-world/
│   ├── ocp-02/             # Pipeline runs for ocp-02
│   └── ocp-03/             # Pipeline runs for ocp-03
└── cloud-infrastructure-provisioning/
    ├── ocp-02/
    └── ocp-03/

deployments/                     # Service deployments (auto-generated)
└── ocm/
    ├── ocp-02/             # OCM services for ocp-02
    └── ocp-03/             # OCM services for ocp-03

operators/                       # Operator deployments ({operator-name}/{deployment-target})
├── advanced-cluster-management/
│   └── global/                 # ACM hub cluster deployment
├── gitops-integration/
│   └── global/                 # GitOps integration policies
├── openshift-pipelines/
│   ├── global/                 # Pipelines hub cluster deployment
│   ├── ocp-02/             # Pipelines operator for ocp-02
│   ├── ocp-03/             # Pipelines operator for ocp-03
│   └── eks-02/             # Pipelines operator for eks-02
└── vault/
    └── global/                 # Vault secret management
```

### 🚀 GitOps Applications (orchestration)
```bash
gitops-applications/             # ArgoCD ApplicationSets
├── ocp-02.yaml            # ApplicationSet for ocp-02 (all components)
├── ocp-03.yaml            # ApplicationSet for ocp-03 (all components)
└── kustomization.yaml          # Main GitOps entry point
```

## 🧭 Navigation Pattern

**Each level shows your next options** - making discovery and management intuitive:

```bash
# Start with regions to see what's available
ls regions/                     # → us-east-1, us-west-2, eu-west-1

# Drill down to see clusters in a region  
ls regions/us-east-1/          # → ocp-02, ocp-03, ocp-04

# See what's deployed for any cluster
ls clusters/                   # → ocp-02, ocp-03, eks-02
ls pipelines/hello-world/      # → ocp-02, ocp-03, eks-02  
ls deployments/ocm/           # → ocp-02, ocp-03, eks-02
ls operators/openshift-pipelines/ # → global, ocp-02, ocp-03, eks-02

# Check GitOps applications
ls gitops-applications/       # → ocp-02.yaml, ocp-03.yaml, global/

# Explore operators by type
ls operators/                 # → advanced-cluster-management, gitops-integration, openshift-pipelines, vault
ls operators/vault/           # → global/
```

**🎯 Key Navigation Benefits:**
- **Consistent pattern**: Every directory level follows the same structure
- **Self-documenting**: Directory names clearly indicate their purpose  
- **Easy discovery**: `ls` at any level shows your available options
- **Logical grouping**: Related resources are co-located
- **Semantic organization**: Resource type first, then deployment target
- **Global vs Regional**: Clear separation between hub cluster (`global/`) and managed cluster (`cluster-XX/`) deployments

## 🚀 How Reuse Works

### Two-Phase Bootstrap Pattern

**Phase 1: Bootstrap from GitHub**
```bash
git clone https://github.com/openshift-online/bootstrap.git
oc apply -k operators/openshift-gitops/global
oc apply -k gitops-applications/
```

**Phase 2: Self-Referential Management**
After bootstrap, your cluster becomes self-managing:
- Internal Gitea service contains cluster-specific configurations
- ArgoCD switches to internal Git for ongoing management
- New clusters reference their own internal Git repository

### Adding Clusters (Simple)

```bash
./bin/cluster-create    # Interactive cluster specification
# GitOps automatically handles the rest
```

**The system automatically:**
- ✅ Creates cluster provisioning resources (OpenShift/EKS)
- ✅ Generates pipeline deployments  
- ✅ Configures operator installations
- ✅ Sets up service deployments
- ✅ Orders deployment with sync waves
- ✅ Integrates with ACM management

## 📖 Documentation

- **[REUSE.md](./REUSE.md)** - How to clone and reuse this repository
- **[BOOTSTRAP.md](./BOOTSTRAP.md)** - Step-by-step bootstrap walkthrough  
- **[docs/getting-started/QUICKSTART.md](./docs/getting-started/QUICKSTART.md)** - 5-minute overview
- **[docs/architecture/ARCHITECTURE.md](./docs/architecture/ARCHITECTURE.md)** - Visual architecture diagrams  

## 🏛️ Architecture & Components

### Hub Cluster Components
- **OpenShift GitOps (ArgoCD)**: Manages all cluster deployments via ApplicationSets
- **Red Hat Advanced Cluster Management (ACM)**: Multi-cluster lifecycle and governance
- **Cluster API (CAPI)**: EKS cluster provisioning with AWS infrastructure provider
- **Hive**: OpenShift cluster provisioning operator
- **OpenShift Pipelines (Tekton)**: CI/CD automation across all clusters

### Deployment Flow with Sync Waves
ApplicationSets deploy resources in ordered waves to ensure proper dependencies:

1. **Wave 1**: Cluster provisioning (Hive ClusterDeployment or CAPI resources)
2. **Wave 2**: Operator installation (OpenShift Pipelines operator)
3. **Wave 3**: Pipeline deployment (Tekton Pipeline and PipelineRun resources)
4. **Wave 4**: Service deployment (OCM database services and applications)

### GitOps Integration
- **Automated cluster registration**: ACM automatically registers managed clusters with ArgoCD
- **ApplicationSet pattern**: Single ApplicationSet generates all required applications per cluster
- **Resource exclusions**: ArgoCD excludes transient resources like TaskRuns but allows Pipeline/PipelineRun
- **Multi-platform support**: Seamlessly manages both OpenShift and EKS clusters


## 🔄 Workflow Diagram

```mermaid
sequenceDiagram
   participant Admin
   participant Generator as bin/cluster-generate
   participant Git as Git Repository
   participant Hub as Hub Cluster
   participant ArgoCD
   participant ACM
   participant Target as Managed Cluster
   
   Admin->>Generator: ./bin/cluster-generate regions/us-west-2/ocp-05/
   Generator->>Git: Create overlays + ApplicationSet
   
   Admin->>Hub: ./bin/bootstrap.sh
   Hub->>ArgoCD: Deploy ApplicationSet
   
   ArgoCD->>Git: Sync Wave 1 (cluster)
   Git->>ArgoCD: Cluster resources
   ArgoCD->>ACM: Deploy cluster (Hive/CAPI)
   ACM->>Target: Provision cluster
   
   Target->>ACM: Cluster ready
   ACM->>ArgoCD: Register cluster
   
   ArgoCD->>Git: Sync Wave 2-4 (operators→pipelines→services)
   Git->>ArgoCD: Application resources
   ArgoCD->>Target: Deploy applications
   
   Target->>Admin: Multi-cluster environment ready
```

## 🛠️ Development & Troubleshooting

- **Validation**: All overlays include `kustomize build` validation and dry-run checks
- **Monitoring**: Built-in status monitoring scripts for cluster provisioning
- **Rollback**: Clean rollback procedures for failed deployments
- **Extensibility**: Base template system allows easy addition of new services and pipelines

**For detailed installation and troubleshooting guidance, see [docs/getting-started/production-installation.md](./docs/getting-started/production-installation.md)**
