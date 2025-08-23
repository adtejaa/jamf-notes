## 🔷 What is HCL?

**HCL** stands for **HashiCorp Configuration Language**.

- It is the language used to write `.tf` files in Terraform.
- It is **declarative**, **human-readable**, and structured to define infrastructure resources clearly.
- Developed by HashiCorp, used in Terraform, Packer, Consul, and Vault.

---

## 🔷 Examples of HCL

```hcl
provider "aws" {
  region = "us-west-2"
}

resource "aws_s3_bucket" "example" {
  bucket = "my-example-bucket"
  acl    = "private"
}
```

---

## 🔷 Common Terms and Whether You're Using Them Correctly

| Term                 | Meaning                                                                 | ✅ You’re using it correctly? |
|----------------------|-------------------------------------------------------------------------|-------------------------------|
| **HCL**              | Configuration language for Terraform (`.tf` files)                      | ✅ Yes                        |
| **Schema**           | Go code that defines what inputs are valid for a resource               | ✅ Yes                        |
| **Terraform Provider** | A Go binary that implements CRUD for a Terraform resource             | ✅ Yes                        |
| **Go SDK**           | A Go package that wraps raw API calls, used inside providers            | ✅ Yes                        |
| **Plan / Apply**     | Terraform’s execution lifecycle commands                                | ✅ Yes                        |
| **gRPC**             | Protocol used between Terraform CLI and provider binaries               | ✅ Yes                        |

---

## 🔷 Where HCL Is Used

| File               | Purpose                        | Language |
|--------------------|--------------------------------|----------|
| `main.tf`          | Terraform resource definitions | **HCL**  |
| `variables.tf`     | Input variables                | **HCL**  |
| `outputs.tf`       | Outputs from module/resource   | **HCL**  |
| `provider.go`      | Registers provider and schema  | **Go**   |
| `resource_x.go`    | Implements CRUD for resource   | **Go**   |
| `client.go` (SDK)  | Sends HTTP requests to API     | **Go**   |

---

## 🔷 Why Schema and HCL Must Stay in Sync

| Concept           | Description |
|-------------------|-------------|
| Provider Schema   | Defines what fields are allowed in `.tf`, their types, and requirements |
| HCL `.tf` file    | Must only contain what the schema allows |
| Validation        | Terraform validates `.tf` input against the schema at plan/apply time |
| Errors if mismatch | Terraform will throw errors like "unsupported argument" or "missing required argument" |

---

## 🔷 Example of Correct Schema and Matching HCL

### Provider Schema (Go):

```go
Schema: map[string]*schema.Schema{
  "name": {
    Type:     schema.TypeString,
    Required: true,
  },
  "description": {
    Type:     schema.TypeString,
    Optional: true,
  },
}
```

### Matching HCL:

```hcl
resource "jamfpro_category" "example" {
  name        = "Security Tools"
  description = "Used by security team"
}
```

---

## ❌ What Happens if They Don't Match?

### Invalid HCL:

```hcl
resource "jamfpro_category" "example" {
  name        = 123                 # Wrong type
  invalid_key = "not allowed"       # Field not in schema
}
```

### Terraform Error:

```
Error: Unsupported argument
Error: Incorrect type for attribute
```

![terraform_hcl_schema_flow](https://github.com/user-attachments/assets/79870b37-cf51-437d-8716-6ffea5bef277)
