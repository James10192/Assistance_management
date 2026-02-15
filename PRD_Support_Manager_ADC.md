# PRD - Support Manager
## Product Requirements Document

**Version:** 2.0
**Date:** 15 Février 2026
**Owner:** Marcel Djedje-li
**Status:** Approuvé

---

## Changelog v2.0

- Stack technique tranchée (Next.js 15 + Prisma + Better-Auth)
- Modèle de données corrigé (ajout `TicketHistory`, `ClientContact`, `TicketCounter`)
- MVP resserré : signature électronique déplacée en Phase 2
- Portail client déplacé en Phase 3
- Questions ouvertes résolues
- Validations et règles de numérotation tickets précisées

---

## 1. VISION DU PRODUIT

### 1.1 Problème
Les équipes de développement perdent un temps considérable sur :
- La rédaction manuelle de rapports d'assistance technique
- La création de mails formatés pour annoncer les nouvelles fonctionnalités
- La gestion papier des rapports signés par les clients
- Le suivi dispersé des bugs et demandes entre différents produits
- La communication non standardisée avec les clients

### 1.2 Solution
**Support Manager** est une plateforme centralisée de gestion de support technique et communication client qui :
- Centralise tous les tickets (bugs, demandes, assistances) multi-produits
- Génère automatiquement des rapports d'assistance professionnels PDF
- Crée des release notes visuelles avec screenshots et guides
- Automatise l'envoi de communications personnalisées aux clients
- Stocke l'historique complet de toutes les interactions

### 1.3 Objectifs Business
- Réduire de 70% le temps de création de rapports d'assistance
- Éliminer complètement la gestion papier des rapports
- Améliorer la satisfaction client avec des communications professionnelles et visuelles
- Supporter la croissance avec scalabilité multi-produits
- Créer un actif réutilisable pour tous les futurs produits ADC

---

## 2. UTILISATEURS CIBLES

### 2.1 Personas Primaires

#### Développeur/Technicien Support (utilisateur quotidien)
- **Besoins:** Résoudre rapidement les tickets, générer des rapports sans effort
- **Pain points:** Rédaction manuelle, duplication d'informations
- **Fréquence d'usage:** Quotidienne (5-20 tickets/semaine)

#### Chef de Projet/Manager (validateur)
- **Besoins:** Valider les communications avant envoi, suivre les métriques
- **Pain points:** Manque de visibilité, processus de validation lourd
- **Fréquence d'usage:** Quotidienne (validation, reporting)

#### Client (École/Organisation) — Phase 3+
- **Besoins:** Signaler des bugs facilement, être informé des résolutions
- **Pain points:** Communication dispersée, manque de suivi
- **Fréquence d'usage:** Occasionnelle (signalement), passive (réception mails)

---

## 3. FONCTIONNALITÉS CORE

### 3.1 Gestion Multi-Produits

#### Entité "Produit"
**Champs:**
- Nom du produit, description, logo
- Code produit (ex: `KLASS-C`, `KLASS-U`) — utilisé dans la numérotation tickets
- Versions actives
- Clients associés
- Équipe assignée (développeurs autorisés)

**Règles métier:**
- Un produit peut avoir plusieurs versions simultanées
- Les tickets sont liés à un produit ET une version spécifique
- Les clients peuvent avoir accès à plusieurs produits

### 3.2 Système de Tickets Unifié

#### Types de Tickets
1. **Bug Report** (`BUG`) — signalé par client ou équipe interne
2. **Assistance Technique** (`INT`) — intervention sur site ou distance
3. **Demande de Fonctionnalité** (`FEAT`) — nouvelle feature demandée
4. **Release** (`REL`) — fonctionnalité développée à annoncer

#### Workflow de Ticket

```
[NOUVEAU] → [ASSIGNÉ] → [EN COURS] → [RÉSOLU] → [VALIDÉ] → [CLÔTURÉ]
```

Chaque transition est loggée dans `TicketHistory` avec : qui, quand, note optionnelle.

#### Numérotation Tickets

Format : `{PRODUCT_CODE}-{TYPE_CODE}-{YYYY}-{SEQ:04d}`

