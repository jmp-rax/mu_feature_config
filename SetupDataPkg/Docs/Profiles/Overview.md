# Configuration Profiles (BIOS Flavors) Implementation

## Table of Contents

- [Introduction](#introduction)
- [Design](#design)

### Description

This document describes the requirements, design considerations and APIs for UEFI Configuration Profiles.

### Revision History

| Revised by   | Date      | Changes           |
| ------------ | --------- | ------------------|
| Oliver Smith-Denny   | 09/15/2022| Initial design |
| Oliver Smith-Denny | 02/27/2023 | XML Spec Update |

## Terms

| Term   | Description                     |
| ------ | ------------------------------- |
| UEFI | Unified Extensible Firmware Interface |
| MFCI | Manufacturer Firmware Configuration Interface |
| FV | Firmware Volume |

### Reference Documents

| Document                                  | Link                                |
| ----------------------------------------- | ----------------------------------- |
| MFCI Documentation | [Link](https://microsoft.github.io/mu/dyn/mu_plus/MfciPkg/Docs/Mfci_Feature/)  |
| SetupVariable Flow | [Link](../Overview/Overview.md) |

## Introduction

### Configuration Profiles

UEFI Configuration Profiles, historically called BIOS Flavors, are sets of defined values for UEFI configuration variables.
Such profiles are useful where different owners may use the same hardware but have different requirements for UEFI
configuration variables, such as one owner requiring Secure Boot enabled and SMT disabled and another owner requiring
Secure Boot disabled and SMT enabled. Configuration profiles are provided by the FW as a means to allow both groups to
use the same hardware and FW, but choosing different profiles with the set of configuration they require.

## Design

### Flow

There will be one generic profile that describes the default values for all UEFI configuration variables. This
generic profile will be generated during build time from one XML configuration file specified in PlatformBuild.py as
`MU_SCHEMA_DIR` to the directory path and `MU_SCHEMA_FILE_NAME` as the XML filename. Additional profiles will be
represented as change files (CSV files generated by the [ConfigEditor UI tool](../../Tools/ConfigEditor.py))
with the profile name as the filename.

Platform owners can develop a configuration profile for their use case. Following examples and the format provided in
the [ConfigurationFiles doc](../ConfigurationFiles/ConfigurationFiles.md), these owners can create an XML change file
describing the set of configuration variables and their values that are in the profile that differ from the generic
profile. The [ConfigEditor UI tool](../../Tools/ConfigEditor.py) may be used to assist in loading the XML and saving
the output into change files, as well as manipulating the values as necessary.

The profiles will be used as overrides to the generic profile, to be published as the config policy. The profiles will
define the entire set of UEFI configuration variables.

_Profiles in the XML update are under active development and this section will be updated as additional work is
completed._

### Profile Update

Profiles will only be added and have values updated during build time.

If a new configuration knob is required to be added to the configuration profile, it must go into the generic profile
with a default value in addition to whichever profiles choose to override it.

If a configuration knob is required to change to a new value with the same structure, it can simply be updated using the
Config Editor UI tool to the new value.

If a new structure is required for an existing configuration knob, then it is required that a new configuration knob
be added, with the old knob being removed as soon as feasible.

### Active Profile Selection

The ActiveProfileSelectorLib library class is intended to be overridden by the OEM/platform to retrieve the active
profile index from the relevant source of truth. If ActiveProfileSelectorLib has a failure fetching the active profile
index, the OEM/platform has the choice of failing to boot, defaulting to the last used profile, or defaulting to the
generic profile.
