The `Flake Initiative` is an attempt to enable clean modular reuse for
the Nix_ expression language. "Flakes" [#flake]_ are what other
languages [#special]_ might call "packages": libraries of conceptually
related functionality meant to be developed and distributed as a unit.
Flakes consist of "flake members", which in other languages
[#special]_ would be called "modules": individual definitions grouped
in a single file, with access to each other's internals and private
definitions where relevant. The initiative includes defining both the
core technical interface for defining and using flakes as well as the
expected conventions/best practices for what flakes should look like,
from the overall structure down to the individual definitions.

.. _Nix: https://nixos.org/nix
.. [#flake] Named after the snowflake logo.
.. [#special] For `Nix`, using the word "package" or "module" in this
	      context would be... problematic.
