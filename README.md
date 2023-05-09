# azure
variable "subscriptions" {
  type = list(string)
  default = [
    "SUBSCRIPTION_ID_1",
    "SUBSCRIPTION_ID_2",
    "SUBSCRIPTION_ID_3"
  ]
}

variable "locations" {
  type = list(string)
  default = [
    "LOCATION_1",
    "LOCATION_2",
    "LOCATION_3"
  ]
}

resource "azurerm_resource_group" "example_rg" {
  for_each = {
    for i in range(length(var.subscriptions)) : var.subscriptions[i] => var.locations[i]
  }

  name     = "example-rg-${each.key}-${each.value}"
  location = each.value
}

resource "azurerm_virtual_network" "example_vnet" {
  for_each = {
    for i in range(length(var.subscriptions)) : var.subscriptions[i] => var.locations[i]
  }

  name                = "example-vnet-${each.key}-${each.value}"
  address_space       = ["10.0.0.0/16"]
  location            = each.value
  resource_group_name = azurerm_resource_group.example_rg[each.key].name
}

resource "azurerm_subnet" "example_subnet" {
  for_each = {
    for i in range(length(var.subscriptions)) : var.subscriptions[i] => [
      for j in range(2) : "subnet-${j+1}-${var.locations[i]}"
    ]
  }

  name                 = each.value[0]
  resource_group_name  = azurerm_resource_group.example_rg[each.key].name
  virtual_network_name = azurerm_virtual_network.example_vnet[each.key].name
  address_prefixes     = ["10.0.${each.key}.${format("%02d", count.index+1)}/24"]
  count                = length(each.value)
}
