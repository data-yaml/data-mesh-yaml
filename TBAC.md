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
3. Tag identifiers may disallow characters allowed by the customer's SSO systems
4. Tag semantics are very limited (e.g., "Equals" and "Like", "ForAny" and "ForAll")  

One common pattern to work around the 10-tag limit is emulating multiple values per tag key with a "|val1|val2|" list, since LIKE "|val1|" is well-defined if we don't allow "|" in names.

## Design Goals

- Do not require customers to change their SSO configuration (since that is usually fixed by corporate IT)
- Grant customers maximum flexibility to define their own tag keys and tag values (since it is _their_ private cloud)
- Make it easy for customers to precisely specify intent without having to worry about the implementation
- Specify configurations via an easy-to-parse data format (versus a document format or programming language) so different tools can easily read, write, visualize, and diff

## Design Proposal

1. Specify configuration via a YAML file (data-mesh.yaml) modeled after Swagger
2. Define distinct tag keys and values for  both principals and resources
3. Explicitly specify the mapping from JWT claims (derived from the SSO) onto various principal tags
4. Provide a small set of YAML-based policy primitives to express which combinations of principal tags do (or do not) have access to particular combinations of resource tags
5. Develop an open source policy compiler that reproducibly generates an appropriate CFT template

[Sample data-mesh.yaml](./data-mesh.yaml)


## References

- <https://docs.aws.amazon.com/general/latest/gr/aws_tagging.html>
- <https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-tagging.html>
- <https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction_attribute-based-access-control.html>
- <https://docs.aws.amazon.com/IAM/latest/UserGuide/tutorial_attribute-based-access-control.html>
- <https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_condition_operators.html>
- <https://docs.aws.amazon.com/IAM/latest/UserGuide/access_iam-tags.html>
- <https://stackoverflow.com/questions/46062084/how-to-provide-multiple-stringnotequals-conditions-in-aws-policy>
- <>