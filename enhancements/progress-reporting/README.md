# MTC Progress Reporting Improvements

The goal of this document is to highlight user experience issues in current implementation of progress reporting in MTC, and propose improvements in some of the areas for a better user experience.

## How progress reporting looks like in MTC?

The Status field in the _MigMigration_ CR contains the information about the progress of ongoing migration. Upon creation of _MigMigration_ CR, the migration transitions from one _Phase_ to another until it reaches the _Completed_ phase. For a user, a _Phase_ is simply a _Step_ among different steps that are involved in the migration. 

## Update in CR Status field for progress reporting

All _Phase_ of miration would be broken down to 3 to 4 high level stages that would be useful to give user a better idea about what is the progress of current migration. This could be implemented by introducing anothor attribute _Stage_ in `status` field of _MigMigration_ CR. This new field could hold values like `Initial`, `Backup`, `Restore` etc, pertaining to the stages that migration is in. This would be more user friedly than what currently is getting displayed, as there are many _Phase_ about which user would not generally care.

```
apiVersion: migration.openshift.io/v1alpha1
kind: MigMigration
[...]
status:
    conditions:
        [...]
    itenerary: <itenerary-type>
    observedDigest: <digest>
    phase: <phases of itenerary>
    stage: <Stage name of current phase>
    startTimestamp: '<time-stamp>'
```