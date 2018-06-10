Introduction to Late Launch
===========================

This is a brief introduction of the "Late Launch" process on x86-based systems
to establish a Dynamic Root of Trust for Measurement (DRTM). On x86 platforms
that support the capability, it is possible to initialize the CPU in two
manners. The first is a static launch that is started by a Power-On/Soft
Reset/Hard Reset sequence that results in the CPU executing firmware located at
the Reset Vector. The second, which is available with Intel TXT or AMD SVM, is
started by executing an instruction that set the CPU into a known state and
begins executing a preloaded execution environment. This second launch type is
referred to as a "Late Launch" as this launch can happen at anytime after a
static launch has occurred.

# What problem does "Late Launch" resolve?

When establishing "trust" about what is executing on a system, the mechanism
used is called "transitive trust". A core principle to "transitive trust" is
that before execution can be handed over to an externally loaded subsequent
module, properties about that module must be established. The means by which
these properties are established is accomplished through either verification
and/or measurement. When this is done as a recursive load, measure, pass
control process, a chain of evidence is created that links the integrity of
each module back to the first module in the chain. When this first module is
the "static" firmware loaded from the CPU's reset vector, it is referred to as
the Static Root of Trust Measurement (SRTM). When this first module is an
"dynamic" executable, it is referred to as a Dynamic Root of Trust Measurement
(DRTM).

The issue arises during a legacy BIOS boot process, there can and often are
code modules, e.g. Option ROMs, that are not measured before they execute and
which have access to system memory. This creates a "gap" in the trust chain
which results in an inability to trust what is executing on a system after the
gap. To overcome this gap and to establish a new trust chain, the Late Launch
process enables the system to move into a known state where a DRTM is taken.
The DRTM is the used to build up a trust chain linking up to the end execution
environment.
