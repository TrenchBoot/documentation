
## Agenda/Notes

* Outreach and Engagement
    * TrenchBoot Developers Forum
        * Determine time period and/or dates
            - Piotr feels every 12 mo would be quite long
                - is six months too short to discuss things
                - thinks sept. we should have something
                - OSFC planned for Dec.
            - Rich, should do it when things get cold and are locked up
                - Do TB talks at multiple events
                - We should have an internal event, but maybe make it a DRTM event
            - DK, LPC at end of 24Sept
                - Should have confirmation by the end of June for LPC MC
                - should have slots available for TB talks
            
        * Select chairman to oversee planning/coordination
            - Piotr we would like to help but cannot do it alone
                - will engage IBM POWER about sponsoring
            - Rich recommends bring the topic up at DRTM related meetings and try to find a corporate sponsor

    * TrenchBoot website
        * Identify content that would like to be on the site
            - Piotr: coreboot is hooking to rss feeds that feed into blog
                - will look at what coreboot implementation
                - will look at mkdocs setup to move TB documentation over to
            - DanK: will speak with a colleague, will have a response when back from vacation

        * Discuss approach to maintenance/development
            - should apply GH auto fixes

    * TrenchBoot Social Media
        * Review social media accounts and strategy
            * Twitter
            * LinkedIn project site/group
            * Others?

* Project
    * Moving AMD support forward
        * LZ renaming
            - Piotr thinks AMD should be involved in TB AMD related topics
                - we should also try to get them involved with the call
            - Rich, this is a public meeting, may need to have NDA meeting
        
        * LZ IOMMU approach adoption
            - Ross: just adopt current proposed approach as a starting place
                - yes the pitsaw card could be used
            - Piotr, the current approach is better than nothing
                - are there tests?
            - Kanth: can the pitsaw card be used to test iommu for txt, can be used for AMD
            
        * DRTM log approach adoption
            - Piotr: based on TCG spec
                - this is where the HCL will be useful
                - can test Ross changes for linux kernel
                - will work on "legacy mode" support
            - Ross: need to handle system without the ACPI table
                - can make so that the ACPI table is preferred approach then fail back to non-ACPI table
                - for the LZ will need to also be made to handle non-ACPI table situation (legacy mode)
                - will work on "legacy mode" support
            - Daniel: merge the PR
                - has been confirmed on pc-engines

        * iPXE support
            - Piotr: submitted patches but rejected as too big
                - https://github.com/ipxe/ipxe/pull/300
                - we should care about iPXE

    * Upcoming v2 LKML submission
        - Ross: it looks like it will be going out tomorrow (6/18/2021)

    * GRUB submission
        - Daniel K: currently working with Lukasz and will looking to submit in July
            - will be working with 3mdeb on aligning AMD changes
    
    * Deployment/Adoption support
        * TrenchBoot Hardware Compatibility List
            * How to check if my hardware is supported or can be supported?
                - Piotr: resource constrained but we need something very basic
                    - what all should be checked, log, pcrs, etc
                    - this is QubesOS HCL as an example
                        - https://www.qubes-os.org/hcl/
                    - list of hardware for people to get started with
                - Rich:
                    - there are a lot of things that make you feel better but not gain much
                        - maybe skip over that and focus on community
                    - biggest issues will be with hardware that is not MS certified
                    - both OXT and Qubes HCL are not correct because every system has quirks
            
        * TrenchBoot Canonical Demo
        
        * TrenchBoot as AEM for QubesOS
            - Piotr: would be good to assist with anit-evil maid demo

    * Test automation (Kanth)
        - Rich suggest this is a place for Qemu support for DRTM to enable software based testing
            - at tdf txt lead mentioned txt test suite in FWUPD
            - we should get OEM testing
            - could TB lead DRTM test suite development
            - if there isn't one, who is willing to fund its development
            - live cd is not enough, need to build a cross-community project
            - this can be a theme for the DRTM event
        - Piotr agrees that this FWUPD testing support is desired
        - Kanth, Oracle will be increasing supported platforms and would like to see automated testing/validation
            - Oracle would be interested in helping with building a DRTM test framework
        - Daniel K: GRUB does not have automated testing but it is in progress
            - thinks it would be quite easy to introduce tests for preamble 

* Additional Topics (time permitting)
   * DRTM/TrenchBoot for Arm
        - Stuart: Beta spec will be public by fall and possible reference implementation
        - Rich: coordinate a TB event around Arm event/announcement 
        
   * Plan for SMM
        - intel whitepaper on SMM DRTM protection
        
   * Integration with FWUPD hardware security test

* General Business
    * Open floor for community members
    
    * Next meeting
        - Piotr we missed several topics
            - discuss getting more resources
            - fobnail
            - testing
        - Rich we should not do meetings during the summer and do out-of-band discussions (chat/email)
            - perhaps use OSFC TB slack
        - Will be done virtually via TB slack channel

