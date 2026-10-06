# Runbook de Backup y Restauración — Cliente A

> **Nota sobre anonimización.** Este informe documenta un trabajo real, realizado con
> autorización del propietario. El nombre del cliente y los identificadores de su
> infraestructura se reemplazaron por marcadores (`Cliente A`, `cliente-a`) antes de
> publicarlo. Belona Cybersecurity no publica datos identificables de un cliente sin su
> autorización por escrito.

## Estrategia implementada (Nivel A)
- **Firestore PITR:** recuperación a cualquier punto de los últimos **7 días**.
- **Firestore backups programados:** diarios, retención **14 días**.
- **Storage Object Versioning:** versiones anteriores de cada documento.
- **Código:** respaldado en GitHub (repo `<organizacion>/<repo-privado>`).

**Objetivos:** RPO ~0 (últimos 7 días vía PITR) / RTO ~minutos (base pequeña).
Regla 3-2-1: pendiente Nivel B (copia offsite en bucket separado).

> Todos los comandos se ejecutan en **Google Cloud Shell** (proyecto `cliente-a`).

---

## Verificar que todo está activo
```bash
# PITR y edición de la base
gcloud firestore databases describe --database="(default)" --format="value(pointInTimeRecoveryEnablement)"
# Backups programados
gcloud firestore backups schedules list --database="(default)"
# Versionado de Storage
gcloud storage buckets describe gs://cliente-a.firebasestorage.app --format="default(versioning)"
```

---

## Escenario 1 — Se borró/corrompió algo hace poco (≤ 7 días) → PITR
Consultar datos como estaban en un momento pasado (solo lectura):
```bash
# Reemplazar TIMESTAMP por un instante ISO dentro de los ultimos 7 dias
# Ej: 2026-07-02T10:00:00Z
gcloud firestore export gs://<BUCKET-TEMP>/pitr-recovery \
  --database="(default)" --snapshot-time="TIMESTAMP"
```
Luego se revisan/reimportan los documentos necesarios. Para recuperar TODO a ese
punto, ver Escenario 2 (restaurar a una base nueva).

## Escenario 2 — Restaurar desde un backup diario (hasta 14 días)
```bash
# 1) Listar backups disponibles
gcloud firestore backups list --format="table(name,snapshotTime,state)"
# 2) Restaurar a una base NUEVA (no se puede sobrescribir la default directamente)
gcloud firestore databases restore \
  --source-backup="projects/cliente-a/locations/nam5/backups/BACKUP_ID" \
  --destination-database="restore-tmp"
```
Luego se valida `restore-tmp` y se migran los datos correctos a `(default)`
(o se reconfigura la app). Nunca borrar la base original hasta validar.

## Escenario 3 — Recuperar un documento (PDF) borrado/sobrescrito en Storage
```bash
# 1) Listar TODAS las versiones (incluye borradas) de la ruta del archivo
gcloud storage ls -a "gs://cliente-a.firebasestorage.app/procesos/EXPEDIENTE/ARCHIVO"
# 2) Restaurar una version especifica (usar el numero de generacion #123456)
gcloud storage cp "gs://.../ARCHIVO#GENERACION" "gs://.../ARCHIVO"
```

---

## Prueba periódica (recomendado cada 3-6 meses)
- **Test liviano:** recuperar un archivo de Storage desde una versión anterior (Escenario 3).
- **Test completo:** restaurar un backup a `restore-tmp`, verificar datos, y borrar `restore-tmp`.
- Registrar la fecha del último test aquí:  _(último test: —)_

## Nivel B — offsite / 3-2-1 (IMPLEMENTADO)
- **Bucket separado:** `gs://cliente-a-backups` (multi-region `us`, geo-redundante),
  con lifecycle que borra exports > 90 dias.
- **Export automatico:** Cloud Scheduler job `firestore-backup-weekly` (us-central1),
  domingos 3am (America/Bogota), exporta a `gs://cliente-a-backups/scheduled`.
  Usa la cuenta de servicio `backup-sa@cliente-a.iam.gserviceaccount.com`
  (rol `datastore.importExportAdmin`).
- **Nota:** el prefijo `scheduled` se sobrescribe cada semana (copia "ultima" rodante).
  El historial mas fino lo dan los backups diarios de Nivel A (14 dias) y PITR (7 dias).

### Restaurar desde el export offsite
```bash
# Importa el export offsite a la base (CUIDADO: sobrescribe/mezcla datos).
# Idealmente probar primero importando a una base nueva.
gcloud firestore import gs://cliente-a-backups/scheduled
```

### Verificar / operar el backup automatico
```bash
gcloud scheduler jobs list --location=us-central1
gcloud scheduler jobs run firestore-backup-weekly --location=us-central1   # correr a mano
gcloud storage ls gs://cliente-a-backups/scheduled/                   # ver la ultima copia
```
