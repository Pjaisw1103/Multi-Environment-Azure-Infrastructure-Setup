<h1>📁 Multi-Environment Azure Infrastructure Setup</h1>

<p>
  <a href="https://azure.microsoft.com/"><img src="https://img.shields.io/badge/azure-%230072C6.svg?style=for-the-badge&logo=microsoftazure&logoColor=white" alt="Azure"></a>
  <a href="https://www.terraform.io/"><img src="https://img.shields.io/badge/terraform-%235835CC.svg?style=for-the-badge&logo=terraform&logoColor=white" alt="Terraform"></a>
  <a href="https://azure.microsoft.com/en-us/products/devops">
  <img src="https://img.shields.io/badge/Azure%20DevOps-0078D7?style=for-the-badge&logo=azuredevops&logoColor=white" alt="Azure DevOps"></a>
</p>

<p>An enterprise-grade, highly secure, scalable, and fully automated multi-environment (Dev, Staging, Prod) 3-Tier Web Application Infrastructure deployment on Microsoft Azure using Infrastructure as Code (IaC).</p>

<hr />

<h2>🏗️ System Architecture</h2>
<p>This repository provisions a secure, highly-available 3-tier architecture isolated inside private Virtual Networks (VNets), managed securely via Azure Bastion, and audited by Azure Monitor.</p>

<h3>🗺️ Architecture Diagram</h3>
<img src="./Screenshot%202026-06-11%20111246.png" alt="Enterprise Azure Architecture" width="100%" />

<hr />

<h2>📦 Resource Breakdown & Tier Explanations</h2>
<p>The infrastructure is broken down hierarchically to mimic real-world enterprise cloud governance:</p>

<h3>1. 🏢 Management & Scope Layers</h3>
<ul>
  <li><b>Management Group:</b> Enforces global enterprise policies and governance controls across subscriptions.</li>
  <li><b>Subscription:</b> Acts as a hard billing and access boundary (e.g., isolating Non-Prod from Prod workloads).</li>
  <li><b>Resource Group:</b> Logical container that lifecycle-manages resources bound to a specific application and environment.</li>
</ul>

<h3>2. 🌐 Networking Tier (VNet & Subnets)</h3>
<ul>
  <li><b>Virtual Network (VNet):</b> Configured with isolated address blocks dedicated per environment.</li>
  <li><b>Frontend Subnet:</b> Houses the Internet-facing entry points guarded by rigid Network Security Groups (NSGs).</li>
  <li><b>Backend Subnet:</b> Houses internal business-logic servers with no direct internet access.</li>
  <li><b>Azure Bastion Subnet:</b> Dedicated subnet used to stream secure RDP/SSH tunnel traffic over TLS directly into your backend subnet instances from the Azure Portal.</li>
</ul>

<h3>3. 💻 Application & Compute Tier</h3>
<ul>
  <li><b>Application Gateway:</b> Functions as an L7 load balancer, SSL offloader, and Web Application Firewall (WAF) routing user traffic into the Frontend subnets.</li>
  <li><b>Frontend Virtual Machines (VMs):</b> Standard compute scale-sets that render client applications.</li>
  <li><b>Internal Load Balancer:</b> Distributes traffic evenly from the Frontend VM pool down to the Backend VM layer.</li>
  <li><b>Backend Virtual Machines (VMs):</b> Hosts API processing services, totally isolated from public web components.</li>
</ul>

<h3>4. 🔑 Data & Secrets Tier</h3>
<ul>
  <li><b>Private Endpoint:</b> Restricts traffic traveling to the database layer over a private IP network interface instead of public endpoints.</li>
  <li><b>Azure SQL Database Server & DB:</b> Managed relational database tier holding transactional system data.</li>
  <li><b>Azure Key Vault:</b> Central cryptographic vault housing application configuration strings, TLS certificates, and database admin secrets.</li>
</ul>

<h3>5. 📊 Observability Tier</h3>
<ul>
  <li><b>Azure Monitor:</b> Tracks system performance metrics and infrastructure telemetry.</li>
  <li><b>Log Analytics Workspace:</b> Consolidates logs from Network Security Groups, Application Gateways, and Virtual Machines for diagnostic auditing and querying.</li>
</ul>

<hr />

<h2>🔄 End-to-End Traffic Flow</h2>

<pre>
[ User Browser ]
       │
       ▼ (HTTPS over Public Internet)
[ DNS (Azure Traffic Manager / Azure DNS) ]
       │
       ▼
[ Azure Application Gateway (L7 WAF) ]
       │
       ▼ (Routed securely into VNet)
[ Frontend Subnet / VMs ] ──► Protected by Frontend NSG
       │
       ▼
