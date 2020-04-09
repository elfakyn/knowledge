Basically every terraform `for_each` example out there has the following structure:

```
variable "rg_locations" {
  default = {
    a_group = "eastus"
    another_group = "westus2"
  }
     
resource "azurerm_resource_group" "rg" {
  for_each = var.rg_locations
  name     = each.key
  location = each.value
}
```

But they never tell you what to do if you want to access more than one value. It's easy:

```
variable "rg_locations" {
  default = {
    a_group = {
      location = "eastus"
      description = "our main location"
      tags = "primary"
    }
    another_group = {
      location = "westus2"
      description = "our fallback location"
      tags = "backup"
    }
  }
}
     
resource "azurerm_resource_group" "rg" {
  for_each    = var.rg_locations
  name        = each.key
  location    = each.value["location"]
  description = each.value["description"]
  tags        = each.value["tags"]
}
```

# FAQ

## Why a map and not a list?

for_each doesn't accept a list, only maps and sets.

## Can you offload multiple variables without having to say `location = ... "location"` for each of them?

You can do that if it's a [dynamic nested block](https://www.hashicorp.com/blog/hashicorp-terraform-0-12-preview-for-and-for-each/) but couldn't find anything for the toplevel resource block. Submit a ticket or PR if you do.
