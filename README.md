# Resumable-File-Migration-Tool
This project aims to develop a command-line tool that safely migrates files from a source directory on an aging computer to an external storage drive with limited available space. The primary objective is to ensure data integrity and prevent data loss during long-running, interruption-prone file transfers.

The tool will recursively scan a specified source directory and build a persistent manifest containing metadata for all files to be migrated, including paths and file sizes. Files will be transferred in batches based on the available free space on the destination drive, ensuring that disk space is not exhausted during the process.

Each file will be copied to a temporary location on the destination drive and verified before being marked as successfully transferred. Verification will be based on file size and timestamps, with optional checksum validation for higher assurance. Source files will only be deleted after successful verification if the tool is run in “move” mode; otherwise, the default behavior will be copy-only to maximize safety.

The system will persist transfer state to disk, allowing it to resume seamlessly after crashes, power failures, or manual interruptions. Already verified files will be skipped on subsequent runs, while failed or partial transfers will be retried without duplicating completed work.

Progress reporting will provide visibility into overall migration status, including the number of files and total data transferred, as well as per-directory progress where feasible. The tool will log errors and failed transfers for later review.

The project explicitly excludes real-time synchronization, network transfers, graphical user interfaces, and performance optimization beyond what is required for safe, reliable disk I/O. The focus is on correctness, recoverability, and transparency rather than speed.

This tool is intended for single-user, one-time or infrequent data migrations where reliability and resumability are more important than raw performance.

Classes: 
    Scanner → Planner → Transfer Engine → Verifier → State Tracker
                                   ↘ Progress Monitor

1. Scanner
Purpose

The Scanner is responsible for discovering what exists in the source directory.

What it does

Walks the source directory recursively

Identifies:

Files

Directories

Collects metadata for each file:

Relative path

Size

Last modified time

(Optionally) file type

What it does not do

Does not copy files

Does not check destination

Does not decide transfer order

Inputs

Source root directory

Optional exclusion rules (system folders, temp files)

Outputs

A complete file inventory / manifest

Logical directory structure representation

Key idea

“What needs to be moved, and how big is it?”

2. Planner
Purpose

The Planner decides what to transfer next, given constraints.

What it does

Checks available space on the destination drive

Applies a safety margin

Selects a batch of files that fit within that space

Skips files already marked as completed

What it does not do

Does not copy or verify files

Does not change file state directly

Inputs

File manifest (from Scanner)

Current state (from State Tracker)

Destination free space

Outputs

An ordered list of files for the next transfer batch

Key idea

“Given the current situation, what is safe to move now?”

3. Transfer Engine
Purpose

The Transfer Engine physically moves data.

What it does

Creates directory structure on destination

Copies files to temporary locations

Handles partial transfers and retries

Ensures source files are untouched until success

What it does not do

Does not decide which files to copy

Does not decide when a file is ‘done’

Does not calculate progress summaries

Inputs

Transfer batch (from Planner)

Source and destination paths

Outputs

Raw transfer results (success, partial, failure)

Temporary files on destination

Key idea

“Move bytes from A to B safely.”

4. Verifier
Purpose

The Verifier ensures copied data is correct.

What it does

Compares source and destination files

Performs:

Size checks

Timestamp checks

Optional checksums

Approves or rejects transferred files

What it does not do

Does not copy data

Does not delete source files automatically

Inputs

Source file metadata

Destination file data

Outputs

Verification result per file (pass / fail)

Diagnostic information on mismatches

Key idea

“Can we trust that this file arrived intact?”

5. State Tracker
Purpose

The State Tracker is the memory of the system.

What it does

Persists file-level state:

Pending

In-progress

Transferred

Verified

Failed

Records errors and retry counts

Enables resume after interruption

What it does not do

Does not copy files

Does not verify data

Does not decide transfer order

Inputs

Updates from Transfer Engine and Verifier

Outputs

Current authoritative migration state

Key idea

“What has already happened, and what still needs to happen?”

6. Progress Monitor
Purpose

The Progress Monitor provides visibility.

What it does

Aggregates state information

Displays:

Overall progress

Per-directory progress

Current batch status

Reports errors and retries

What it does not do

Does not influence logic

Does not change state

Inputs

State Tracker data

Planner and Transfer events

Outputs

Human-readable progress and statistics

Key idea

“How far along are we, and is anything wrong?”

How they fit together (mentally)

Scanner defines scope

Planner defines next steps

Transfer Engine moves data

Verifier confirms correctness

State Tracker ensures resumability

Progress Monitor ensures trust

