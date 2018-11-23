=======================
Nix Flake Specification
=======================
----------------------
Version 0.0.0
----------------------

..  DANGER::
    This specification covers a prototype implementation of the
    specified interface. It has not yet been adopted by the community
    and is more subject to change and/or abandonment than usual.

Introduction
==============

.. include:: description.rst

This specification formalizes the technical interface for `Nix`
flakes. It includes of formats and semantics for flake.nix_,
describing the metadata and members of the flakes, and for
flake.lock, a "lock file" pinning the flake's dependencies for a
particular evaluation. It also defines the format and semantics of the
files defining the individual flake members.

flake.nix
===========
The `flake.nix` file describes the flake being defined, including
metadata and enumerating its members. Here is an example illustrating
the format:

.. code-block:: nix

   {
     metadata = {
       name = "flake-example";

       version = { major = 1; patch = 1; }; # i.e. semver 1.0.1

       description = "A fake flake to illustrate flake.nix";
     };

     dependencies = {
       foo = {
         good-versions = [ { major = 2; minor = 1; } ];

         bad-version-ranges = [ { upper-bound = {
           version.major = 2;

           included = false;
         } } ];

         # So, we depend on "foo", we know that versions prior to
         # 2.0.0 won't work, and we expect 2.1.x and higher (but
         # lower than 3.x) will work, but we're not sure about 2.0
         # or anything 3.x or higher.
       };

       foo-legacy = {
         name = "foo";

         good-versions = [ { major = 1; } ];

         bad-version-ranges = [ { lower-bound = {
           version.major = 2;

           included = true;
         } } ];

         # One of our modules still depends on the old version of
         # foo, which we name locally foo-legacy.
       };
     };

     members = {
       top-level = ./src/default.nix;

       named = {
         some-member = ./src/some.nix;
       };
     };

     flake-specification-version.major = 0;
   }

`flake.nix` should define a `Nix` expression evaluating to an
attribute set with the following keys and values:

metadata
---------

This key is required. Its value specifies general metadata about the
flake, represented as an attribute set with the following keys:

name
  This key is required. Its value is a string providing, surprisingly,
  the name of the flake.

.. _version set:

version
  This key is required. Its value represents the version of the flake,
  following `Semantic Versioning 2.0.0`_ semantics, as an attribute
  set with the keys `major`, `minor`, and `patch` whose values are
  integers corresponding to the `major`, `minor`, and `patch` versions
  of the package, respectively. The `major` version is required, while
  the `minor` and `patch` versions individually default to `0` if
  omitted. This representation is referred to as a `version set`
  elsewhere in this specification.

.. _Semantic Versioning 2.0.0: https://semver.org/spec/v2.0.0.html

description
  This key is optional. Its value is a string containing a short
  human-readable description of the flake, intended for a quick
  overview.

.. admonition:: To be determined

   Should we have other fields here? A long description, in RST maybe?

dependencies
-------------

.. _Local Names:
.. sidebar:: Local Names

   In most cases, the `local name` of some dependency is simply the
   name of the flake being depended upon. In some cases, however, it's
   necessary to depend on two different versions of the same upstream
   flake, for example if some package set-providing flake wanted to
   use a different version of `nixpkgs` for evaluation-time packages
   (i.e. those used in import-from-derivation) than the version used
   for build-time packages to reduce the incidence of large rebuilds
   at evaluation time. To support this use case, flake dependency
   specifications can use an arbitrary `local name` to locally refer
   to some given dependency and use the `name` key in the dependency
   specification to denote the actual name of the depended-upon flake.
   So, going back to the package set example, we might have dependency
   keys `nixpkgs-eval` and `nixpkgs-build` to denote the separate
   sets, even if the same range of allowed versions is given for both.

This key is optional. Its value is an attribute set that enumerates
the flakes that this flake depends on for its proper functioning. The
keys of the set correspond `local name` of that dependency (see the
`local name sidebar`_ for more details). The values of the set are
themselves attribute sets with the following keys:

name
  This key is optional, defaulting to the `local name` of the
  dependency. Its value is a string denoting the actual upstream name
  of the flake. See the `local name sidebar`_ for more details.

good-versions
  This key is optional, defaulting to an empty list. Its value is a
  list of versions of this dependency that are known to work with this
  flake, represented as a `version set`_.

bad-version-ranges
  This key is optional, defaulting to an empty list. Its value is a
  list of version ranges of this dependency that this flake is
  expected **not** to work with, represented as a set with the
  following fields:

  upper-bound
    This key is optional. Its value is a set with a `version` field
    indicating the upper bound of the given bad version range,
    represented as a `version set`_, and an `included` boolean field
    indicating whether or not the range is inclusive or exclusive of
    its upper bound. If this key is omitted, every version is
    considered lower than the upper bound.
  lower-bound
    This key is optional. Its value is a set with a `version` field
    indicating the lower bound of the given bad version range,
    represented as a `version set`_, and an `included` boolean field
    indicating whether or not the range is inclusive or exclusive of
    its lower bound. If this key is omitted, every version is
    considered greater than the lower bound.

  It is an error to omit both bounds from a range, as that would
  denote a dependency with no working versions.

Taken together, `good-versions` and `bad-version-ranges` can tell you
the expected-to-be-good versions (those in `good-versions` and those
that are semver-replaceable with those in `good-versions`, as long as
they don't match any of the `bad-version-ranges`), the
expected-to-be-bad versions (those matching one of the
`bad-version-ranges`), and those whose suitability is unknown
(everything else). These categories can be leveraged by dependency
resolution tools as well as CI systems to make appropriate dependency
composition decisions.

.. _local name sidebar: `Local Names`_

.. admonition:: To be determined

   Should we have testing dependencies? Required plugins?

members
---------

This key is required. Its value specifies the files defining this
flake's members, represented as an attribute set with the following
keys:

top-level
  This key is optional. Its value is a path to the file defining the
  top-level member of the flake, i.e. the one that is accessible
  to dependents via this flake's local name alone.
named
  This key is optional. Its value gives the paths of all named members
  of the flake, i.e. the ones that are accessible to dependents via
  this flake's local name and the member's name, represented as a set
  whose keys are member names and whose values are the path to the
  file defining the corresponding flake member.

Note that there must be at least one member defined, whether it be the
`top-level` one or a `named` one.

.. admonition:: To be determined

   Should we restrict paths to be within the current directory or its
   children?

flake-specification-version
----------------------------

This key is required. Its value specifies the version of the flake
specification the `flake.nix` was written to conform to, represented
as a `version set`_.
