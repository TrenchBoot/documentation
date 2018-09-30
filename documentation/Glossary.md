Glossary
========

Provided are definitions of terms used throughout TrenchBoot's documents and
designs to encourage a common vocabulary and understanding.

# Table of Contents

* [Dynamic Launch](#dynamic-launch)
* [Explicit Trust](#explicit-trust)
* [Implicit Trust](#implicit-trust)
* [Root of Trust](#root-of-trust)
* [Static Launch](#static-launch)
* [Transitive Trust](#transitive-trust)
* [Trust](#trust)
* [Trust Anchor](#trust-anchor)
* [Trust Boundary](#trust-boundary)
* [Trustee](#trustee)
* [Trustor](#trustor)

<hr/>

#### Dynamic Launch:
A system launch that can be done repeatedly with the execution code able to
reside at different locations in memory. This is sometimes referred to as a
"Late Launch".

#### Explicit Trust:
When a trustor has explicitly established a degree of trust with a trustee

#### Implicit Trust:
When a trustor has relied upon a trustee to establish a degree of trust with
another trustee

#### Root of Trust:
An idempotent mechanism whereby the result is used to assert a fact about the
entity it acted upon.

#### Static Launch:
A system launch that is a one time execution with the execution code at a fixed
location in memory

#### Transitive Trust:
An operation conducted by a trustor that consists of one or more mechanisms
used to assess one or more facts about a trustee before allowing the trustee to
be included within the trustor's trust boundary and delegated the authority to
act as a trustor.

#### Trust:
Assured reliance on the properties, ability, strength, or truth of an entity.

#### Trust Anchor:
The result of a RoT mechanism that is fact being relied upon to assert
correctness, e.g. trustworthiness

#### Trust Boundary:
A demarcation that identifies a subset of entities as those that a trustor has
explicitly or implicitly established as trustworthy

#### Trustee:
An entity that is trusted by another entity

#### Trustor:
An entity that establishes a degree of trust of another entity

