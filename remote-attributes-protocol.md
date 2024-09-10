# Remote Attributes Protocol

## The problem

As of 2024-09-10, Bazel lacks a unified approach to releasing artifacts. Alas, the community found various and insiduous escape hatches, like creaing publishing scripts that can be run through `bazel run`. 

Though this may seem enough on a first look, it does suffer from multiple issues. Firstly, such an approach makes it impossible to publish multiple artifacts at the same time without adding one more level of script nesting to coordinate multiple run targets.

Furthermore, pushing to an external artifactory may be rather unnecessary when RBE's CAS already is a perfectly valid storage mechanism, and reusing it would avoid additional transfers in and out of the network.

