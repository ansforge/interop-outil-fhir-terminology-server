This folder contains some basic starter configurations:

* Terminology server: see tx-config.json for a vanilla server that doesn't contain any licensed content
* NPM web server: see projector.json for a basic configuration to make a package available online

To use these, copy the relevant file to your local data directory, and rename to config.json

---

## Configuration ANS

![ANS](https://user-images.githubusercontent.com/48218773/227532484-eff82649-4e42-49c6-966a-dc3ea78cf59c.png)

Les fichiers `ans-config.json` et `ans-library.yaml` fournissent une configuration prête à l'emploi
pour déployer un serveur de terminologie FHIR avec les ressources françaises de l'ANS.

### Contenu de ans-library.yaml

Le fichier `ans-library.yaml` définit les sources terminologiques chargées par le serveur :

| Source | Description |
|---|---|
| `ucum:tx/data/ucum-essence.xml` | Unités de mesure UCUM |
| `npm:hl7.terminology.r4#7.1.0` | Terminologies HL7 (v2, v3, FHIR core) |
| `npm/cs:hl7.fhir.uv.ips#1.1.0` | CodeSystems du profil IPS (International Patient Summary) |
| `npm/cs:ihe.formatcode.fhir` | Codes de format IHE (XDS, IUA...) |
| `url:…/ans.fr.terminologies-enriched-1.10.0.tgz` | Package ANS enrichi (voir ci-dessous) |
| `snomed:snomed-fr.cache` | SNOMED CT édition française |
| `loinc:loinc.db` | LOINC avec libellés français |

### Qu'est-ce que le package enrichi ?

Le package `ans.fr.terminologies` publié par l'ANS contient les ValueSets et CodeSystems du référentiel français.
Cependant, certains CodeSystems y sont déclarés sans contenu (`"content": "not-present"`).

Le **package enrichi** (`ans.fr.terminologies-enriched`) est une version augmentée de ce package dans laquelle
le contenu complet de chaque CodeSystem a été téléchargé depuis le SMT (Système de Management des Terminologies
d'eSanté, `https://smt.esante.gouv.fr/fhir`). Cela permet au serveur de terminologie de répondre aux
opérations `$expand` et `$lookup` sur l'ensemble des codes ANS.

Les CodeSystems enrichis dans la version 1.10.0 sont :

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

Les paramètres `internalLimit` et `externalLimit` sont fixés à 1 000 000 pour permettre l'expansion
des grands ValueSets ANS (certains contiennent plus de 3 000 codes).

### Téléchargement automatique

Les fichiers terminologiques volumineux (SNOMED CT, LOINC, package ANS enrichi) sont hébergés sur le
serveur ANS à l'adresse `http://interop.esante.gouv.fr/tx/data`. FHIRsmith les télécharge
automatiquement au démarrage s'ils ne sont pas déjà présents dans le cache local.

**Aucune action manuelle n'est nécessaire** pour récupérer ces fichiers.

### Installation

```bash
# Cloner le dépôt
git clone https://github.com/ansforge/interop-outil-fhir-terminology-server
cd interop-outil-fhir-terminology-server

# Installer les dépendances
npm install

# Créer les répertoires nécessaires
mkdir -p data data/logs

# Copier la configuration ANS
cp configurations/ans-config.json data/config.json
cp configurations/ans-library.yaml data/library.yaml
```

### Démarrer le serveur

**Mode développement** — rechargement automatique à chaque modification du code :
```bash
npm run dev
```

**Mode production** — démarrage stable sans rechargement automatique :
```bash
npm start
```

Au premier lancement, FHIRsmith télécharge automatiquement les fichiers terminologiques volumineux
(SNOMED CT, LOINC, package ANS enrichi) depuis `http://interop.esante.gouv.fr/tx/data`.
Ce téléchargement initial peut prendre quelques minutes.

Le serveur de terminologie est ensuite accessible à l'adresse `http://localhost:3000/tx/r4`.

### Déploiement via Docker

Une image Docker est automatiquement construite à chaque push sur la branche `add-artefacts-ans` :

```bash
# Créer le répertoire de données sur l'hôte
mkdir -p /var/lib/fhirsmith-ans

# Copier la configuration ANS
cp configurations/ans-config.json /var/lib/fhirsmith-ans/config.json
cp configurations/ans-library.yaml /var/lib/fhirsmith-ans/library.yaml

# Démarrer le conteneur
docker run -d --name fhirsmith-ans \
  -p 3000:3000 \
  -e FHIRSMITH_DATA_DIR=/app/data \
  -v /var/lib/fhirsmith-ans:/app/data \
  ghcr.io/ansforge/interop-outil-fhir-terminology-server:cibuild
```

Au premier démarrage, FHIRsmith télécharge automatiquement les fichiers terminologiques dans
`/var/lib/fhirsmith-ans/terminology-cache/`. Les démarrages suivants utilisent le cache local.
