# MTC Progress Reporting Improvements

The goal of this document is to highlight user experience issues in current implementation of progress reporting in MTC, and propose improvements in some of the areas for a better user experience.

## How progress reporting looks like in MTC?

The Status field in the _MigMigration_ CR contains the information about the progress of ongoing migration. Upon creation of _MigMigration_ CR, the migration transitions from one _Phase_ to another until it reaches the _Completed_ phase. For a user, a _Phase_ simply means a one of the _Steps_ in the migration. 

Here is an example of _MigMigration_ status field during an ongoing migration:

```yml
  conditions:
  - category: Advisory
    lastTransitionTime: "2020-09-15T14:54:46Z"
    message: 'Step: 7/33'
    reason: InitialBackupCreated
    status: "True"
    type: Running
  - category: Required
    lastTransitionTime: "2020-09-15T14:53:15Z"
    message: The migration is ready.
    status: "True"
    type: Ready
  itenerary: Final
  observedDigest: 9834d071975562d5e2c2eb855bca6950711ded8a0e45af5307fa56cd0f5ba3c7
  phase: InitialBackupCreated
  startTimestamp: "2020-09-15T14:53:15Z"
```

The `message` field shows how many steps/phases have been completed so far in the migration. The total number of steps change based on whether it's a stage migration or a full migration. The MTC UI uses this field to derive progress in percentages and depicts it in the form a progress bar:

![MTC Progress Bar](./mtc-progress-bar.png)

### Issues with the current implementation

* In a full migration, there are over 30 steps/phases involved. The migration transitions from one phase to another until completion. All these phases are posted to the Status field of the _MigMigration_ CR as and when encountered. We cannot expect an end user to know the meaning of each of these phases. For instance, one of the migration phases is `EnsureCloudSecretPropagated`. While, from a debugging point of view, knowing that the migration is attempting to propagate cloud secret is an important piece of information, it is questionable how much value it adds to the end user experience. 

* Some phases in migration run for a longer period of time than other phases. For such long running phases, MTC currently does not have a way to inform the user of the progress of the ongoing phase itself. For instance, during an initial Backup, the migration controller enters `EnsureInitialBackup` phase where it creates a Velero Backup object and transitions to `InitialBackupCreated` phase upon successful creation of Backup. It then waits until the underlying Backup object has a Completed / PartiallyFailed / Failed condition. While the backup runs in the background, the end user only sees `InitialBackupCreated` in the progress bar. The problem aggrevates when the Backup contains a huge number of resources. The progress bar is stuck in `InitialBackupCreated` phase with a certain percentage value for a very long period of time. The user is left with no reason to believe that there's actually something happening in the background. 