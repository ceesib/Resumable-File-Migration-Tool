# Resumable-File-Migration-Tool
This project aims to develop a command-line tool that safely migrates files from a source directory on an aging computer to an external storage drive with limited available space. The primary objective is to ensure data integrity and prevent data loss during long-running, interruption-prone file transfers.

The tool will recursively scan a specified source directory and build a persistent manifest containing metadata for all files to be migrated, including paths and file sizes. Files will be transferred in batches based on the available free space on the destination drive, ensuring that disk space is not exhausted during the process.

Each file will be copied to a temporary location on the destination drive and verified before being marked as successfully transferred. Verification will be based on file size and timestamps, with optional checksum validation for higher assurance. Source files will only be deleted after successful verification if the tool is run in “move” mode; otherwise, the default behavior will be copy-only to maximize safety.

The system will persist transfer state to disk, allowing it to resume seamlessly after crashes, power failures, or manual interruptions. Already verified files will be skipped on subsequent runs, while failed or partial transfers will be retried without duplicating completed work.

Progress reporting will provide visibility into overall migration status, including the number of files and total data transferred, as well as per-directory progress where feasible. The tool will log errors and failed transfers for later review.

The project explicitly excludes real-time synchronization, network transfers, graphical user interfaces, and performance optimization beyond what is required for safe, reliable disk I/O. The focus is on correctness, recoverability, and transparency rather than speed.

This tool is intended for single-user, one-time or infrequent data migrations where reliability and resumability are more important than raw performance.
