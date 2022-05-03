# Learn Terraform count and for_each

Learn what Terraform count and for_each are and when to use them.

Follow along with these tutorials for
[count](https://learn.hashicorp.com/tutorials/terraform/count) and
[for_each](https://learn.hashicorp.com/tutorials/terraform/for-each) on
HashiCorp Learn.


# 概要
## 初期構成
git checkout tags/count-initial-configuration -b count-initial-configuration

## count
複数の同様な構成のEC2を1つの構成によって実現する。
- resourceの宣言部にcountを宣言
  countで指定した個数分resourceの構成が作成される。

```
   resource "aws_instance" "app" {
   count = var.instances_per_subnet * length(module.vpc.private_subnets)
```

- 個々のリソースにおける値の設定
  別途配列を宣言しておき、配列の要素を指定する際にcount.indexを用いて、個々のresourceに適した値を配列から参照して設定することが可能。

```
subnet_id              = module.vpc.private_subnets[count.index % length(module.vpc.private_subnets)]
```

- outputにおいては「*」を指定することでcount分生成されたリソースの値を出力する事が可能

```
  instances           = aws_instance.app.*.id
```

git checkout tags/count-multiple-instances -b count-multiple-instances



## foreach
複数の同様な構成のEC2を1つの構成によって実現する。
- resourceの宣言部にfor_eachを宣言
  for_eachで指定した個数分resourceの構成が作成される。

```
module "vpc" {
  for_each = var.project
```

- 個々のリソースにおける値の設定
  for_eachで設定したmapの要素を指定する際にeach.key、each.valueを用いて、個々のresourceに適した値をmapから参照して設定することが可能。

```
private_subnets = slice(var.private_subnet_cidr_blocks, 0, each.value.private_subnets_per_vpc)
public_subnets  = slice(var.public_subnet_cidr_blocks, 0, each.value.public_subnets_per_vpc)

name        = "web-server-sg-${each.key}-${each.value.environment}"
vpc_id      = module.vpc[each.key].vpc_id
ingress_cidr_blocks = module.vpc[each.key].public_subnets_cidr_blocks
```

- outputにおいてはforを用いて、foreachの数分生成されたリソースの値を出力する事が可能

```
  value       = { for p in sort(keys(var.project)) : p => module.elb_http[p].this_elb_dns_name }
```

git checkout tags/foreach-multiple-projects -b foreach-multiple-projects
