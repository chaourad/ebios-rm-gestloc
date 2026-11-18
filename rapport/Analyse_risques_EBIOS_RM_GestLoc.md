---
title: "Analyse de risques EBIOS Risk Manager – Application GestLoc"
subtitle: "Étude de cas fictive – Imane Chaourad – MSc Management de la Cybersécurité (ECE Paris)"
date: "Version 1.0 – décembre 2026"
---

> **Avertissement.** L'entreprise *ImmoGest SA* et l'application *GestLoc* sont fictives. Le contexte s'inspire de mon expérience de développeuse sur des applications de gestion financière, mais aucune donnée, architecture ou information confidentielle d'un employeur réel n'est utilisée.

# Synthèse pour la direction

ImmoGest SA gère 12 000 locataires via l'application GestLoc, qui traite les encaissements, la comptabilité et les données personnelles des locataires. L'analyse, menée selon la méthode EBIOS Risk Manager de l'ANSSI, retient **4 risques**, dont **2 jugés inacceptables** en l'état :

| Risque | Gravité | Vraisemblance initiale | Niveau initial | Niveau résiduel |
|---|---|---|---|---|
| R1 – Rançongiciel via le prestataire d'infogérance | 4 – Critique | 3 – Très vraisemblable | **Inacceptable** | Toléré sous surveillance |
| R2 – Fraude au changement d'IBAN | 4 – Critique | 3 – Très vraisemblable | **Inacceptable** | Acceptable |
| R3 – Fuite de données via une dépendance vulnérable de l'API | 4 – Critique | 2 – Vraisemblable | À traiter | Acceptable |
| R4 – Abus de droits par un collaborateur | 3 – Grave | 2 – Vraisemblable | À traiter | Acceptable |

**Les 5 mesures prioritaires** (à lancer sous 3 mois) :

1. Authentification multifacteur (MFA) obligatoire pour tous les accès d'administration, y compris ceux du prestataire.
2. Sauvegardes hors ligne et immuables, avec un test de restauration trimestriel.
3. Double validation obligatoire de tout changement de coordonnées bancaires.
4. Analyse des dépendances (SCA) bloquante dans la chaîne CI/CD.
5. Revue trimestrielle des droits d'accès et procédure de retrait des droits lors des mobilités.

Le budget estimé est d'**environ 60 k€ la première année** (outils et prestations, voir le plan de traitement), pour une réduction de la vraisemblance des deux risques critiques d'au moins un niveau.

# 1. Méthode et échelles

EBIOS Risk Manager, la méthode d'analyse de risques de l'ANSSI, se déroule en 5 ateliers :

1. **Cadrage et socle de sécurité** : périmètre, valeurs métier, biens supports, événements redoutés, écarts au socle.
2. **Sources de risque** : qui pourrait attaquer, et dans quel but.
3. **Scénarios stratégiques** : chemins d'attaque de haut niveau, passant éventuellement par l'écosystème.
4. **Scénarios opérationnels** : modes opératoires techniques détaillés.
5. **Traitement du risque** : stratégie, mesures, risques résiduels.

**Échelle de gravité**

| Niveau | Libellé | Exemple d'impact pour ImmoGest |
|---|---|---|
| G1 | Mineure | Gêne interne, sans impact client |
| G2 | Significative | Retard de traitement de quelques heures, impact limité |
| G3 | Grave | Indisponibilité > 48 h, perte financière < 500 k€, plainte de clients |
| G4 | Critique | Fuite massive de données (sanction CNIL), perte > 500 k€, atteinte durable à la réputation |

**Échelle de vraisemblance**

| Niveau | Libellé | Signification |
|---|---|---|
| V1 | Peu vraisemblable | Scénario possible mais exigeant des moyens importants |
| V2 | Vraisemblable | Scénario réaliste pour un attaquant motivé |
| V3 | Très vraisemblable | Mode opératoire courant, observé sur des cibles similaires |
| V4 | Quasi certain | Scénario déjà observé ou très facile à réaliser |

# 2. Atelier 1 – Cadrage et socle de sécurité

## 2.1 Contexte et périmètre

ImmoGest SA est une société de gestion locative de 250 salariés. Son application **GestLoc** (Java / Spring Boot, Angular, PostgreSQL, hébergée sur AWS) couvre trois modules : **Débiteurs** (quittances, encaissements, relances), **Comptabilité** (écritures, clôtures, virements) et **Portail locataire** (consultation, paiement en ligne, mise à jour des coordonnées). L'infrastructure est administrée par un **prestataire d'infogérance**.

