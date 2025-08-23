
## 🔷 1. What is a Terraform Provider Schema?

In a custom Terraform provider (written in Go), **each resource** (like `jamfpro_category`) defines a **schema**. This schema declares:

- Which fields a user can set in `.tf` (HCL)
- The data types (`string`, `bool`, `int`, etc.)
- Whether each field is `Required`, `Optional`, or `Computed`

### Example:

```go
func resourceCategory() *schema.Resource {
    return &schema.Resource{
        CreateContext: resourceCategoryCreate,
        ReadContext:   resourceCategoryRead,
        UpdateContext: resourceCategoryUpdate,
        DeleteContext: resourceCategoryDelete,
        Schema: map[string]*schema.Schema{
            "name": {
                Type:     schema.TypeString,
                Required: true,
            },
            "description": {
                Type:     schema.TypeString,
                Optional: true,
            },
        },
    }
}
```

---

## 🔷 2. What Happens During `terraform plan` or `terraform apply`

1. User writes `.tf`:
    ```hcl
    resource "jamfpro_category" "example" {
      name        = "Security Tools"
      description = "Used by security team"
    }
    ```

2. Terraform CLI reads the HCL and checks the fields.
3. Terraform starts the provider binary and sends a gRPC request:
    - Resource type: `jamfpro_category`
    - Fields: `name`, `description`
4. The provider **uses the schema** to validate:
    - ✅ Are required fields present?
    - ✅ Are types correct?
5. If valid, it runs the `CreateContext` function.

---

## 🔷 3. Inside `CreateContext`: Mapping to Go SDK

```go
func resourceCategoryCreate(ctx context.Context, d *schema.ResourceData, m interface{}) diag.Diagnostics {
    client := m.(*sdk.Client)

    category := sdk.Category{
        Name:        d.Get("name").(string),
        Description: d.Get("description").(string),
    }

    err := client.CreateCategory(category)
    if err != nil {
        return diag.FromErr(err)
    }

    d.SetId(category.Name)
    return nil
}
```

- `d.Get()` reads values from `.tf`
- The values are passed into your **Go SDK struct**
- SDK sends HTTP request to Jamf API

---

## 🔷 4. Go SDK Responsibilities (Low Level)

```go
func (c *Client) CreateCategory(cat Category) error {
    body, _ := json.Marshal(cat)

    req, _ := http.NewRequest("POST", c.BaseURL+"/JSSResource/categories/id/0", bytes.NewBuffer(body))
    req.Header.Set("Authorization", "Bearer "+c.AuthToken)
    req.Header.Set("Content-Type", "application/json")

    resp, err := c.HttpClient.Do(req)
    if err != nil {
        return err
    }
    defer resp.Body.Close()

    if resp.StatusCode != 201 {
        return fmt.Errorf("failed to create category")
    }

    return nil
}
```

---

## 🔷 5. Role of the Schema

| Schema Field | Role |
|--------------|------|
| `Required: true` | Must be provided in `.tf` |
| `Optional: true` | Can be omitted |
| `Computed: true` | Filled in by provider, not user |
| `d.Get("field")` | Accesses the field's value |
| `d.SetId()` | Sets the resource ID after creation |
| `d.Set()` | Writes back a computed value into `.tfstate` |

---

## 🔷 6. How All Pieces Connect (Full Flow)

```plaintext
User writes: .tf
     ↓
Terraform CLI parses HCL
     ↓
Calls provider binary via gRPC
     ↓
Provider validates input against schema
     ↓
Provider converts input to Go SDK struct
     ↓
Go SDK sends HTTP request to Jamf API
     ↓
Jamf returns success
     ↓
Provider updates .tfstate
```
> 🧠 Schema is the bridge between `.tf` and Go SDK. It ensures valid input, maps data, and manages lifecycle behavior.

