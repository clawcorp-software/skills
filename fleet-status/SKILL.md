---
name: fleet-status
description: "Use to get a complete status of all agents, their compliance scores, pending POKEs, and system health. Quick overview of the entire fleet."
user_invocable: true
kind: ours
---

# Fleet Status — Skill de Management

## Quand appeler ce skill (PROACTIF — pas seulement sur demande)

Appelle ce skill automatiquement dans ces situations :
- **Au boot** — toujours, avant toute action
- **Après chaque merge de PR** — vérifier que tous les workers sont synced
- **Tous les ~15 tours pendant un speedrun** — détecter drift/blockers en temps réel
- **Quand tu détectes un SELF-CHECK** (tu relis un fichier déjà lu = compression) — signal de drift
- **Quand tu reçois un POKE** — vérifier le contexte global avant d'exécuter
- **Après 2h de session sans fleet check** — routine de santé

---

## Process COMPLET (dans cet ordre)

### 1. POKEs en attente
Pour chaque worker : architecte, forge, backbone, pixel, watchdog, echo, manager
```bash
# Pour chaque agent:
test -f workers/{agent}/POKE.md && cat workers/{agent}/POKE.md | grep -E "Priority:|From:" || echo "CLEAR"
```
- CLEAR = aucun POKE actif
- PENDING P0/P1 = ordres en attente (noter qui a envoyé)

### 2. Chantier actif par worker (CRITIQUE — manquait avant)
Pour chaque worker, vérifier s'il a du travail non-PRé :
```bash
# Commits ahead de dev (travail non mergé)
git fetch origin dev 2>/dev/null
git log --oneline origin/{branche} ^origin/dev --no-merges 2>/dev/null | wc -l

# Uncommitted changes sur sa branche
git -C workers/{agent}/worksites/clawcorp status --short 2>/dev/null | wc -l
```
- 0 commits ahead + 0 uncommitted = IDLE (safe)
- N commits ahead = chantier en cours (PR pas encore ouverte)
- Uncommitted = travail non sauvegardé (risque de perte)

### 3. Date du dernier HANDOFF (fraîcheur de session)
```bash
# Lire la date du dernier handoff
head -3 workers/{agent}/brain/LATEST-HANDOFF.md 2>/dev/null | grep Date
```
- HANDOFF < 24h = session récente ✅
- HANDOFF 1-3 jours = stale, vérifier si actif
- HANDOFF > 3 jours = probablement idle / besoin de reboot

### 4. Activité Slack récente par worker
```bash
powershell.exe -ExecutionPolicy Bypass -File "C:/Users/user/clawcorp/tower/scripts/slack/slack-read.ps1" -Bot "manager" -Channel "#clawcorp" -Limit 30 2>/dev/null | grep -i "{agent}" | head -3
```
Chercher le pattern `[DONE:{agent}]`, `[DEMO:{agent}]`, `[WIP:{agent}]` pour dernière activité.

### 5. Serveurs (tous les ports)
```bash
for port in 3331 3332 3333 3334 3335 3336; do
  curl -s --max-time 2 http://localhost:$port/health 2>&1 | grep -q "Online" && echo "$port: UP" || echo "$port: DOWN"
done
```
Map ports: architecte=3331, manager=3332, forge=3333, backbone=3334, pixel=3335, echo=3336

### 6. PRs ouvertes + statut CI
```bash
gh pr list --state open --json number,title,headRefName,statusCheckRollup,mergeable 2>/dev/null
```
- 0 PRs = tout mergé ✅
- PR avec statusCheckRollup=[] = CI ne tourne pas ⚠️
- PR avec mergeable=CONFLICTING = conflit à résoudre 🔴

### 7. Token budget (via token-police)
```bash
cat "C:/Users/user/clawcorp/tower/config/budget-status.json" 2>/dev/null | python -c "import sys,json; d=json.load(sys.stdin); print(f'Status: {d[\"status\"]} | Usage: {d[\"currentUsagePct\"]}% | Lockdown: {d[\"lockdown\"]}')" 2>/dev/null || echo "budget-status not found"

# Coût réel (derniers 3 jours)
npx ccusage@latest daily --last 3 2>/dev/null | tail -5
```

