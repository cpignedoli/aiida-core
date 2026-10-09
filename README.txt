AiiDA CalcJob upload recovery: Daint integration-test evidence
============================================================
Date: 2026-10-09 (Europe/Zurich)
Related issue: https://github.com/aiidateam/aiida-core/issues/7721

These are the author's original screenshots of the controlled upload-failure
test on Daint. This evidence branch is separate from the proposed source-code
changes and is not intended for merging into aiida-core.

01-paused-upload.png
  The CalcJob paused after six failed upload attempts. Direct inspection on
  Daint shows the complete BASIS_MOLOPT and the 64-byte partial POTENTIAL.

02-manual-resume.png
  The author disabled the test-only outage and used verdi process play with
  the same calculation UUID. The calculation reached the scheduler queue.

03-recovered-directory.png
  The same work directory contains the complete inputs and CP2K output.
  BASIS_MOLOPT retains its earlier timestamp; POTENTIAL is now 135825 bytes.

Additional verification from saved logs and file metadata
-------------------------------------------------------
The same CalcJob and remote work directory were reused. BASIS_MOLOPT retained
its SHA-256, inode, modification time and permission bits and was uploaded
only once. All seven expected file checksums matched after recovery. There
were no lost+found backups or duplicate source-data nodes.

CP2K 2024.3 converged the CH4 geometry optimization. One Slurm allocation
completed with exit code 0:0 in 1 minute 42 seconds. The CalcJob and its
Cp2kBaseWorkChain and Cp2kGeoOptWorkChain parents all finished with exit status 0.

The run used one node, four MPI tasks and four allocated CPUs per task, with
a 20-minute time limit. The existing MPS wrapper and thread expression were
preserved; CP2K reported three OpenMP threads per task.

The failure was injected into a test-only transport: after transferring one
complete input and 64 bytes of the next, it aborted its own SSH connection
and blocked its own reconnects. The live installation, shared SSH key and
shared SSH configuration were unchanged. The isolated test used AiiDA
2.10.0.dev0 with the proposed patch, based on upstream commit
237c763c23de4d1acac1ce71bc6863a8cd164248.

This run did not restart the daemon; restart recovery was tested separately
with an isolated daemon and the actual process-play CLI. The wildcard-copy
guard was added after the Daint run and tested in the subsequent 288-test
focused validation. This was a small integration test, not a large-file
performance benchmark or a claim of universal transport compatibility.
