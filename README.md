# Odoo Custom Addons

Repository dedicata allo sviluppo di nuovi moduli e customizzazioni Odoo.

## Scopo

Addon custom pensati per estendere Odoo in modo:

- compatibile con Odoo.sh
- aderente alle best practice Odoo 19

## Contesto tecnico

- Piattaforma target: Odoo 19.0
- Runtime: Python 3.12+ e PostgreSQL 16+
- Deployment: Odoo.sh
- Strategia: estensione tramite ereditarieta dei modelli e delle viste

## Struttura Repository

La repository e predisposta con la seguente struttura:

- `.github/workflows/` - Contiene gli script automatici di verifica (non modificare).
- `.pylintrc`, `ruff.toml`, `.pre-commit-config.yaml` - Configurazioni dei controlli di qualita del codice (non modificare).
- `requirements.txt` - Installa automaticamente gli strumenti necessari in background.
- Directory principale - Area di lavoro del progetto. Crea qui i moduli Odoo (ad esempio `__manifest__.py`, `models/`, `views/`) direttamente nella cartella base della repository.

## Setup iniziale

Dopo aver clonato la repository:

```sh
pip install -r requirements.txt
pre-commit install
```

`pre-commit install` va eseguito una sola volta per clone e installa l'hook
`pre-commit`, che esegue i controlli di qualità sui file in stage a ogni
commit e blocca il commit se falliscono.

Senza questo comando il controllo non viene eseguito in locale.

## Processo di sviluppo consigliato

1. Creare branch dedicato dalla base corretta.
2. Implementare una singola modifica logica per commit.
3. Aggiornare sicurezza, viste e dati coerentemente alla modifica.
4. Aggiungere o aggiornare i test del modulo.
5. Verificare installazione/upgrade del modulo in ambiente Odoo.sh di test.
6. Aprire PR verso il branch di destinazione.

## Checklist minima prima di una PR

- Manifest aggiornato e coerente (`__manifest__.py`)
- Dipendenze dichiarate correttamente
- Accessi e regole sicurezza verificati
- Viste valide e senza override fragili
- Test eseguiti con esito positivo
- Nessuna modifica al core Odoo
