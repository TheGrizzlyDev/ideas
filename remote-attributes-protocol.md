# Remote Attributes Protocol

## Abstract

Add an API that allows decorating CAS entries with arbitrary attributes and a counterpart API to query CAS entries via existing attributes. This should make it possible to have a registry API on top of the existing remote APIs infra that avoids unnecessary redownloads and uploads. RE implementations can also use these attributes to implement customised storage behaviours (like allowing the user to enfore the minimum TTL of a blob on RE). 

## Introduction

As of 2024-09-10, Bazel lacks a unified approach to releasing artifacts. Alas, the community found various and insiduous escape hatches, like creating publishing scripts that can be run through `bazel run`. 

Though this may seem enough on a first look, it does suffer from multiple issues. Firstly, such an approach makes it impossible to publish multiple artifacts at the same time without adding one more level of script nesting to coordinate multiple run targets.

Furthermore, pushing to an external artifactory may be rather unnecessary when RBE's CAS already is a perfectly valid storage mechanism, and reusing it would avoid additional transfers in and out of the network.

## Proposal

Remote Attributes Protocol would allow 2 simple operations, one to add attributes to a CAS entity and the other to retrieve a list of CAS entities with their attributes, given a subset of said attributes.

```protobuf
message CasAttribute {
  string name = 1;
  string value = 2;
}

message SetAttributesRequest {
  Digest digest = 1;
  repeated CasAttribute attributes = 2;
}

message SetAttributesResponse {}

message SearchAttributesAndBlobsRequest {
  repeated CasAttribute attributes = 1;
}

message SearchAttributesAndBlobsResponse {
  Digest digest = 1;
  repeated CasAttribute attributes = 2;
}

service RemoteAttributes {
  rpc SetAttributes(SetAttributesRequest) returns (SetAttributesResponse);
  rpc SearchAttributesAndBlobs(SearchAttributesAndBlobsRequest) returns (stream SearchAttributesAndBlobsResponse);
}
```

this would allow a client to annotate CAS objects with:
- attributes that can help identify blobs relating to a specific release of the user's software
- attributes with well-known meaning that can affect how a blob is stored (eg: we can imaging an attribute like `"min-ttl": "30d"` that sets the minimum time-to-live of a blob to 30 days)
- attributes specific to a RBE implementation with unique behaviours

On the client side one could imagine the 2 following interfaces (examples based on Bazel, even though the API is not restricted to Bazel):
- Searching and fetching blobs via attributes
```python
def _my_module_impl(mctx):
  for (digest, attributes) in mctx.search_attributes_and_blobs({"release": "0.0.1", "os": "Linux/Debian"}):
    if "distro" in attributes and attributes["distro"] == "Debian":
      mctx.download_from_cas(digest, output="release.deb")
```
- Adding attributes to a blob via rules (which can be pushed by running a dedicated command like `bazel release/publish`)
```python
def _my_rule_impl(ctx):
  # ... do something ...
  return [
    # ... DefaultInfo(...) ...
    AttributesInfo(
      blobs = {
        "some_output.txt": {
          "distro": ctx.attr.distro,
        },
      },
    ),
  ]
```
- Adding attributes manually by invoking the command line: `bazel release/publish --blobs-regex=*.txt --attributes=distro:Debian,os:Linux`