[ Internal Load Balancer ]
       │
       ▼
[ Backend Subnet / VMs ]  ──► Protected by Backend NSG
       │
       ▼ (Routed over Private Link)
[ Private Endpoint ]
       │
       ▼
[ Azure SQL Database Server ]
</pre>

<ul>
  <li><b>Management Control Plane:</b> Administrators authenticate through the <b>Azure Portal</b>, connect securely via the <b>Azure Bastion</b> service, and bridge straight into backend workloads securely without standard public IPs.</li>
  <li><b>Secret Management:</b> Application tiers pull certificates/keys on startup from <b>Azure Key Vault</b> using Managed Identities.</li>
</ul>

<hr />

<h2>📂 Repository Structure</h2>

<pre>
├── .github/
│   └── workflows/
│       ├── terraform-validation.yml   # Linter, Security Scanning, and Plan generation
│       └── terraform-deployment.yml   # Multi-environment continuous deployment pipeline
├── modules/                           # Reusable Infrastructure Blocks
│   ├── networking/                    # VNet, Subnets, NSGs, and Private Link endpoints
│   ├── compute/                       # Application Gateway, Internal Load Balancer, and VMs
│   ├── database/                      # Azure SQL Database Server setup
│   └── security/                      # Key Vault, Access Policies, Log Analytics
├── environments/                      # Environment-specific configurations
│   ├── dev/
│   │   ├── main.tf                    # Dev root main.tf calling standard modules
│   │   └── dev.tfvars                 # Small compute sizes (Standard_B1s)
│   ├── staging/
│   │   ├── main.tf                    # Staging configuration
│   │   └── staging.tfvars             # Medium compute sizes
│   └── prod/
│       ├── main.tf                    # High availability production orchestration
│       └── prod.tfvars                # Large configurations (Standard_DS2_v2), geo-redundancy
└── README.md
</pre>

<hr />

<h2>🚀 CI/CD GitOps Pipeline Blueprint</h2>
<p>The automation workflow uses a fully declarative GitHub Actions pipeline leveraging Terraform Workspaces or Environment states.</p>

<h3>🛠️ Workflow Pipeline Stages</h3>

<pre>
┌────────────────────────┐      ┌────────────────────────┐      ┌────────────────────────┐
│      GitHub Push       │ ───► │  Terraform Validate    │ ───► │   TF Security Scan     │
│   (Feature Branch)     │      │   (Check Syntax/Lint)  │      │  (tfsec / Checkov)     │
└────────────────────────┘      └────────────────────────┘      └────────────────────────┘
                                                                             │
                                                                             ▼
┌────────────────────────┐      ┌────────────────────────┐      ┌────────────────────────┐
│     PROD DEPLOYMENT    │ ◄─── │  Manual Approval Gate  │ ◄─── │   DEV/STG Deployment   │
│   (Post PR Merge)      │      │  (Environment Sign-off)│      │ (Auto-applied on Main) │
└────────────────────────┘      └────────────────────────┘      └────────────────────────┘
</pre>

<ol>
  <li><b>Validation Stage:</b> Executes <code>terraform fmt -check</code>, <code>terraform validate</code>, and runs static security checks (<code>tfsec</code>/<code>checkov</code>).</li>
  <li><b>Dev/Staging Continuous Deployment:</b> Runs on successful merge events to the primary <code>main</code> branch. Evaluates differences and updates infrastructure automatically.</li>
  <li><b>Production Gatekeeping:</b> Deploys staging validations into production environments only after authorized team reviewers acknowledge and clear the manual approval gates.</li>
</ol>

<hr />

<h2>🛠️ Getting Started</h2>

<h3>📌 Prerequisites</h3>
<ul>
  <li>An active Azure Account with Subscription Owner/Contributor access rights.</li>
  <li>Terraform CLI installed locally.</li>
  <li>Azure CLI tool configured.</li>
</ul>

<h3>💻 Local Initialization & Deployment</h3>

<pre><code># 1. Clone the repository
git clone https://github.com/Pjaisw1103/Multi-Environment-Azure-Infrastructure-Setup.git
cd Multi-Environment-Azure-Infrastructure-Setup

# 2. Login to your Azure Tenant
az login

# 3. Choose your target environment directory (e.g., Development)
cd environments/dev

# 4. Initialize Terraform modules and cloud backend providers
terraform init

# 5. Generate and preview resource planning changes
terraform plan -var-file="dev.tfvars"

# 6. Apply configuration state directly to Microsoft Azure
terraform apply -var-file="dev.tfvars" -auto-approve</code></pre>

<hr />

