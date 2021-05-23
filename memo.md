# terraform memo

[Get Started - AWS](https://learn.hashicorp.com/collections/terraform/aws-get-started)やったときのメモ

## まずはつかってみる

tfファイルを作ってawscliをインストール
設定ファイルを作成
```
MacBook-Pro-2:terraform vaivailx$ aws configure
AWS Access Key ID [None]: *************
AWS Secret Access Key [None]: ********************
Default region name [None]: ap-northeast-1
Default output format [None]: json
MacBook-Pro-2:getstarted vaivailx$ ls ~/.aws/
config          credentials
MacBook-Pro-2:getstarted vaivailx$ ll
-rw-r--r--  1 vaivailx  staff  172  6 28 19:11 example.tf
```

terraform init実行して初期化
```
MacBook-Pro-2:getstarted vaivailx$ terraform init

Initializing the backend...

Initializing provider plugins...
- Checking for available provider plugins...
- Downloading plugin for provider "aws" (terraform-providers/aws) 2.17.0...

The following providers do not have any version constraints in configuration,
so the latest version was installed.

To prevent automatic upgrades to new major versions that may contain breaking
changes, it is recommended to add version = "..." constraints to the
corresponding provider blocks in configuration, with the constraint strings
suggested below.

* provider.aws: version = "~> 2.17"

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```

.terraformができてる
```
MacBook-Pro-2:getstarted vaivailx$ ls ./.terraform/plugins/darwin_amd64/
lock.json                          terraform-provider-aws_v2.17.0_x4
```

applyするも失敗
```
MacBook-Pro-2:getstarted vaivailx$ terraform apply

...

Plan: 1 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

aws_instance.example: Creating...

Error: Error launching source instance: OptInRequired: You are not subscribed to this service. Please go to http://aws.amazon.com to subscribe.
        status code: 401, request id: 89c96d86-9a91-41c6-b78e-d14091ac8fe2

  on example.tf line 6, in resource "aws_instance" "example":
   6: resource "aws_instance" "example" {
```

401なので権限不足。EC2のAmazonEC2FullAccessというポリシーを追加して再度実行する。
あと、連絡先登録とか、登録後のSMS確認やらできていなかった。
で、再度実施。
```
MacBook-Pro-2:getstarted vaivailx$ terraform apply
...
Error: Error launching source instance: InvalidAMIID.NotFound: The image id '[ami-0c3fd0f5d33134a76]' does not exist
        status code: 400, request id: 8101b7d7-a9fb-4d59-8c66-9006598d3be1

  on example.tf line 6, in resource "aws_instance" "example":
   6: resource "aws_instance" "example" {
...
```

そんなAMI IDはないと言われる。
→ami idは東京リージョンのものをみて変えて、リージョンは変更していなかったからだ。
まずはEC2コンソールで使えることを確認してからやろうとしたので中途半端にAMI IDだけ変えていた。

そんなわけでリージョン修正して再実行。
```
provider "aws" {
  profile    = "default"
  region     = "ap-northeast-1"
}

resource "aws_instance" "example" {
  ami           = "ami-0c3fd0f5d33134a76"
  instance_type = "t2.micro"
}
```

```
MacBook-Pro-2:getstarted vaivailx$ terraform apply

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  ## aws_instance.example will be created
  + resource "aws_instance" "example" {
      + ami                          = "ami-0c3fd0f5d33134a76"
      + arn                          = (known after apply)
      + associate_public_ip_address  = (known after apply)
      + availability_zone            = (known after apply)
      + cpu_core_count               = (known after apply)
      + cpu_threads_per_core         = (known after apply)
      + get_password_data            = false
      + host_id                      = (known after apply)
      + id                           = (known after apply)
      + instance_state               = (known after apply)
      + instance_type                = "t2.micro"
      + ipv6_address_count           = (known after apply)
      + ipv6_addresses               = (known after apply)
      + key_name                     = (known after apply)
      + network_interface_id         = (known after apply)
      + password_data                = (known after apply)
      + placement_group              = (known after apply)
      + primary_network_interface_id = (known after apply)
      + private_dns                  = (known after apply)
      + private_ip                   = (known after apply)
      + public_dns                   = (known after apply)
      + public_ip                    = (known after apply)
      + security_groups              = (known after apply)
      + source_dest_check            = true
      + subnet_id                    = (known after apply)
      + tenancy                      = (known after apply)
      + volume_tags                  = (known after apply)
      + vpc_security_group_ids       = (known after apply)

      + ebs_block_device {
          + delete_on_termination = (known after apply)
          + device_name           = (known after apply)
          + encrypted             = (known after apply)
          + iops                  = (known after apply)
          + snapshot_id           = (known after apply)
          + volume_id             = (known after apply)
          + volume_size           = (known after apply)
          + volume_type           = (known after apply)
        }

      + ephemeral_block_device {
          + device_name  = (known after apply)
          + no_device    = (known after apply)
          + virtual_name = (known after apply)
        }

      + network_interface {
          + delete_on_termination = (known after apply)
          + device_index          = (known after apply)
          + network_interface_id  = (known after apply)
        }

      + root_block_device {
          + delete_on_termination = (known after apply)
          + iops                  = (known after apply)
          + volume_id             = (known after apply)
          + volume_size           = (known after apply)
          + volume_type           = (known after apply)
        }
    }

Plan: 1 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

aws_instance.example: Creating...
aws_instance.example: Still creating... [10s elapsed]
aws_instance.example: Still creating... [20s elapsed]
aws_instance.example: Still creating... [30s elapsed]
aws_instance.example: Still creating... [40s elapsed]
aws_instance.example: Creation complete after 46s [id=i-007f0f7cb518f4c75]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
MacBook-Pro-2:getstarted vaivailx$
```
成功っぽい。
コンソールも見てみる。
動いている！

CLIでも確認できる。
```
MacBook-Pro-2:getstarted vaivailx$ terraform show
## aws_instance.example:
resource "aws_instance" "example" {
    ami                          = "ami-0c3fd0f5d33134a76"
    arn                          = "arn:aws:ec2:ap-northeast-1:682816909333:instance/i-007f0f7cb518f4c75"
    associate_public_ip_address  = true
    availability_zone            = "ap-northeast-1a"
    cpu_core_count               = 1
    cpu_threads_per_core         = 1
    disable_api_termination      = false
    ebs_optimized                = false
    get_password_data            = false
    id                           = "i-007f0f7cb518f4c75"
    instance_state               = "running"
    instance_type                = "t2.micro"
    ipv6_address_count           = 0
    ipv6_addresses               = []
    monitoring                   = false
    primary_network_interface_id = "eni-0d1f9f36a74715a5a"
    private_dns                  = "ip-172-31-41-45.ap-northeast-1.compute.internal"
    private_ip                   = "172.31.41.45"
    public_dns                   = "ec2-3-112-5-170.ap-northeast-1.compute.amazonaws.com"
    public_ip                    = "3.112.5.170"
    security_groups              = [
        "default",
    ]
    source_dest_check            = true
    subnet_id                    = "subnet-c1d82c89"
    tenancy                      = "default"
    volume_tags                  = {}
    vpc_security_group_ids       = [
        "sg-15d1d56a",
    ]

    credit_specification {
        cpu_credits = "standard"
    }

    root_block_device {
        delete_on_termination = true
        iops                  = 100
        volume_id             = "vol-0929ce2c6f06ee574"
        volume_size           = 8
        volume_type           = "gp2"
    }
}
MacBook-Pro-2:getstarted vaivailx$
```

## 設定変更をする

https://learn.hashicorp.com/terraform/getting-started/change

設定変更とその反映をお試し

ami を以下のように変更した
```
//  ami           = "ami-0c3fd0f5d33134a76"
  ami           = "ami-1d37b57b"
```

terraform applyで適用

```
MacBook-Pro-2:getstarted vaivailx$ terraform apply
aws_instance.example: Refreshing state... [id=i-007f0f7cb518f4c75]

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
-/+ destroy and then create replacement

Terraform will perform the following actions:

  ## aws_instance.example must be replaced

...
Plan: 1 to add, 0 to change, 1 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

aws_instance.example: Destroying... [id=i-007f0f7cb518f4c75]
aws_instance.example: Still destroying... [id=i-007f0f7cb518f4c75, 10s elapsed]
aws_instance.example: Destruction complete after 11s
aws_instance.example: Creating...
aws_instance.example: Still creating... [10s elapsed]
aws_instance.example: Still creating... [20s elapsed]
aws_instance.example: Creation complete after 27s [id=i-06b903b99910de839]

Apply complete! Resources: 1 added, 0 changed, 1 destroyed.
MacBook-Pro-2:getstarted vaivailx$

ami を変えるので、
>  aws_instance.example must be replaced
とでている。
その上でyesとして実行すると、
> Apply complete! Resources: 1 added, 0 changed, 1 destroyed.
とされて、既存のは壊して、新しいの作ったよってなる。

ただEC2コンソールを見ると、既存のものは停止されて、新しいインスタンスが作られている。
消すわけではないようだ。

planだと消すらしい。ためしてみよう。

//  ami           = "ami-0c3fd0f5d33134a76"
  ami           = "ami-1d37b57b"

  を
  逆にする

  ami           = "ami-0c3fd0f5d33134a76"
  // ami           = "ami-1d37b57b"


  ```
  MacBook-Pro-2:getstarted vaivailx$ terraform plan
Refreshing Terraform state in-memory prior to plan...
The refreshed state will be used to calculate this plan, but will not be
persisted to local or remote state storage.

aws_instance.example: Refreshing state... [id=i-06b903b99910de839]

------------------------------------------------------------------------

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
-/+ destroy and then create replacement

Terraform will perform the following actions:

  ## aws_instance.example must be replaced
-/+ resource "aws_instance" "example" {
      ~ ami                          = "ami-1d37b57b" -> "ami-0c3fd0f5d33134a76" ## forces replacement
      ~ arn                          = "arn:aws:ec2:ap-northeast-1:682816909333:instance/i-06b903b99910de839" -> (known after apply)
      ~ associate_public_ip_address  = true -> (known after apply)
      ~ availability_zone            = "ap-northeast-1a" -> (known after apply)
      ~ cpu_core_count               = 1 -> (known after apply)
      ~ cpu_threads_per_core         = 1 -> (known after apply)
      - disable_api_termination      = false -> null
      - ebs_optimized                = false -> null
        get_password_data            = false
      + host_id                      = (known after apply)
      ~ id                           = "i-06b903b99910de839" -> (known after apply)
      ~ instance_state               = "running" -> (known after apply)
        instance_type                = "t2.micro"
      ~ ipv6_address_count           = 0 -> (known after apply)
      ~ ipv6_addresses               = [] -> (known after apply)
      + key_name                     = (known after apply)
      - monitoring                   = false -> null
      + network_interface_id         = (known after apply)
      + password_data                = (known after apply)
      + placement_group              = (known after apply)
      ~ primary_network_interface_id = "eni-05d59dd6faf5384da" -> (known after apply)
      ~ private_dns                  = "ip-172-31-42-80.ap-northeast-1.compute.internal" -> (known after apply)
      ~ private_ip                   = "172.31.42.80" -> (known after apply)
      ~ public_dns                   = "ec2-54-250-240-177.ap-northeast-1.compute.amazonaws.com" -> (known after apply)
      ~ public_ip                    = "54.250.240.177" -> (known after apply)
      ~ security_groups              = [
          - "default",
        ] -> (known after apply)
        source_dest_check            = true
      ~ subnet_id                    = "subnet-c1d82c89" -> (known after apply)
      - tags                         = {} -> null
      ~ tenancy                      = "default" -> (known after apply)
      ~ volume_tags                  = {} -> (known after apply)
      ~ vpc_security_group_ids       = [
          - "sg-15d1d56a",
        ] -> (known after apply)

      - credit_specification {
          - cpu_credits = "standard" -> null
        }

      + ebs_block_device {
          + delete_on_termination = (known after apply)
          + device_name           = (known after apply)
          + encrypted             = (known after apply)
          + iops                  = (known after apply)
          + snapshot_id           = (known after apply)
          + volume_id             = (known after apply)
          + volume_size           = (known after apply)
          + volume_type           = (known after apply)
        }

      + ephemeral_block_device {
          + device_name  = (known after apply)
          + no_device    = (known after apply)
          + virtual_name = (known after apply)
        }

      + network_interface {
          + delete_on_termination = (known after apply)
          + device_index          = (known after apply)
          + network_interface_id  = (known after apply)
        }

      ~ root_block_device {
          ~ delete_on_termination = true -> (known after apply)
          ~ iops                  = 0 -> (known after apply)
          ~ volume_id             = "vol-03b7751ed50fc2af9" -> (known after apply)
          ~ volume_size           = 8 -> (known after apply)
          ~ volume_type           = "standard" -> (known after apply)
        }
    }

Plan: 1 to add, 0 to change, 1 to destroy.

------------------------------------------------------------------------

Note: You didn't specify an "-out" parameter to save this plan, so Terraform
can't guarantee that exactly these actions will be performed if
"terraform apply" is subsequently run.

MacBook-Pro-2:getstarted vaivailx$

```
ん？このあとにapply?
やってみるか

```
MacBook-Pro-2:getstarted vaivailx$ terraform apply
aws_instance.example: Refreshing state... [id=i-06b903b99910de839]

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
-/+ destroy and then create replacement

Terraform will perform the following actions:

  ## aws_instance.example must be replaced
-/+ resource "aws_instance" "example" {
      ~ ami                          = "ami-1d37b57b" -> "ami-0c3fd0f5d33134a76" ## forces replacement
      ~ arn                          = "arn:aws:ec2:ap-northeast-1:682816909333:instance/i-06b903b99910de839" -> (known after apply)
      ~ associate_public_ip_address  = true -> (known after apply)
      ~ availability_zone            = "ap-northeast-1a" -> (known after apply)
      ~ cpu_core_count               = 1 -> (known after apply)
      ~ cpu_threads_per_core         = 1 -> (known after apply)
      - disable_api_termination      = false -> null
      - ebs_optimized                = false -> null
        get_password_data            = false
      + host_id                      = (known after apply)
      ~ id                           = "i-06b903b99910de839" -> (known after apply)
      ~ instance_state               = "running" -> (known after apply)
        instance_type                = "t2.micro"
      ~ ipv6_address_count           = 0 -> (known after apply)
      ~ ipv6_addresses               = [] -> (known after apply)
      + key_name                     = (known after apply)
      - monitoring                   = false -> null
      + network_interface_id         = (known after apply)
      + password_data                = (known after apply)
      + placement_group              = (known after apply)
      ~ primary_network_interface_id = "eni-05d59dd6faf5384da" -> (known after apply)
      ~ private_dns                  = "ip-172-31-42-80.ap-northeast-1.compute.internal" -> (known after apply)
      ~ private_ip                   = "172.31.42.80" -> (known after apply)
      ~ public_dns                   = "ec2-54-250-240-177.ap-northeast-1.compute.amazonaws.com" -> (known after apply)
      ~ public_ip                    = "54.250.240.177" -> (known after apply)
      ~ security_groups              = [
          - "default",
        ] -> (known after apply)
        source_dest_check            = true
      ~ subnet_id                    = "subnet-c1d82c89" -> (known after apply)
      - tags                         = {} -> null
      ~ tenancy                      = "default" -> (known after apply)
      ~ volume_tags                  = {} -> (known after apply)
      ~ vpc_security_group_ids       = [
          - "sg-15d1d56a",
        ] -> (known after apply)

      - credit_specification {
          - cpu_credits = "standard" -> null
        }

      + ebs_block_device {
          + delete_on_termination = (known after apply)
          + device_name           = (known after apply)
          + encrypted             = (known after apply)
          + iops                  = (known after apply)
          + snapshot_id           = (known after apply)
          + volume_id             = (known after apply)
          + volume_size           = (known after apply)
          + volume_type           = (known after apply)
        }

      + ephemeral_block_device {
          + device_name  = (known after apply)
          + no_device    = (known after apply)
          + virtual_name = (known after apply)
        }

      + network_interface {
          + delete_on_termination = (known after apply)
          + device_index          = (known after apply)
          + network_interface_id  = (known after apply)
        }

      ~ root_block_device {
          ~ delete_on_termination = true -> (known after apply)
          ~ iops                  = 0 -> (known after apply)
          ~ volume_id             = "vol-03b7751ed50fc2af9" -> (known after apply)
          ~ volume_size           = 8 -> (known after apply)
          ~ volume_type           = "standard" -> (known after apply)
        }
    }

Plan: 1 to add, 0 to change, 1 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value:yes

aws_instance.example: Destroying... [id=i-06b903b99910de839]
aws_instance.example: Still destroying... [id=i-06b903b99910de839, 10s elapsed]
aws_instance.example: Still destroying... [id=i-06b903b99910de839, 20s elapsed]
aws_instance.example: Still destroying... [id=i-06b903b99910de839, 30s elapsed]
aws_instance.example: Destruction complete after 33s
aws_instance.example: Creating...
aws_instance.example: Still creating... [10s elapsed]
aws_instance.example: Still creating... [20s elapsed]
aws_instance.example: Still creating... [30s elapsed]
aws_instance.example: Creation complete after 36s [id=i-075f48d2e50127a75]

Apply complete! Resources: 1 added, 0 changed, 1 destroyed.
MacBook-Pro-2:getstarted vaivailx$

コンソールでは3つ目のインスタンスができている。

planってなにするんだ？


止めるのをしたいのでついでにdestory もためす

```
MacBook-Pro-2:getstarted vaivailx$ terraform destroy
aws_instance.example: Refreshing state... [id=i-075f48d2e50127a75]

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  - destroy

Terraform will perform the following actions:

  ## aws_instance.example will be destroyed
  - resource "aws_instance" "example" {
      - ami                          = "ami-0c3fd0f5d33134a76" -> null
      - arn                          = "arn:aws:ec2:ap-northeast-1:682816909333:instance/i-075f48d2e50127a75" -> null
      - associate_public_ip_address  = true -> null
      - availability_zone            = "ap-northeast-1a" -> null
      - cpu_core_count               = 1 -> null
      - cpu_threads_per_core         = 1 -> null
      - disable_api_termination      = false -> null
      - ebs_optimized                = false -> null
      - get_password_data            = false -> null
      - id                           = "i-075f48d2e50127a75" -> null
      - instance_state               = "running" -> null
      - instance_type                = "t2.micro" -> null
      - ipv6_address_count           = 0 -> null
      - ipv6_addresses               = [] -> null
      - monitoring                   = false -> null
      - primary_network_interface_id = "eni-022576da21e146aa9" -> null
      - private_dns                  = "ip-172-31-43-91.ap-northeast-1.compute.internal" -> null
      - private_ip                   = "172.31.43.91" -> null
      - public_dns                   = "ec2-54-250-38-218.ap-northeast-1.compute.amazonaws.com" -> null
      - public_ip                    = "54.250.38.218" -> null
      - security_groups              = [
          - "default",
        ] -> null
      - source_dest_check            = true -> null
      - subnet_id                    = "subnet-c1d82c89" -> null
      - tags                         = {} -> null
      - tenancy                      = "default" -> null
      - volume_tags                  = {} -> null
      - vpc_security_group_ids       = [
          - "sg-15d1d56a",
        ] -> null

      - credit_specification {
          - cpu_credits = "standard" -> null
        }

      - root_block_device {
          - delete_on_termination = true -> null
          - iops                  = 100 -> null
          - volume_id             = "vol-038be9ff2e90c018b" -> null
          - volume_size           = 8 -> null
          - volume_type           = "gp2" -> null
        }
    }

Plan: 0 to add, 0 to change, 1 to destroy.

Do you really want to destroy all resources?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value: yes

aws_instance.example: Destroying... [id=i-075f48d2e50127a75]
aws_instance.example: Still destroying... [id=i-075f48d2e50127a75, 10s elapsed]
aws_instance.example: Destruction complete after 22s

Destroy complete! Resources: 1 destroyed.
MacBook-Pro-2:getstarted vaivailx$
```

コンソール上ではEC2はとまっていた。
が、どんどんインスタンスができる。
これは運用時に面倒だ。
なんとかする方法があるはず。

再度確認したら、コンソール上では何もなくなっていた。
削除までには時間がかかるみたい。

## 依存関係の定義

https://learn.hashicorp.com/terraform/getting-started/dependencies

### Elastic IPを適用

```
MacBook-Pro-2:getstarted vaivailx$ terraform apply

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  ## aws_eip.ip will be created
  + resource "aws_eip" "ip" {
      + allocation_id     = (known after apply)
      + association_id    = (known after apply)
      + domain            = (known after apply)
      + id                = (known after apply)
      + instance          = (known after apply)
      + network_interface = (known after apply)
      + private_dns       = (known after apply)
      + private_ip        = (known after apply)
      + public_dns        = (known after apply)
      + public_ip         = (known after apply)
      + public_ipv4_pool  = (known after apply)
      + vpc               = (known after apply)
    }

  ## aws_instance.example will be created
  + resource "aws_instance" "example" {
      + ami                          = "ami-0c3fd0f5d33134a76"
      + arn                          = (known after apply)
      + associate_public_ip_address  = (known after apply)
      + availability_zone            = (known after apply)
      + cpu_core_count               = (known after apply)
      + cpu_threads_per_core         = (known after apply)
      + get_password_data            = false
      + host_id                      = (known after apply)
      + id                           = (known after apply)
      + instance_state               = (known after apply)
      + instance_type                = "t2.micro"
      + ipv6_address_count           = (known after apply)
      + ipv6_addresses               = (known after apply)
      + key_name                     = (known after apply)
      + network_interface_id         = (known after apply)
      + password_data                = (known after apply)
      + placement_group              = (known after apply)
      + primary_network_interface_id = (known after apply)
      + private_dns                  = (known after apply)
      + private_ip                   = (known after apply)
      + public_dns                   = (known after apply)
      + public_ip                    = (known after apply)
      + security_groups              = (known after apply)
      + source_dest_check            = true
      + subnet_id                    = (known after apply)
      + tenancy                      = (known after apply)
      + volume_tags                  = (known after apply)
      + vpc_security_group_ids       = (known after apply)

      + ebs_block_device {
          + delete_on_termination = (known after apply)
          + device_name           = (known after apply)
          + encrypted             = (known after apply)
          + iops                  = (known after apply)
          + snapshot_id           = (known after apply)
          + volume_id             = (known after apply)
          + volume_size           = (known after apply)
          + volume_type           = (known after apply)
        }

      + ephemeral_block_device {
          + device_name  = (known after apply)
          + no_device    = (known after apply)
          + virtual_name = (known after apply)
        }

      + network_interface {
          + delete_on_termination = (known after apply)
          + device_index          = (known after apply)
          + network_interface_id  = (known after apply)
        }

      + root_block_device {
          + delete_on_termination = (known after apply)
          + iops                  = (known after apply)
          + volume_id             = (known after apply)
          + volume_size           = (known after apply)
          + volume_type           = (known after apply)
        }
    }

Plan: 2 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

aws_instance.example: Creating...
aws_instance.example: Still creating... [10s elapsed]
aws_instance.example: Still creating... [20s elapsed]
aws_instance.example: Still creating... [30s elapsed]
aws_instance.example: Still creating... [40s elapsed]
aws_instance.example: Creation complete after 47s [id=i-0df49a9ad7c07a7dd]
aws_eip.ip: Creating...
aws_eip.ip: Creation complete after 4s [id=eipalloc-0eac8bf0fcc0e9a72]

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.
MacBook-Pro-2:getstarted vaivailx$
```


### 依存関係の定義

```
## New resource for the S3 bucket our application will use.
resource "aws_s3_bucket" "example" {
  ## NOTE: S3 bucket names must be unique across _all_ AWS accounts, so
  ## this name must be changed before applying this example to avoid naming
  ## conflicts.
  bucket = "terraform-getting-started-guide"
  acl    = "private"
}

## Change the aws_instance we declared earlier to now include "depends_on"
resource "aws_instance" "example" {
  ami           = "ami-2757f631"
  instance_type = "t2.micro"

  ## Tells Terraform that this EC2 instance must be created only after the
  ## S3 bucket has been created.
  depends_on = [aws_s3_bucket.example]
}
```

https://learn.hashicorp.com/terraform/getting-started/dependencies#non-dependent-resources
によると、明示的にdepends_onを指定しなければ依存しない。


## プロビジョニングについて

https://learn.hashicorp.com/terraform/getting-started/provision

### provision

```
  provisioner "local-exec" {
    command = "echo ${aws_instance.example.public_ip} > ip_address.txt"
  }
```
といれると、プロビジョニングもしてくれる

今回のprovisionerはterraformを実行する側で実行するもの。
```
The local-exec provisioner executes a command locally on the machine running Terraform.
```

```
Terraform supports multiple provisioners,
```
とある。chefやpuppetでのプロビジョニングや、fileの転送ができるよう。
ansibleが無いのは、local-execでansible呼び出せばできてしまうからかな？
→これはちがった。後述の記事でlocal-execが実行されるタイミングではsshできないからできない（1年前の記事だけど）
それは先人たちが苦労したり、ツールを使っているみたい。
terraform -> ansibleで構築 https://qiita.com/k1LoW/items/d3ff2fedc8c354b7db67
ツールを使ってterraform->toolかませてansible http://blog.serverworks.co.jp/tech/2018/03/07/post-63012/
ansibleの中でterraformを実行 https://blog.engineer.adways.net/entry/2016/11/11/174438

ざっと見た感じansibleからterraformを実行するのが良さそうだが、
そうでない方式をとっている人が多いので、やるとしたらよく検討しないといけないな。
理想はvagrantのvagrant upのように、terraform applyワンコマンドで必要なサーバーができている状態だけど。
いやでも、家畜のようにインスタンスを作ったり捨てたりするならば、プロビジョニングした
amiがあってそれをもとにterraformでセットアップが適切かもな。

話がとんだので、チュートリアルのプロビジョニングをお試し。
```
aws_instance.example (local-exec): Executing: ["/bin/sh" "-c" "echo 13.230.171.100 > ip_address.txt"]
aws_instance.example: Creation complete after 39s [id=i-0326e014c10dd8abf]
aws_eip.ip: Creating...
aws_eip.ip: Creation complete after 2s [id=eipalloc-05019d40771294ef8]

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.
MacBook-Pro-2:getstarted vaivailx$ cat ip_address.txt
13.230.171.100
MacBook-Pro-2:getstarted vaivailx$
```
実行されて、ファイルも生成されている。


さっきansibleでどうこうとか書いたけど、チュートリアルに以下のように書いていた。
```
Provisioners are only run when a resource is created. They are not a replacement for configuration management and changing the software of an already-running server, and are instead just meant as a way to bootstrap a server.For configuration management, you should use Terraform provisioning to invoke a real configuration management solution.
 ```
その後に書いているね。
provisionersはあくまでブートストラップ用に使うこと。
terraformとは設定管理に対するソリューションとして使うものですよ。
と。
だからprovisionerはそこまで複雑なことはできない（terraformのポリシーを考えると、できるようにする必要はないということかな）


ちなみにチュートリアルに書いてあるとおり、provisioningはインスタンス生成時にしか実行されない。
ipかえて適用しても、destroyしてもファイルは生成されない。

```
MacBook-Pro-2:getstarted vaivailx$ terraform apply
aws_instance.example: Refreshing state... [id=i-056e7b486beaa7273]
aws_eip.ip: Refreshing state... [id=eipalloc-021442fe2417e5d85]

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  - destroy

Terraform will perform the following actions:

  ## aws_eip.ip will be destroyed
  - resource "aws_eip" "ip" {
      - association_id    = "eipassoc-07c40d08d53392ce5" -> null
      - domain            = "vpc" -> null
      - id                = "eipalloc-021442fe2417e5d85" -> null
      - instance          = "i-056e7b486beaa7273" -> null
      - network_interface = "eni-07840e53f4a7eaecf" -> null
      - private_dns       = "ip-172-31-36-41.ap-northeast-1.compute.internal" -> null
      - private_ip        = "172.31.36.41" -> null
      - public_dns        = "ec2-3-113-239-127.ap-northeast-1.compute.amazonaws.com" -> null
      - public_ip         = "3.113.239.127" -> null
      - public_ipv4_pool  = "amazon" -> null
      - tags              = {} -> null
      - vpc               = true -> null
    }

Plan: 0 to add, 0 to change, 1 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

aws_eip.ip: Destroying... [id=eipalloc-021442fe2417e5d85]
aws_eip.ip: Destruction complete after 3s

Apply complete! Resources: 0 added, 0 changed, 1 destroyed.
MacBook-Pro-2:getstarted vaivailx$
MacBook-Pro-2:getstarted vaivailx$ terraform destroy
aws_instance.example: Refreshing state... [id=i-056e7b486beaa7273]

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  - destroy

Terraform will perform the following actions:

  ## aws_instance.example will be destroyed
  - resource "aws_instance" "example" {
      - ami                          = "ami-0c3fd0f5d33134a76" -> null
      - arn                          = "arn:aws:ec2:ap-northeast-1:682816909333:instance/i-056e7b486beaa7273" -> null
      - associate_public_ip_address  = true -> null
      - availability_zone            = "ap-northeast-1a" -> null
      - cpu_core_count               = 1 -> null
      - cpu_threads_per_core         = 1 -> null
      - disable_api_termination      = false -> null
      - ebs_optimized                = false -> null
      - get_password_data            = false -> null
      - id                           = "i-056e7b486beaa7273" -> null
      - instance_state               = "running" -> null
      - instance_type                = "t2.micro" -> null
      - ipv6_address_count           = 0 -> null
      - ipv6_addresses               = [] -> null
      - monitoring                   = false -> null
      - primary_network_interface_id = "eni-07840e53f4a7eaecf" -> null
      - private_dns                  = "ip-172-31-36-41.ap-northeast-1.compute.internal" -> null
      - private_ip                   = "172.31.36.41" -> null
      - public_dns                   = "ec2-54-168-228-201.ap-northeast-1.compute.amazonaws.com" -> null
      - public_ip                    = "54.168.228.201" -> null
      - security_groups              = [
          - "default",
        ] -> null
      - source_dest_check            = true -> null
      - subnet_id                    = "subnet-c1d82c89" -> null
      - tags                         = {} -> null
      - tenancy                      = "default" -> null
      - volume_tags                  = {} -> null
      - vpc_security_group_ids       = [
          - "sg-15d1d56a",
        ] -> null

      - credit_specification {
          - cpu_credits = "standard" -> null
        }

      - root_block_device {
          - delete_on_termination = true -> null
          - iops                  = 100 -> null
          - volume_id             = "vol-01a63ed340842f21f" -> null
          - volume_size           = 8 -> null
          - volume_type           = "gp2" -> null
        }
    }

Plan: 0 to add, 0 to change, 1 to destroy.

Do you really want to destroy all resources?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value: yes

aws_instance.example: Destroying... [id=i-056e7b486beaa7273]
aws_instance.example: Still destroying... [id=i-056e7b486beaa7273, 10s elapsed]
aws_instance.example: Still destroying... [id=i-056e7b486beaa7273, 20s elapsed]
aws_instance.example: Still destroying... [id=i-056e7b486beaa7273, 30s elapsed]
aws_instance.example: Destruction complete after 37s

Destroy complete! Resources: 1 destroyed.
MacBook-Pro-2:getstarted vaivailx$ cat ip_address.txt
cat: ip_address.txt: No such file or directory
MacBook-Pro-2:getstarted vaivailx$
```

### provisionに失敗したリソースの扱い

https://learn.hashicorp.com/terraform/getting-started/provision#failed-provisioners-and-tainted-resources

にあるように、プロビジョニングに失敗したらtainted（汚れている）とterraformは状態を記録する。taintedのリソースにterraform applyすると、対象のリソースは一度壊されて、再生成される。
terraformのポリシーとしてplan実行時に出される期待される変更状態にならなければ、ぶっ壊してでも再生成するらしい。

### destroy時のprovisoin
destroy時のprovisoiningも設定できるとのこと。

https://learn.hashicorp.com/terraform/getting-started/provision#destroy-provisioners

## 変数定義

変数定義方法は3種類

1. \*.tfファイルかterraform.tfvarsか\*.auto.tfvarsに定義して、var.hogeで参照する
2. \*.tfファイルかterraform.tfvarsか\*.auto.tfvarsに変数名のみ定義して、コマンド実行時に入力
3. 別にファイルに定義して、実行時に-var-fileオプションで指定
4. 環境変数TF_VAR_hogeとか設定してterraformコマンドを実行
5. -varオプションを実行時に指定

1はファイルがあるだけで展開されてしまうので、秘匿情報は定義すべきではない。
4は文字列しか定義できないし、これもコマンド履歴に残る可能性があるので秘匿情報は定義すべきではない
秘匿情報は2か3の方法かな
https://learn.hashicorp.com/terraform/getting-started/variables


変数はいくつか型があり、List, Map。
Listなら
```
cidrs = [ "10.0.0.0/16", "10.1.0.0/16" ]
```

Map
```
variable "amis" {
  type = "map"
  default = {
    "us-east-1" = "ami-b374d5a5"
    "us-west-2" = "ami-4b32be2b"
  }
}
```

実行時に変数を出力するときoutputを定義する

```
output "ami" {
  value = aws_instance.example.ami
}
```

## 本番環境ではどうつかうか？

https://learn.hashicorp.com/terraform/getting-started/remote
ローカルやテスト環境づくりにはいいけど、複数人で同じインフラ管理をする場合、
stateファイルはローカルにしかないため、複数人で状態管理しづらい。
→本番でも使えるようにstateファイルを管理できるようにTerraform Cloudというサービスがある。
アカウントを作って定義ファイルでbackendを定義すれば、OK
不要な変更適用はされない。


## 次のステップ

もっと学びたいなら以下をやること

Documentation https://www.terraform.io/docs/index.html
Operations and Development Tracks https://learn.hashicorp.com/terraform/#operations-and-development  アクセス権設定をするための方法とか
example https://www.terraform.io/intro/examples/index.html
import https://www.terraform.io/docs/import/index.html 既存のリソースからstateをとれるらしい。→それなら絶対にbackendを使わなきゃいけないってではなさそうな。。

## 所感

いちいちAWSコンソールを立ち上げなくとも設定変更をかけられるのがよい。
たとえばプロダクション環境と同じ構成だけど台数は制限してステージング環境を立てて、開発環境はSPOTで立てられるようにしておくと利点はあるかな。
どのAMI使う？設定の違いは？構成変更はどこで保存してどう適応する？とか気にする部分は少なくなる。
これ以上はなんかサービスでも作らないと恩恵がわかりにくいな。


## 追記 しばらくたってからもう一度terraformを使おうとする

