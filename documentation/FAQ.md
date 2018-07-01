TrenchBoot FAQ
==============

1. [Why does TrenchBoot use an intermediate
   launcher?](https://github.com/TrenchBoot/trenchboot/blob/drtm-blueprint/documentation/FAQ.md#1-why-does-trenchboot-use-an-intermediate-launcher)
2. [What are the benefits of measurement over signature
validation?](https://github.com/TrenchBoot/trenchboot/blob/drtm-blueprint/documentation/FAQ.md#2-what-are-the-benefits-of-measurement-over-signature-validation)
3. [What do I need to incorporate TrenchBoot into my
system?](https://github.com/TrenchBoot/trenchboot/blob/drtm-blueprint/documentation/FAQ.md#3-what-do-i-need-to-incorporate-trenchboot-into-my-system)
4. [Where do I start if I want to help with
contributions?](https://github.com/TrenchBoot/trenchboot/blob/drtm-blueprint/documentation/FAQ.md#4-where-do-i-start-if-i-want-to-help-with-contributions)


## 1. Why does TrenchBoot use an intermediate launcher?

For Linux systems doing both verified(secure) and measured boot, there is an
intermediary that handles the security enforcement. For verified boot it is the
UEFI shim loader and for measured boot it is tboot. TrenchBoot replaces these
intermediary loaders with a common Linux-based loader that provides a rich
security processing framework. One role that TrenchBoot does not fulfill is
that the UEFI shim also serves as a trust delegation point that transitions
from Microsoft Authority to Distribution/Installer/No Authority. The response
why this is not of concern will be addressed in Question 2.


## 2. What are the benefits of measurement over signature validation?

It is important to understand that one solution is not necessarily more
beneficial over the other. Measurement and Verification each have their merits
and it is important to understand the environment and requirements of the
solution. In the case of verification, it provides a one-time strong assertion
to origination and correctness that relies on Authorities and Control which
becomes brittle when dealing with delegating control. For example when
verification is being used as the Root of Trust that the transitive trust
builds upon, these solutions are strongest when the ecosystem is closed and
under control of a core entity. Where as measurement provides for establishing
a strong assertion to correctness that can be repeatedly extended and verified.
It therefore relies on the ability to know what correct is and to securely
verify measurement with expected correctness.


## 3. What do I need to incorporate TrenchBoot into my system? 

TrenchBoot is a framework that allows you to build a Linux kernel with a
tailored, embedded initramfs that functions as an intermediate loader to launch
your system. You will need to use the build system to select the security
engine components you desire, provide any necessary configurations, and build
an instance of the loader. After that, you configure your system boot to launch
the loader.


## 4. Where do I start if I want to help with contributions?

The [TrenchBoot
Blueprints](https://github.com/TrenchBoot/trenchboot/tree/drtm-blueprint/blueprints)
are how feature requests are collected for the project. Check if there is a
blueprint that is of interested, if not, submit a blueprint via a pull request
for a feature you would like to see implemented.