### 8. Drift detection (via audit-anomaly)
```bash
# Lancer audit-anomaly en mode dry (lecture seule)
powershell.exe -ExecutionPolicy Bypass -File "C:/Users/user/clawcorp/tower/scripts/monitoring/audit-anomaly.ps1" -WindowMinutes 30 -DryRun 2>/dev/null | tail -10
```
- Aucune anomalie = OK ✅
- [DRIFT] = worker bloqué en boucle → POKE ou reset recommandé
- [WRITE-LOOP] = agent qui écrit en boucle → intervention urgente

---

## Format de sortie

```
╔══════════════════════════════════════════════════════════╗
║         FLEET STATUS — {DATE} {HEURE}                   ║
╚══════════════════════════════════════════════════════════╝

=== WORKERS ===

| Agent      | POKE    | Chantier        | HANDOFF  | Serveur | Statut        |
|------------|---------|-----------------|----------|---------|---------------|
| Tour       | CLEAR   | X commits ahead | 2h       | --      | ✅ ACTIF      |
| Architecte | CLEAR   | IDLE            | 1j       | 3331 ✅ | 💤 IDLE       |
| Forge      | PENDING | 3 commits ahead | 4h       | 3333 ✅ | 🔧 EN COURS   |
| Backbone   | CLEAR   | IDLE            | 3j       | 3334 ✅ | 💤 STALE      |
| Pixel      | CLEAR   | 2 commits ahead | session  | 3335 ✅ | 🔧 EN COURS   |
| Watchdog   | CLEAR   | IDLE            | 1j       | --      | 💤 IDLE       |
| Echo       | CLEAR   | IDLE            | 2j       | 3336 ✅ | 💤 IDLE       |
| Manager    | CLEAR   | PR ouverte      | maintenant | 3332 ✅ | ✅ ACTIF    |

=== PRs & CI ===
PRs ouvertes: X
CI status: OK / ⚠️ BLOQUE

=== BUDGET ===
Token police: GREEN/YELLOW/RED ({pct}% usage)
Coût aujourd'hui: $XX | Cette semaine: $XXX

=== DRIFT ===
Anomalies détectées: OK / [DRIFT] {agent} ...

=== RECOMMANDATIONS ===
🔴 ACTION REQUISE:
  - [agent] a X commits non-PRés depuis Xh → ouvrir PR ou POKE
  - [agent] POKE PENDING depuis Xh → vérifier exécution

🟡 À SURVEILLER:
  - [agent] HANDOFF > 3j + commits ahead → vérifier si session toujours active
  - CI ne tourne pas sur PR #{N} → investiguer (trigger manquant?)

🟢 INFO:
  - Budget: XX% sur plan
  - Tous serveurs UP
```

---

## Règles de recommandations

### POKE recommandé si :
- Worker a commits ahead de dev MAIS aucune PR ouverte depuis > 4h
- Worker a POKE PENDING mais n'a pas posté d'ACK dans #clawcorp dans l'heure
- Drift détecté (audit-anomaly) sur un worker actif

### Save-and-Sleep recommandé si :
- Worker actif depuis > 4h sans save (HANDOFF > 4h)
- Compaction détectée (SELF-CHECK signals dans Slack ou re-lecture répétée de fichiers)
- Après N compactions — signal à escalader au Keymaster : "Recommande save-and-sleep pour [agent], Keymaster approuve?"
- Budget token RED ou >80k tokens estimés pour la session

### Alerte urgente si :
- Zombie détecté (lockfile actif mais PID mort)
- POKE PENDING depuis > 2h sans ACK
- Commits uncommitted + serveur DOWN (travail non sauvegardé, risque de perte)
- Budget LOCKDOWN actif (bloquer les nouveaux boots)

---

## Notes d'implémentation

- **POKE CLEAR ≠ worker idle** : vérifier HANDOFF + commits ahead pour distinguer idle de "en cours sans POKE"
- **Pixel / Fork actif** : toujours vérifier `git log origin/{branche} ^origin/dev` — pas juste le POKE
- **CI silence** : `statusCheckRollup: []` = CI pas déclenché (problème infra, pas de failure)
- Ce skill est appelable par manager proactivement — pas besoin d'attendre le Keymaster
- Pendant un speedrun : appeler tous les 15 tours ou après chaque merge