**Périmètre de l'étude** : l'application GestLoc, ses données, son infrastructure AWS, sa chaîne CI/CD et les accès des utilisateurs internes et du prestataire.

## 2.2 Valeurs métier

| ID | Valeur métier | Nature | Responsable |
|---|---|---|---|
| VM1 | Encaissement des loyers et relance des débiteurs | Processus | Directrice financière |
| VM2 | Production des écritures comptables et des virements | Processus | Responsable comptable |
| VM3 | Données personnelles des locataires (identité, IBAN, revenus) | Information | DPO |
| VM4 | Service en ligne aux locataires (portail) | Processus | Directeur relation client |

## 2.3 Biens supports

| ID | Bien support | Valeurs métier concernées |
|---|---|---|
| BS1 | API Spring Boot (exposée sur Internet) | VM1, VM2, VM3, VM4 |
| BS2 | Application Angular (portail et back-office) | VM1, VM4 |
| BS3 | Base de données PostgreSQL | VM1, VM2, VM3 |
| BS4 | Chaîne CI/CD (Jenkins, Git, Docker) | Toutes |
| BS5 | Comptes d'administration et annuaire (IAM) | Toutes |
| BS6 | Prestataire d'infogérance et son accès distant | Toutes |
| BS7 | Postes de travail des comptables | VM2 |
| BS8 | Sauvegardes | VM1, VM2, VM3 |

## 2.4 Événements redoutés

| ID | Événement redouté | Valeur métier | Impacts | Gravité |
|---|---|---|---|---|
| ER1 | Divulgation des données personnelles des locataires | VM3 | Sanction CNIL, notification sous 72 h, perte de confiance | **G4** |
| ER2 | Détournement de fonds (virements frauduleux) | VM2 | Perte financière directe, responsabilité envers les propriétaires | **G4** |
| ER3 | Indisponibilité de GestLoc pendant plus de 48 h | VM1, VM4 | Encaissements bloqués, clôture retardée, image | **G3** |
| ER4 | Altération des écritures comptables | VM2 | Comptes faux, risque de redressement, perte de confiance des commissaires aux comptes | **G3** |

## 2.5 Socle de sécurité et écarts

Référentiels retenus : **ISO/IEC 27001:2022 (annexe A)**, **RGPD**, **guide d'hygiène informatique de l'ANSSI**.

| Référentiel / mesure | État | Écart constaté |
|---|---|---|
| ISO 27001 A.8.5 – Authentification sécurisée | Partiel | Pas de MFA pour les administrateurs ni pour le prestataire |
| ISO 27001 A.5.18 – Droits d'accès | Partiel | Matrice RBAC en place, mais aucune revue périodique des droits |
| ISO 27001 A.8.13 – Sauvegarde | Partiel | Sauvegardes en ligne sur le même compte AWS, jamais testées |
| ISO 27001 A.8.15 – Journalisation | Partiel | Journaux applicatifs incomplets, pas de centralisation |
| ISO 27001 A.5.30 – Continuité TIC | Non conforme | PCA/PRA rédigé mais jamais testé |
| ISO 27001 A.8.8 – Vulnérabilités techniques | Partiel | SAST (SonarQube) en place, pas d'analyse des dépendances (SCA) |
| ISO 27001 A.5.20 – Accords avec les fournisseurs | Non conforme | Contrat d'infogérance sans clause de sécurité ni droit d'audit |
| RGPD art. 32 – Sécurité du traitement | Partiel | Chiffrement au repos activé, pas d'AIPD réalisée |

# 3. Atelier 2 – Sources de risque et objectifs visés

| ID | Source de risque | Objectif visé | Motivation | Ressources | Pertinence | Retenu |
|---|---|---|---|---|---|---|
| SR1 | Groupe cybercriminel (rançongiciel) | Extorsion par chiffrement et menace de publication | Forte | Importantes | Élevée | **Oui** |
| SR2 | Fraudeur (ingénierie sociale) | Détournement de virements | Forte | Modérées | Élevée | **Oui** |
| SR3 | Attaquant opportuniste | Vol et revente de données personnelles | Moyenne | Modérées | Moyenne | **Oui** |
| SR4 | Collaborateur malveillant ou mécontent | Vengeance, gain personnel | Faible à moyenne | Accès légitimes | Moyenne | **Oui** |
| SR5 | Concurrent | Espionnage commercial | Faible | Limitées | Faible | Non |
| SR6 | Hacktiviste | Atteinte à l'image | Faible | Limitées | Faible | Non |

