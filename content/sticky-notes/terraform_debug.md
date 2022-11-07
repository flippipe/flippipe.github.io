---
title: "Terraform debug"
date: 2022-05-31T12:55:15+01:00
summary: "How to debug Terraform variable"
description: "How to debug Terraform variable"
tags: ["terraform", "debug"]
draft: false
---

All credits go to [How to debug Terraform variable content using this custom module](https://nexxai.dev/how-to-debug-terraform-variable-content-using-this-custom-module/), this is just a note to copy & past to my projects



``` terraform
resource "null_resource" "terraform_debug" {
    provisioner "local-exec" {
        command = "echo $VARIABLE1 >> debug.txt"
        environment = {
            VARIABLE1 = jsonencode(var.example)
        }
    }
}
```

Remove from `state` to be re-generated again

``` shell
terraform state rm null_resource.terraform_debug
```

**Big gotcha...** just happen when do a ```terraform apply```
