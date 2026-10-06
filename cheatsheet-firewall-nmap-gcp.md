# Chuleta — Firewall, nmap y GCP (Cloud Shell)

**Regla de oro:** crear regla ALLOW = **ABRIR** un puerto · borrar la regla = **CERRAR**.
nmap solo **lee** (escanea), NO cambia nada; quien abre/cierra es el **firewall**.

---

## Entorno (cómo empezar)
```bash
# Cloud Shell: boton >_ arriba a la derecha en console.cloud.google.com
gcloud config get-value project            # ver proyecto activo
gcloud auth list                           # ver cuentas
gcloud config set account TU_EMAIL         # activar una cuenta
```

## Reglas de firewall (VPC)
```bash
# VER todas las reglas
gcloud compute firewall-rules list

# VER una regla en detalle
gcloud compute firewall-rules describe NOMBRE_REGLA

# ABRIR un puerto (crear regla ALLOW)
gcloud compute firewall-rules create NOMBRE_REGLA \
  --network=default \
  --direction=INGRESS \
  --action=ALLOW \
  --rules=tcp:PUERTO \
  --target-tags=ETIQUETA \
  --source-ranges=0.0.0.0/0        # cualquier IP; o TU_IP/32 para una sola

# CERRAR un puerto (borrar la regla)
gcloud compute firewall-rules delete NOMBRE_REGLA

# MODIFICAR una regla (ej. restringir el origen = hardening)
gcloud compute firewall-rules update NOMBRE_REGLA --source-ranges=TU_IP/32
```

## Escanear puertos (nmap = solo LEE)
```bash
sudo apt-get install -y nmap               # instalar (Cloud Shell lo borra al reiniciar la sesion)

# RECONOCIMIENTO completo (primera vez): todos los puertos + versiones + scripts + guardar
sudo nmap -sS -sV -sC -p- -T4 -oN recon.txt IP_OBJETIVO

# VERIFICAR un puerto puntual (tras un cambio)
nmap -Pn -p PUERTO IP_OBJETIVO

# SYN scan (estandar profesional; necesita sudo)
sudo nmap -sS -Pn -p 22,80,443 IP_OBJETIVO

# Ver SOLO los abiertos de un resultado guardado
grep open recon.txt
```
**Estados de un puerto:**
- `open` = hay servicio escuchando **Y** el firewall lo permite
- `closed` = el firewall permite, pero **no hay servicio** (te rechaza)
- `filtered` = el **firewall bloquea** (no responde)

## VM de laboratorio (Compute Engine)
```bash
gcloud compute instances create NOMBRE --zone=ZONA --machine-type=e2-micro \
  --image-family=debian-12 --image-project=debian-cloud --tags=ETIQUETA
gcloud compute instances list                          # ver VMs
gcloud compute ssh NOMBRE --zone=ZONA                  # entrar a la VM
gcloud compute instances delete NOMBRE --zone=ZONA     # BORRAR (para no gastar)
```

## Sintaxis de comandos (recordatorio)
- Estructura gcloud: `gcloud SERVICIO RECURSO ACCION --opciones`
- `-` = una **opcion/ajuste** (NO "quitar"). Ej: `-p 80` = "puerto 80"; `-Pn` = "sin ping".
- `--` = opcion **larga** (palabra completa): `--zone`, `--action`.
- Opcion interruptor (on/off): `-Pn`, `-sS`. Opcion con valor: `-p 80`, `--zone=ZONA`.
- `tcp:80` = protocolo:puerto · `0.0.0.0/0` = "cualquier IP" · `/32` = "una sola IP".

## Tipos de escaneo nmap
| Flag | Que hace |
|---|---|
| `-sS` | SYN scan (half-open, estandar; necesita sudo) |
| `-sT` | TCP connect (sin sudo; mas ruidoso) |
| `-sV` | Detecta version del servicio |
| `-sC` | Corre scripts por defecto |
| `-sU` | Escanea puertos UDP |
| `-p-` | TODOS los 65.535 puertos |
| `-Pn` | No hacer ping (asumir host vivo) |
| `-oN archivo` | Guardar salida a archivo |
