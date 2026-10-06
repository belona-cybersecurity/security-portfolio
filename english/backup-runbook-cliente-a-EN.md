# Backup & Restore Runbook — Cliente A

> **Anonymization note.** This report documents real work performed under authorization
> from the owner. The client's name and infrastructure identifiers were replaced with
> placeholders (`Cliente A`, `cliente-a`) before publication. Belona Cybersecurity does not
> publish identifiable client data without written authorization.

## Implemented strategy
- **Firestore PITR (Point-in-Time Recovery):** restore to any point within the last **7 days**.
- **Firestore scheduled backups:** daily, **14-day** retention.
- **Storage Object Versioning:** previous versions of each document.
- **Offsite (3-2-1):** weekly Firestore export to a separate `us` multi-region bucket.
- **Code:** backed up on GitHub (`<organizacion>/<repo-privado>`).

**Objectives:** RPO ~0 (last 7 days via PITR) / RTO ~minutes (small dataset).
**3-2-1 rule:** met — 3 copies, 2 locations, 1 offsite.

> All commands run in **Google Cloud Shell** (project `cliente-a`).

## Verify everything is active
```bash
gcloud firestore databases describe --database="(default)" --format="value(pointInTimeRecoveryEnablement)"
gcloud firestore backups schedules list --database="(default)"
gcloud storage buckets describe gs://cliente-a.firebasestorage.app --format="default(versioning)"
```

## Scenario 1 — Recent deletion/corruption (≤ 7 days) → PITR
```bash
# Read data as of a past instant (ISO within the last 7 days)
gcloud firestore export gs://<TEMP-BUCKET>/pitr-recovery \
  --database="(default)" --snapshot-time="TIMESTAMP"
```

## Scenario 2 — Restore from a daily backup (up to 14 days)
```bash
gcloud firestore backups list --format="table(name,snapshotTime,state)"
gcloud firestore databases restore \
  --source-backup="projects/cliente-a/locations/nam5/backups/BACKUP_ID" \
  --destination-database="restore-tmp"
```
Validate `restore-tmp`, then migrate the correct data to `(default)`. Never delete the original until validated.

## Scenario 3 — Recover a deleted/overwritten document in Storage
```bash
gcloud storage ls -a "gs://cliente-a.firebasestorage.app/procesos/CASE/FILE"
gcloud storage cp "gs://.../FILE#GENERATION" "gs://.../FILE"
```

## Offsite backup (Level B) — operation
- **Bucket:** `gs://cliente-a-backups` (`us` multi-region, 90-day lifecycle).
- **Automated export:** Cloud Scheduler job `firestore-backup-weekly` (us-central1), Sundays 3am, exports to `gs://cliente-a-backups/scheduled` using the service account `firestore-backup@...` (role `datastore.importExportAdmin`).

```bash
gcloud scheduler jobs list --location=us-central1
gcloud scheduler jobs run firestore-backup-weekly --location=us-central1   # run manually
gcloud storage ls gs://cliente-a-backups/scheduled/                   # view latest copy
# Restore from offsite export (CAUTION: overwrites/merges data)
gcloud firestore import gs://cliente-a-backups/scheduled
```

## Periodic test (recommended every 3-6 months)
- **Light test:** recover a Storage file from a previous version (Scenario 3).
- **Full test:** restore a backup to `restore-tmp`, verify, then delete `restore-tmp`.
- Last test date: _(—)_
