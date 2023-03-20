# data-mesh-yaml
Collaboration as Code for Ubiquitous Data Products

ONLY for Quilt-mediated access 
- Do NOT worry about AWS Users
- Only for Quilt SSO AssumeRole
- Managed entirely via Quilt Admin
- Tags viewed in Quilt Catalog

Two sets of tags
- Principals | Resources
- four-char alphanum code: Human-readable long string
- Always a "|list|"

Three Mappings:
- [JWT claims](https://auth0.com/docs/secure/tokens/json-web-tokens/json-web-token-claims) to Principal Tags
- Policy: Principal Tags to Resource Tags
- Resource Tags to Objects
  - physical S3 URIs (Buckets, Keys)
  - logical Quilt+ URIs (Package, Path) 

[Example Configuration File](./data-mesh.yaml)
