# Remote Attributes Protocol

## Abstract

Add an API that allows decorating CAS entries with arbitrary attributes and a counterpart API to query CAS entries via existing attributes. This should make it possible to have a registry API on top of the existing remote APIs infra that avoids unnecessary redownloads and uploads. RE implementations can also use these attributes to implement customised storage behaviours (like allowing the user to enfore the minimum TTL of a blob on RE). 

## Introduction

As of 2024-09-10, Bazel lacks a unified approach to releasing artifacts. Alas, the community found various and insiduous escape hatches, like creaing publishing scripts that can be run through `bazel run`. 

Though this may seem enough on a first look, it does suffer from multiple issues. Firstly, such an approach makes it impossible to publish multiple artifacts at the same time without adding one more level of script nesting to coordinate multiple run targets.

Furthermore, pushing to an external artifactory may be rather unnecessary when RBE's CAS already is a perfectly valid storage mechanism, and reusing it would avoid additional transfers in and out of the network.

## Proposal