Les erreurs et accidents (panne, mauvaise manipulation) ne sont pas traités comme des sources de risque dans EBIOS RM : ils sont couverts par le socle de sécurité (atelier 1).

# 4. Atelier 3 – Écosystème et scénarios stratégiques

## 4.1 Cartographie de l'écosystème

| Partie prenante | Dépendance | Pénétration (accès au SI) | Maturité cyber | Confiance | Menace | Zone |
|---|---|---|---|---|---|---|
| Prestataire d'infogérance | Forte | Forte (accès admin) | Faible | Moyenne | **Élevée** | **Danger** |
| Banque partenaire (fichiers SEPA) | Forte | Faible | Élevée | Élevée | Faible | Veille |
| Expert-comptable externe | Moyenne | Moyenne (accès lecture) | Moyenne | Élevée | Moyenne | Contrôle |
| Fournisseurs et prestataires payés | Faible | Nulle | Variable | Faible | Moyenne | Contrôle |
| Locataires (portail) | Faible | Faible | Faible | Faible | Moyenne | Contrôle |

Le prestataire d'infogérance est la partie prenante la plus critique : il cumule un accès d'administration et une maturité cyber faible.

## 4.2 Scénarios stratégiques

| ID | Scénario stratégique | Source → objectif | Chemin | Événement redouté | Gravité |
|---|---|---|---|---|---|
| SS1 | Un groupe cybercriminel compromet le prestataire, rebondit sur l'infrastructure GestLoc, exfiltre les données puis chiffre les serveurs et les sauvegardes | SR1 → extorsion | Via l'écosystème (prestataire) | ER3, ER1 | **G4** |
| SS2 | Un fraudeur se fait passer pour un fournisseur ou un locataire et obtient le changement d'un IBAN, puis encaisse des virements | SR2 → détournement | Via l'écosystème (fournisseurs, locataires) | ER2 | **G4** |
| SS3 | Un attaquant exploite une vulnérabilité connue d'une dépendance de l'API exposée et extrait la base des locataires | SR3 → revente de données | Direct | ER1 | **G4** |
| SS4 | Un collaborateur ayant conservé des droits excessifs après une mobilité exporte des données ou modifie des écritures | SR4 → vengeance / gain | Direct (interne) | ER4, ER1 | **G3** |

**Mesures sur l'écosystème** : clause de sécurité et droit d'audit dans le contrat d'infogérance, accès du prestataire via un bastion avec MFA, procédure de vérification des changements de coordonnées bancaires des fournisseurs.

# 5. Atelier 4 – Scénarios opérationnels

**SO1 (pour SS1) – Rançongiciel via le prestataire** · Vraisemblance **V3**

1. *Connaître* : recherche sur LinkedIn des administrateurs du prestataire.
2. *Rentrer* : courriel d'hameçonnage ciblé, vol des identifiants VPN (pas de MFA).
3. *Trouver* : connexion au compte AWS d'ImmoGest avec les droits d'administration du prestataire.
4. *Exploiter* : copie de la base PostgreSQL, suppression des sauvegardes en ligne, chiffrement des serveurs.

**SO2 (pour SS2) – Fraude au changement d'IBAN** · Vraisemblance **V3**

1. *Connaître* : identification d'un fournisseur réel (factures publiques, LinkedIn).
2. *Rentrer* : courriel usurpant le fournisseur, demandant la mise à jour de son IBAN.
3. *Exploiter* : modification de l'IBAN par un comptable, sans contre-appel ni double validation.
4. *Exploiter* : les virements suivants partent vers le compte du fraudeur.

**SO3 (pour SS3) – Exploitation d'une dépendance vulnérable** · Vraisemblance **V2**

1. *Connaître* : identification des technologies de l'API (en-têtes, messages d'erreur).
2. *Rentrer* : exploitation d'une CVE publique sur une bibliothèque non mise à jour (absence de SCA).
3. *Trouver* : accès à la base avec un compte applicatif disposant de droits trop larges.
4. *Exploiter* : extraction de la table des locataires.

