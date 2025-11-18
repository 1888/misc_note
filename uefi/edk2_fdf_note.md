## PCD settings
* All PCDs must be declared in a DEC file in order to be used.
* The Dynamic and DynamicEx PCDs can be accessed or modified during execution, as result, they cannot be set within the FDF file.
* All FLASH related PCD settings MUST be set in the FDF file, not in the platform description (DSC) file. The
FDF file has the final values for Flash Related PCDs. If a DSC file contains a duplicate PCD setting, the
FDF file's PCD setting takes precedence and it is recommended that the build tools throw a warning
message on the PCD defined in the DSC file.

## 2.1.2 Precedence of PCD Values
The following provides the precedence (high to low) to assign a value to a PCD.
* Command-line, --pcd flags (left most has higher priority)
* DSC file, Component INF <Pcd*> section statements
* FDF file, grammar describing automatic assignment of PCD values
* FDF file, SET statements within a section
* FDF file, SET statement in the [Defines] section
* DSC file, global [Pcd*] sections
* INF file, PCD sections, Default Values
* DEC file, PCD sections, Default Values

In addition to the above precedence rules, PCDs set in sections with architectural modifiers take precedence over PCD sections that are common to all architectures.

If a PCD is listed in the same section multiple times, the last one is used.

