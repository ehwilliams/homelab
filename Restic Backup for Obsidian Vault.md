# Backing Up and Restoring My Obsidian Philosophy Wiki

**Home lab case study** · November 2026

## Goal and result

I built a backup and recovery workflow for my Obsidian philosophy wiki, which contains research notes, original source files, and scripts. I wanted to be able to recover from accidental deletion or a possible drive failure.

I restored the vault from a previous version and verified it by opening recovered files, such as notes and a PDF, and following a link in Obsidian. The backup took 1 minute 41 seconds, the restoration took 1 minute 46 seconds with 4.443 GiB worth of files.

## Setup

| Component                      | Role                                                           |
| ------------------------------ | -------------------------------------------------------------- |
| Fedora workstation             | Hosts the live Obsidian vault and runs restic                  |
| restic                         | Creates encrypted snapshots and retrieves earlier versions     |
| Synology NAS                   | Stores the repository in a dedicated folder on a mounted share |
| systemd user service and timer | Runs the backup daily                                          |
| systemd journal                | Records backup output and failures                             |

**Backup scope** · Obsidian notes as markdown files, PDF's, epub's, python scripts, and git files.

## Implementation

1. I initialized a restic repository in a dedicated folder on the NAS and set the vault as the backup source.
2. I ran a manual backup and confirmed via restic that the snapshot returned no errors.
3. I configured a systemd user service and timer. I tested a manual service start and checked what the journal and snapshot list showed.

## Troubleshooting

The systemd service passed its vault directory check, but restic exited with code `10` and reported a missing repository `config` file. The service pointed to a new folder I hadn’t initialized as a restic repository. I ran `restic init` for the intended destination, then manually started the systemd service, then confirmed the snapshot was present via restic.

## Restore drill

Scenario: Deleted some notes and a couple PDF files to test restoration. Also modified some markdown files by deleting some links to other notes.

Restored the snapshot directly before I made the deletions/modifications. I restored it to another folder on my workstation. I opened the restored vault in Obsidian and found the deleted markdown and PDF files. I verified that all the deleted files were present and all modifications to other markdown files had been restored to their previous state.

## What I learned

Using systemctl to check the status of the service was a large help in troubleshooting as I was able to quickly verify what the exact problem was. On a conceptual level: checking to make sure you've taken the first steps when modifying the approach. In practice this means I should've made sure to initialize the repository when switching from a test repo to a production repo. Something as simple as forgetting to run `init` means an incomplete or failed backup.

**Next improvement** · Adding an off-site backup. Using AWS with a scheduled restic backup would be a big improvement and add redundancy.