Exemples :
- `KLASS-C-BUG-2026-0001`
- `KLASS-U-INT-2026-0003`
- `KLASS-C-FEAT-2026-0012`

**Règles :**
- Séquence atomique par combinaison (product_code, type_code, year)
- Repart à 0001 chaque 1er janvier
- Générée via la table `TicketCounter` pour éviter les race conditions

#### Champs du Ticket

**Informations de base:**
- `ticketNumber` (unique, auto-généré)
- `type` : bug | assistance | feature | release
- `productId` → Product
- `productVersionId` → ProductVersion
- `clientId` → Client
- `priority` : low | medium | high | critical
- `status` : open | assigned | in_progress | resolved | validated | closed
- `createdAt`, `updatedAt`, `closedAt`
- `dueAt` (date de résolution souhaitée)

**Détails techniques:**
- `title` (court, max 200 chars)
- `description` (détaillée)
- `reproductionSteps` (JSON, pour bugs)
- `environmentInfo` (JSON : navigateur, OS, version)
- `affectedModule` (section du produit)

**Assignation:**
- `createdByUserId` → User (nullable si créé par ClientContact via portail)
- `createdByClientContactId` → ClientContact (nullable si créé par équipe interne)
- `assignedToUserId` → User
- `estimatedHours`, `actualHours`

**Résolution:**
- `resolutionDescription`
- `resolutionDate`
- `codeChanges` (description des changements)
- `testingNotes`

### 3.3 Rapports d'Assistance Technique

#### Génération Automatique
Quand un ticket de type `assistance` passe au statut `resolved` : bouton "Générer rapport" disponible.

#### Contenu du Rapport PDF

**En-tête :** Logo ADC, nom du produit, numéro de rapport, date, référence ticket

**Informations Client :** Organisation, contact, adresse, produit/version

**Détails de l'Intervention :**
- Problème signalé (description client)
- Diagnostic effectué
- Actions réalisées (bullet points)
- Résultat obtenu
- Recommandations

**Temps :** Heure début/fin, durée totale, technicien(s)

**Validation :** Section signature (Phase 2)

**Pied de page :** Coordonnées ADC

#### Signature Électronique (Phase 2, pas MVP)
- Lien de signature envoyé par mail au client
- Interface de signature canvas (tactile mobile-friendly)
- Valeur légale : signature simple avec preuve d'email (suffisant pour usage ADC Côte d'Ivoire)
- PDF final regénéré avec signature intégrée et horodatage

#### Archivage
- Numérotation séquentielle annuelle : `RPT-2026-0001`
- Stockage R2 (Cloudflare)
- Recherche par client, date, produit, technicien
- Export ZIP

### 3.4 Release Notes & Nouvelles Fonctionnalités

#### Workflow
```
[Tickets FEAT/BUG résolus] → [Grouper en Release] → [Rédiger Release Note] → [Validation Manager] → [Envoi Clients]
```

#### Éditeur de Release Notes
- Sélection des tickets résolus à inclure
- Pour chaque feature :
  - Titre marketing (non technique)
  - Description bénéfice client
  - Screenshots annotés (upload → R2)
  - Guide étape par étape (JSON array)
  - Lien vidéo optionnel
  - Badge : `new` | `improved` | `fixed`

#### Circuit de Validation (obligatoire avant envoi)
1. Developer crée la release note
2. Manager reçoit notification
3. Manager approuve ou rejette avec commentaire
4. **SLA validation : 48h** — si dépassé, relance automatique par mail
5. Si toujours pas de réponse après 72h : Admin peut auto-approuver
6. Envoi aux clients uniquement après approbation

### 3.5 Communication Client

#### Déclencheurs Automatiques
- Ticket créé → Accusé réception au client
- Ticket assigné → "Pris en charge"
- Ticket résolu → Notification + demande retour
- Rapport généré → Mail avec lien (+ lien signature en Phase 2)
- Release approuvée → Mail release notes

#### Templates Resend + React Email
Types : `bug_resolved`, `assistance_report`, `feature_release`, `release_major`, `release_minor`, `hotfix`, `maintenance`

