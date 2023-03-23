# Tag-Based Access Control

Tag-Based Access Control (TBAC) is a streamlined implementation of Attribute-Based Access Contorl ([ABAC](https://aws.amazon.com/identity/attribute-based-access-control/)) that takes advantage of how the Quilt Catalog provides an unbreakable abstraction layer for viewing, assigning, and using tags.

In particular, we only use ABAC to **grant** access.
Since Quilt Catalog is a single-tenant system running inside the customer's private cloud, it already _restricts_ access to users who have authenticated with the customer-managed SSO system.  Any "back doors" introduced by other AWS Policies are the customer's concern, not ours.

As a result, we believe it is possible to define the entire security configuration using a single user-friendly file we call `data-mesh.yaml`. This would not only compile down to low-level security policies, but also enable the Quilt Catalog provide an elegant interface for viewing, assigning,
and managing tags.

## Design Constraints

The key design challenges are:

1. You can only associate 10 tags (each a single key-value pair) per object
2. Each tag key can only be 128 case-sensitive unicode characters, and each tag value 256
3. Resource tags are stored as tag key-value pairs. A resource tag key can [only] have a single value in a given tag
4. Tag identifiers may disallow characters allowed by the customer's SSO systems
5. Tag semantics are very limited (e.g., "Equals" and "Like", "ForAny" and "ForAll")  

One common pattern to work around the 10-tag limit is emulating multiple values per tag key with a "|val1|val2|" list, since LIKE "|val1|" is well-defined if we don't allow "|" in names.

## Design Goals

1. Do not require customers to change their SSO configuration (since that is usually fixed by corporate IT)
1. Grant customers maximum flexibility to define their own tag keys and tag values (since it is _their_ private cloud)
1. Make it easy for customers to precisely specify intent without having to worry about the implementation
1. Specify configurations via an easy-to-parse data format (versus a document format or programming language) so different tools can easily read, write, visualize, and diff

## Design Proposal

[Sample data-mesh.yaml](./data-mesh.yaml)

1. Specify configuration via a YAML file (`data-mesh.yaml`) modeled after Swagger
1. Explicitly define valid tag keys and values, which may be used by both principals and resources
1. Explicitly specify the mapping from JWT claims (derived from the SSO) onto various tags
1. ONLY create policies that ALLOW access
    1. Requires the user already have bucket-level Access
    1. Highly sensitive files should be on restricted buckets
    1. User-level DENY is handled by SSO
1. Provide a small set of YAML-based policy primitives to express which combinations of principal tags ALLOW access to particular combinations of resource tags
    1. valueEquals: for a given key, resource tag value matches principal tag value
    1. valueLike: for a given key, resource tag matches the specific pattern
    1. hasKey: resource has a tag containing one of these keys
1. Develop an open source 'policy compiler' that generates a reproducible CFT template.
    1. Should also validate correctness
    2. Could optionally compress long names into short identifiers to avoid the 256-character limit.
    3. Can definitely predict which claims can access a given resource tag set.
1. Enhance the Quilt Catalog to view/edit the per-stack `data-mesh.yaml`
    1. Should always validate edits
    2. Might possibly trigger policy recompilation.

## Open Questions

1. Should we encourage/discourage customers from "manually" adding tags directly from AWS?
1. Should we _require_ always storing "multiple values per tag key"?
    1. If so, should we constrain tag values to, e.g., 16 characters?
    1. Or, should we always compress them to human-readable OR opaque identifiers?

## References

- <https://docs.aws.amazon.com/general/latest/gr/aws_tagging.html>
- <https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-tagging.html>
- <https://docs.aws.amazon.com/IAM/latest/UserGuide/access_iam-tags.html>
- <https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction_attribute-based-access-control.html>
- <https://docs.aws.amazon.com/IAM/latest/UserGuide/tutorial_attribute-based-access-control.html>
- <https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_condition_operators.html>
- <https://stackoverflow.com/questions/46062084/how-to-provide-multiple-stringnotequals-conditions-in-aws-policy>
