# Analyse de risques EBIOS Risk Manager – Application GestLoc

**Étude de cas fictive** · Imane Chaourad · MSc Management de la Cybersécurité (ECE Paris)

Ce projet applique la méthode **EBIOS Risk Manager** de l'ANSSI à une application fictive de gestion locative et comptable, *GestLoc*, exploitée par la société fictive *ImmoGest SA* (250 salariés, 12 000 locataires). Le contexte s'inspire de mon expérience de développeuse sur des applications Java / Spring Boot manipulant des données financières. Aucune donnée réelle n'est utilisée.

## Résultats en bref

- **4 risques retenus**, dont **2 inacceptables** au départ : rançongiciel via le prestataire d'infogérance, et fraude au changement d'IBAN.
- **11 mesures** de traitement, chacune reliée à l'**annexe A de l'ISO/IEC 27001:2022**, pour un coût estimé d'environ 60 k€ la première année.
- Après traitement : 3 risques acceptables, 1 risque toléré sous surveillance.

| Risque | Niveau initial | Niveau résiduel |
|---|---|---|
| R1 – Rançongiciel via le prestataire | Inacceptable (G4/V3) | À traiter (G4/V2) |
| R2 – Fraude au changement d'IBAN | Inacceptable (G4/V3) | Acceptable (G4/V1) |
| R3 – Fuite via une dépendance vulnérable | À traiter (G4/V2) | Acceptable (G4/V1) |
| R4 – Abus de droits internes | À traiter (G3/V2) | Acceptable (G3/V1) |

## Contenu du dépôt

| Fichier | Description |
|---|---|
| [`rapport/Analyse_risques_EBIOS_RM_GestLoc.pdf`](rapport/Analyse_risques_EBIOS_RM_GestLoc.pdf) | Rapport complet (8 pages) : synthèse, 5 ateliers, plan de traitement |
| [`rapport/Analyse_risques_EBIOS_RM_GestLoc.md`](rapport/Analyse_risques_EBIOS_RM_GestLoc.md) | Même rapport, lisible directement sur GitHub |
| [`registre/Registre_risques_EBIOS_RM_GestLoc.xlsx`](registre/Registre_risques_EBIOS_RM_GestLoc.xlsx) | Registre des risques : niveaux et matrice calculés par formules, plan de traitement chiffré |

## Démarche

1. **Cadrage et socle** : 4 valeurs métier, 8 biens supports, 4 événements redoutés, écarts au socle ISO 27001 / RGPD / hygiène ANSSI.
2. **Sources de risque** : 6 couples source / objectif évalués, 4 retenus.
3. **Scénarios stratégiques** : cartographie de l'écosystème (le prestataire d'infogérance ressort en zone de danger), 4 scénarios.
4. **Scénarios opérationnels** : modes opératoires détaillés et cotation de la vraisemblance.
5. **Traitement** : mesures, responsables, échéances, coûts, risques résiduels et indicateurs de suivi.

## Ce que ce projet montre

- La maîtrise de la méthode EBIOS RM et du lien avec l'ISO 27001 (annexe A).
- La capacité à produire des livrables de conseil : synthèse pour la direction, registre chiffré, plan d'actions.
- Un regard technique sur les risques applicatifs (dépendances vulnérables, droits de la base de données, CI/CD), issu de mon expérience de développeuse.

## Références

- ANSSI – *La méthode EBIOS Risk Manager – Le guide*
- ISO/IEC 27001:2022 et ISO/IEC 27005:2022
- CNIL – *Guide de la sécurité des données personnelles*

---

*English summary: EBIOS Risk Manager (ANSSI) risk assessment of a fictitious property-management application. 4 risks, 11 treatment measures mapped to ISO/IEC 27001:2022 Annex A, residual risk assessment, and a formula-driven risk register.*