Variables : `{client_name}`, `{product_name}`, `{version}`, `{ticket_number}`, `{solution}`, `{date}`, etc.

#### Tracking
- Ouverture mail (webhook Resend)
- Clics sur liens
- Stocké dans `EmailLog`

### 3.6 Historique et Recherche

#### Vue Client 360°
Par client, une page affiche :
- Tous les tickets (filtrables)
- Timeline des interactions (via `TicketHistory`)
- Rapports d'assistance
- Releases qui les concernent
- Notes internes (non visibles par le client)

#### Recherche
- Full-text sur tickets, rapports, commentaires (PostgreSQL `tsvector`)
- Filtres : produit, client, type, statut, date, assigné, priorité
- Export CSV/Excel

### 3.7 Métriques (simplifiées en Phase 1)

**Phase 1 — Dashboard simple :**
- Compteurs : tickets ouverts par statut, tickets en retard (>48h sans update)
- Mes tickets assignés
- Activité récente

**Phase 4 — Analytics avancées :**
- Temps moyen résolution par type/produit
- Charge par développeur
- Tendances (bugs par module)
- Rapports exportables

---

## 4. SPÉCIFICATIONS TECHNIQUES

### 4.1 Stack Technique (DÉFINITIVE)

```
Framework :       Next.js 15 (App Router, TypeScript strict)
Base de données : PostgreSQL (Neon pour serverless)
ORM :             Prisma 6.x
Auth interne :    Better-Auth + plugins organization + teams
Auth clients :    Better-Auth portail séparé (Phase 3)
Forms :           React Hook Form v7 + Zod v4
State serveur :   TanStack Query v5 + HydrationBoundary pattern
State client :    Zustand v5
URL state :       nuqs
UI :              shadcn/ui + Tailwind CSS v4
Email :           Resend + React Email
Fichiers :        Cloudflare R2 + Presigned URLs
PDF :             @react-pdf/renderer
Toasts :          Sonner
Tables :          TanStack Table v8 (shadcn DataTable)
Package manager : pnpm
Hébergement :     Vercel (frontend + API) + Neon (PostgreSQL) + R2 (fichiers)
```

### 4.2 Modèle de Données (Complet et Corrigé)

Les modèles clés ajoutés en v2.0 :
- `TicketHistory` — audit log obligatoire de chaque transition de statut
- `ClientContact` — contacts clients avec auth séparée (portail Phase 3)
- `TicketCounter` — numérotation atomique sans race condition

Tous les modèles complets sont définis dans `prisma/schema.prisma`.

### 4.3 APIs — Server Actions

Toutes les opérations passent par des **Server Actions** Next.js sauf :
- Les webhooks Resend → Route Handler `POST /api/webhooks/resend`
- L'endpoint de signature client → Route Handler `POST /api/sign/[token]` (Phase 2)
- Le widget embeddable → Route Handler `POST /api/widget/ticket` (Phase 3)

### 4.4 Sécurité et Permissions

#### Rôles
- `admin` — accès complet, gestion utilisateurs
- `manager` — validation communications, analytics
- `developer` — tickets de ses produits assignés, génération rapports
- `support` — tickets assignés seulement (lecture + commentaires)

#### RBAC
- Developer : voit uniquement les tickets des produits qui lui sont assignés
- Manager : voit tous les tickets
- Admin : tout

#### Rate Limiting
- Widget public (Phase 3) : 5 soumissions/IP/heure
- Endpoint signature (Phase 2) : 10 tentatives/token

---

## 5. ROADMAP RÉVISÉE

### Phase 1 : MVP (6-8 semaines) — Objectif : éliminer le papier

**Périmètre strict :**
- Auth interne (Better-Auth) : Admin + Developer
- CRUD tickets : types Bug + Assistance, workflow complet 6 statuts
- Audit log `TicketHistory` sur chaque transition
- Génération PDF rapports d'assistance (`@react-pdf/renderer`)
- Téléchargement PDF + stockage R2
- Envoi mail simple via Resend
- Dashboard simple : compteurs par statut, mes tickets