**SO4 (pour SS4) – Abus de droits internes** · Vraisemblance **V2**

1. *Rentrer* : le collaborateur conserve ses droits « Comptabilité » après une mobilité (pas de revue des accès).
2. *Exploiter* : export massif de données ou modification d'écritures, sans alerte (journalisation incomplète).

# 6. Atelier 5 – Traitement du risque

## 6.1 Cartographie du risque initial

Seuils d'acceptation : **inacceptable** si G4 et V ≥ 3, ou G3 et V4 ; **à traiter** si G ≥ 3 et V = 2 ; **acceptable** sinon.

| Gravité \\ Vraisemblance | V1 | V2 | V3 | V4 |
|---|---|---|---|---|
| **G4** | | R3 | **R1, R2** | |
| **G3** | | R4 | | |
| **G2** | | | | |
| **G1** | | | | |

## 6.2 Plan de traitement

| Mesure | Risques couverts | Annexe A ISO 27001:2022 | Responsable | Échéance | Coût estimé |
|---|---|---|---|---|---|
| M1 – MFA pour tous les comptes d'administration et le prestataire | R1, R4 | A.8.5 | DSI | M+1 | 5 k€ |
| M2 – Bastion d'administration avec journalisation des sessions | R1 | A.8.2, A.8.15 | DSI | M+3 | 15 k€ |
| M3 – Sauvegardes hors ligne et immuables, test de restauration trimestriel | R1 | A.8.13 | DSI | M+2 | 8 k€/an |
| M4 – Clause de sécurité et droit d'audit dans le contrat d'infogérance | R1 | A.5.19, A.5.20 | Achats / RSSI | M+3 | – |
| M5 – Test annuel du PCA/PRA (exercice de crise rançongiciel) | R1 | A.5.30, A.5.24 | RSSI | M+6 | 10 k€ |
| M6 – Double validation et contre-appel pour tout changement d'IBAN | R2 | A.5.3 | Directrice financière | M+1 | – |
| M7 – Campagnes de sensibilisation à l'hameçonnage (comptables, admins) | R1, R2 | A.6.3 | RSSI | M+2 | 6 k€/an |
| M8 – Analyse des dépendances (SCA) bloquante en CI/CD, correctifs critiques sous 7 jours | R3 | A.8.8, A.8.25 | Responsable développement | M+1 | 3 k€/an |
| M9 – Compte applicatif de base de données au moindre privilège | R3 | A.8.2 | Responsable développement | M+2 | – |
| M10 – Revue trimestrielle des droits et retrait des droits lors des mobilités | R4 | A.5.15, A.5.18 | RSSI / RH | M+2 | – |
| M11 – Centralisation des journaux et alertes sur les exports massifs | R3, R4 | A.8.15, A.8.16, A.8.12 | RSSI | M+6 | 12 k€ |

## 6.3 Risques résiduels

| Risque | Niveau initial | Mesures | Vraisemblance résiduelle | Niveau résiduel | Décision |
|---|---|---|---|---|---|
| R1 | G4 / V3 – Inacceptable | M1, M2, M3, M4, M5, M7 | V2 | À traiter | Toléré sous surveillance, réévaluation dans 6 mois |
| R2 | G4 / V3 – Inacceptable | M6, M7 | V1 | Acceptable | Accepté |
| R3 | G4 / V2 – À traiter | M8, M9, M11 | V1 | Acceptable | Accepté |
| R4 | G3 / V2 – À traiter | M1, M10, M11 | V1 | Acceptable | Accepté |

La gravité ne change pas : les mesures réduisent la probabilité de l'attaque, pas l'impact si elle réussit. Pour réduire aussi la gravité de R1, une **cyber-assurance** (transfert du risque) peut être étudiée.

## 6.4 Suivi

- Indicateurs : taux de comptes admin avec MFA (cible 100 %), taux de restaurations réussies, délai de correction des vulnérabilités critiques, part des droits revus chaque trimestre.
- Revue de l'analyse : annuelle, ou après tout changement majeur (nouveau prestataire, nouveau module, incident).

# Annexe – Sources

- ANSSI, *La méthode EBIOS Risk Manager – Le guide* (cyber.gouv.fr).
- ANSSI, *Guide d'hygiène informatique*.
- ISO/IEC 27001:2022, annexe A ; ISO/IEC 27005:2022.
- CNIL, *Guide de la sécurité des données personnelles*.
