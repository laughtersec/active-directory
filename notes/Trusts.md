---
tags:
  - domain-enumeration
  - general
---
- A relationship between two domains or forests which allows users of one domain or forest to access resources in the other domain or forest.
- Trust can be automatic or established (forest, external)
- Trusted Domain Objects (TDOs) represent the trust relationships in a domain.
- Trust Direction:
	- One-way trust - unidirectional. Users in the trusted domain can access resources in the trusting domain but the reverse is not true.
	- Two way trust - bi-directional. Users of both domains can access resources in the other domain.
- Trust types:
	- Transitive - Can be extended to establish trust relationships with other domains. 
		- All the default intra-forest trust relationships (Tree-root, Parent-Child) between domains within a same forest are transitive two-way trusts.
	- Non-transitive - Cannot be extended to other domains in the forest. Can be two-way or one-way
		- This is the default trust (called external trust) between two domains in different forests when forests do not have a trust relationship.
- Default/Automatic Trusts
	- Parent-child trust
		- It is created automatically between the new domain and the domain that precedes it in the namespace hierarchy, whenever a new domain is added in a tree.
		- This trust is always two-way transitive.
	- Tree-root trust
		- It is created automatically between whenever a new domain tree is added to a forest root.
		- This trust is always two-way transitive.
- Non-default/Established Trusts
	- External Trusts
		- Between two domains in different forests when forests do not have a trust relationship.
		- Can be one-way or two-way and is nontransitive.
	- Forest Trusts
		- Between forest root domain.
		- Cannot be extended to a third forest (no implicit trust).
		- Can be one-way or two-way transitive.

###### Domain Trust Mapping

```powershell title:"Get a list of all domain trusts for the current domain"
1. Get-DomainTrust
2. Get-DomainTrust -Domain domain.name
```

```powershell title:"Get details about the current forest"
1. Get-Forest
2. Get-Forest -Forest domain.name
```

```powershell title:"Get all domains in the current forest"
1. Get-ForestDomain
2. Get-ForestDomain -Forest domain.name
```

```powershell title:"Get all global catalogs for the current forest"
1. Get-ForestGlobalCatalog
2. Get-ForestGlobalCatalog -Forest domain.name
```

```powershell title:"Map trusts of a forest"
1. Get-ForestTrust
2. Get-ForestTrust -Forest domain.name
```

