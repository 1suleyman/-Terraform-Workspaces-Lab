# 🧩 Terraform Workspaces Lab

In this lab, I learned how to **use Terraform workspaces to deploy the same infrastructure configuration into multiple isolated environments** (US, UK, India) using a **single codebase**. I also learned how workspaces interact with **state files**, **variables**, and **modules**.

---

## 📋 Lab Overview

**Goal:**

* Understand Terraform workspaces and the default workspace
* Create and switch between multiple workspaces
* Learn where workspace-specific state files are stored
* Use `terraform.workspace` to dynamically select values
* Deploy the same module into multiple environments using one configuration

**Learning Outcomes:**

* Identify the default Terraform workspace
* Create and list workspaces
* Switch between workspaces safely
* Understand workspace-specific state isolation
* Use `lookup()` with `terraform.workspace`
* Deploy identical stacks across regions using a single module

---

## 🛠 Step-by-Step Journey

### Step 1: Identify the Default Workspace

**Question:**
When a Terraform configuration is first created, which workspace exists by default?

✅ **Answer:** `default`

---

### Step 2: Inspect Existing Workspaces

Navigate to the configuration directory:

```bash
cd /root/terraform-projects/project-sapphire
```

List workspaces:

```bash
terraform workspace list
```

**Result:**

```
* default
```

* Only the **default workspace** exists initially

---

### Step 3: Create New Workspaces

**Task:** Create three workspaces:

* `us-payroll`
* `uk-payroll`
* `india-payroll`

**Commands:**

```bash
terraform workspace new us-payroll
terraform workspace new uk-payroll
terraform workspace new india-payroll
```

✔️ Three new isolated workspaces created

---

### Step 4: Switch to a Workspace

**Task:** Switch to the `us-payroll` workspace.

```bash
terraform workspace select us-payroll
```

✔️ Active workspace is now `us-payroll`

---

### Step 5: Understand Workspace State File Location

**Question:**
Where is the state file for the `india-payroll` workspace stored?

✅ **Answer:**

```
.terraform/terraform.tfstate.d/india-payroll/terraform.tfstate
```

**Key Concept:**

* Each workspace has its **own isolated state**
* Same code, **different state files**

---

### Step 6: Inspect Provided Configuration Files

Inside the configuration directory:

* `variables.tf`
* `provider.tf`

---

#### Variable Type: `region`

```hcl
type = map
```

---

#### Default Region Value (india-payroll)

```hcl
india-payroll = "ap-south-1"
```

---

#### Default AMI Value (india-payroll)

```hcl
india-payroll = "ami-05514xxxxxxxx"
```

---

### Step 7: Configure the Root Module to Use a Child Module

**Task:**
Update `main.tf` to call the child module located at:

```
/root/terraform-projects/modules/payroll-app
```

---

### Step 8: Use Workspace-Aware Lookups

**Module Configuration:**

```hcl
module "payroll_app" {
  source = "/root/terraform-projects/modules/payroll-app"

  app_region = lookup(var.region, terraform.workspace)
  ami        = lookup(var.ami, terraform.workspace)
}
```

**Explanation:**

* `terraform.workspace` returns the active workspace name
* `lookup()` fetches the correct region and AMI
* One module → multiple environments

---

### Step 9: Initialize the Configuration

```bash
terraform init
```

✔️ Module initialized successfully

---

### Step 10: Deploy Infrastructure Across All Workspaces

**Task:** Apply the same configuration to all three workspaces.

**Commands:**

```bash
terraform workspace select us-payroll
terraform apply

terraform workspace select uk-payroll
terraform apply

terraform workspace select india-payroll
terraform apply
```

* Confirm with `yes` for each apply

✔️ Infrastructure deployed independently in all three workspaces

---

## ✅ Key Commands Summary

| Task                      | Command                             |
| ------------------------- | ----------------------------------- |
| List workspaces           | `terraform workspace list`          |
| Create a workspace        | `terraform workspace new <name>`    |
| Switch workspace          | `terraform workspace select <name>` |
| Identify active workspace | `terraform.workspace`               |
| Deploy configuration      | `terraform apply`                   |

---

## 💡 Notes / Tips

* Workspaces **do not duplicate code** — only state
* Ideal for **environment isolation** (dev, test, prod)
* Avoid using workspaces for **completely different stacks**
* Use maps + `terraform.workspace` for clean environment logic
* One config → many environments → minimal duplication

---

## 📌 Lab Summary

| Step                         | Status | Key Takeaway                              |
| ---------------------------- | ------ | ----------------------------------------- |
| Default workspace identified | ✅      | Terraform starts with `default`           |
| Workspaces created           | ✅      | US, UK, India isolated environments       |
| Workspace switching          | ✅      | Active environment changes safely         |
| Workspace-aware variables    | ✅      | Used `lookup()` + `terraform.workspace`   |
| Multi-environment deployment | ✅      | One config deployed to three environments |

---

## ✅ References

* [https://developer.hashicorp.com/terraform/language/state/workspaces](https://developer.hashicorp.com/terraform/language/state/workspaces)
* [https://developer.hashicorp.com/terraform/language/expressions/references](https://developer.hashicorp.com/terraform/language/expressions/references)
* [https://developer.hashicorp.com/terraform/language/functions/lookup](https://developer.hashicorp.com/terraform/language/functions/lookup)
* [https://developer.hashicorp.com/terraform/cli/commands/workspace](https://developer.hashicorp.com/terraform/cli/commands/workspace)
