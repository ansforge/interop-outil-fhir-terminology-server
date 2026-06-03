This folder contains some basic starter configurations:

* Terminology server: see tx-config.json for a vanilla server that doesn't contain any licensed content
* NPM web server: see projector.json for a basic configuration to make a package available online

To use these, copy the relevant file to your local data directory, and rename to config.json

---

## Configuration ANS (ans.fr.terminologies)

Les fichiers `ans-config.json` et `ans-library.yaml` fournissent une configuration prête à l'emploi
pour déployer un serveur de terminologie FHIR avec les ressources françaises de l'ANS.

### Qu'est-ce que le package enrichi ?

Le package `ans.fr.terminologies` publié par l'ANS contient les ValueSets et CodeSystems du référentiel français.
Cependant, la plupart des CodeSystems y sont déclarés sans contenu (`"content": "not-present"`).

Le **package enrichi** (`ans.fr.terminologies-enriched`) est une version augmentée de ce package dans laquelle
le contenu complet de chaque CodeSystem a été téléchargé depuis le SMT (Système de Management des Terminologies
d'eSanté, `https://smt.esante.gouv.fr/fhir`). Cela permet au serveur de terminologie de répondre aux
opérations `$expand` et `$lookup` sur l'ensemble des codes ANS.

### Ce qui est inclus

- **`ans.fr.terminologies-enriched#1.10.0`** — package ANS enrichi avec le contenu complet des CodeSystems depuis le SMT
- **SNOMED CT édition française** — terminologie clinique en français
- **LOINC** — avec les libellés français de référence
- **HL7 terminology, IPS, IHE format codes** — terminologies internationales complémentaires

Les paramètres `internalLimit` et `externalLimit` sont fixés à 1 000 000 pour permettre l'expansion
des grands ValueSets ANS (certains contiennent plus de 3 000 codes).

### Terminologies enrichies depuis le SMT

Le package `ans.fr.terminologies#1.10.0` déclare les CodeSystems suivants en `"content": "not-present"`.
Le script `enrich-ans-package.js` télécharge leur contenu complet depuis le SMT :

| CodeSystem | URL | Concepts |
|---|---|---|
| TRE-R13-CommuneOM | https://mos.esante.gouv.fr/NOS/TRE_R13-CommuneOM/FHIR/TRE-R13-CommuneOM | 39 297 |
| ATC WHO | https://smt.esante.gouv.fr/terminologie-atc | 7 055 |
| BDPM (médicaments) | https://smt.esante.gouv.fr/terminologie-bdpm | 41 218 |
| CCAM | https://smt.esante.gouv.fr/terminologie-ccam | 38 223 |
| CIM-10 | https://smt.esante.gouv.fr/terminologie-cim-10 | 19 075 |
| CIM-11 MMS | https://smt.esante.gouv.fr/terminologie-cim11-mms | 36 480 |
| CISP | https://smt.esante.gouv.fr/terminologie-cisp | 1 434 |
| CLADIMED | https://smt.esante.gouv.fr/terminologie-cladimed | 4 717 |
| EMDN | https://smt.esante.gouv.fr/terminologie-emdn | 8 344 |
| NUVA | https://smt.esante.gouv.fr/terminologie-nuva | 1 942 |
| RUIM e-prescription | https://smt.esante.gouv.fr/terminologie-ruim-eeprescription | 22 897 |
| SMS | https://smt.esante.gouv.fr/terminologie-sms | 71 998 |
| Standard Terms (EDQM) | https://smt.esante.gouv.fr/terminologie-standardterms | 1 297 |

### Téléchargement automatique

Les fichiers terminologiques volumineux (SNOMED CT, LOINC, package ANS enrichi) sont hébergés sur le
serveur ANS à l'adresse `http://interop.esante.gouv.fr/tx/data`. FHIRsmith les télécharge
automatiquement au démarrage s'ils ne sont pas déjà présents dans le cache local.

**Aucune action manuelle n'est nécessaire** pour récupérer ces fichiers.

### Installation

```bash
# Cloner le dépôt et installer les dépendances
git clone https://github.com/ansforge/interop-outil-fhir-terminology-server
cd interop-outil-fhir-terminology-server
npm install

# Créer le répertoire de données
mkdir -p data

# Remplacer la configuration par défaut par la configuration ANS
cp configurations/ans-config.json data/config.json
cp configurations/ans-library.yaml data/library.yaml

# Démarrer le serveur — les fichiers terminologiques sont téléchargés automatiquement au premier lancement
npm run dev
```

Le serveur de terminologie est accessible à l'adresse `http://localhost:3000/tx/r4`.