**Hors scope Phase 1 :**
- Signature électronique
- Templates mail avancés
- Analytics détaillées
- Portail client
- Widget embeddable

**Success metrics :**
- 100% des rapports d'assistance dématérialisés
- Rapport généré en < 5 minutes
- Adoption 100% équipe dev

### Phase 2 : Communication (4-6 semaines)

- Templates React Email pour chaque type d'événement
- Workflow validation Manager (SLA 48h, escalade Admin à 72h)
- Signature électronique (canvas + horodatage)
- Notifications automatiques email sur changement statut
- Support multi-produits (KLASSCI University)
- Rôle Manager

### Phase 3 : Release Management + Portail Client (4-6 semaines)

- Éditeur release notes (groupement tickets → release)
- Upload screenshots → R2
- Templates mail visuels release
- Portail client basique (ClientContact auth)
- Widget embeddable (Phase 3)

### Phase 4 : Scale & Analytics (ongoing)

- Analytics avancées + exports
- Base de connaissances auto-générée
- Recherche full-text PostgreSQL
- API publique + webhooks Slack/Discord
- Support multi-langue

---

## 6. QUESTIONS RÉSOLUES (v2.0)

| Question | Décision |
|----------|----------|
| Hébergement | **Vercel + Neon (PostgreSQL) + Cloudflare R2** |
| Stack frontend | **Next.js 15 App Router + TypeScript** |
| Auth | **Better-Auth** (self-hosted, Prisma-native) |
| Clients modifient templates | **Non** — gérés uniquement par l'équipe ADC |
| Intégration comptabilité | **Non** — hors scope |
| Langues | **Français uniquement** pour phases 1-3 |
| SLA validation | **48h** avec relance auto, escalade Admin à 72h |
| Mobile app native | **Non** — web responsive (PWA si besoin futur) |
| Valeur légale signature | **Signature simple** avec preuve d'email (suffisant pour CI) |

---

## 7. RISQUES & MITIGATIONS

| Risque | Impact | Probabilité | Mitigation |
|--------|--------|-------------|------------|
| Faible adoption équipe | Élevé | Moyenne | Formation, champions internes |
| Clients refusent signature électronique | Moyen | Faible | Option téléchargement PDF sans signature |
| Surcharge Manager pour validations | Moyen | Moyenne | SLA 48h + escalade Admin automatique à 72h |
| Race condition numérotation tickets | Faible | Faible | Table `TicketCounter` avec `upsert` atomique |
| Données clients sensibles | Élevé | Faible | R2 privé, URLs signées, RGPD conforme |
| Volume tickets explose | Moyen | Moyenne | Pagination serveur, index DB sur status+productId |

---

## 8. CRITÈRES DE SUCCÈS

### KPIs Phase 1
- Temps création rapport < 5 minutes (vs 30 min actuellement)
- 100% des rapports d'assistance dématérialisés
- Adoption 100% équipe dev

### KPIs Phase 2+
- Taux signature électronique > 90%
- Taux ouverture mails > 60%
- Temps moyen réponse < 24h critique, < 72h normal
- NPS client > 8/10

---

## 9. ANNEXES

### Glossaire
- **Ticket :** Unité de travail (bug, assistance, feature, release)
- **TicketHistory :** Audit log de chaque transition de statut
- **TicketCounter :** Générateur de numéros séquentiels atomiques
- **ClientContact :** Contact d'une organisation cliente (auth séparée Phase 3)
- **Rapport d'assistance :** PDF généré après intervention technique
- **Release Note :** Communication visuelle de nouvelles fonctionnalités

### Références
- Stack technique complète : `CLAUDE.md` (racine du projet)
- Standards PDF/A pour archivage légal
- RGPD compliance pour données clients

---

**Prochaines étapes :**
1. Sprint 0 : Setup infrastructure (Next.js 15 + Prisma + Better-Auth + R2)
2. `/prd-to-tasks Phase 1 MVP` pour décomposer en tâches concrètes
3. `/feature Système de Tickets` pour commencer l'implémentation

---

*Document vivant — v2.0 approuvée le 15/02/2026*
