# Note à destination de la DSI — Notifications e-mail du widget « Gestion de projet » Grist

**Objet :** activer l'envoi d'e-mails de notification (ajout/modification de tâche) pour le widget Grist, **en restant intégralement dans le SI de la collectivité**.

---

## 1. Besoin

Le widget « Gestion de projet » (widget Grist personnalisé) doit **prévenir par e-mail** les agents concernés (rôles Responsable / Approbateur / Consulté / Informé) lorsqu'une tâche les concernant est **créée ou modifiée**.

Contrainte technique : **un widget Grist ne peut pas envoyer d'e-mail** (bac à sable navigateur), et Grist n'envoie pas de mail nativement sur changement de donnée. La voie officielle repose sur un **webhook Grist** + un **relais d'automatisation** qui envoie l'e-mail via le SMTP interne.

## 2. Architecture proposée (100 % interne)

```
Widget Gestion de projet
        │  (écrit une ligne par agent concerné)
        ▼
Table Grist  PM_Notifications   ──►  Webhook Grist (événement « Add »)
                                          │  POST interne
                                          ▼
                         Relais interne (n8n auto-hébergé OU Power Automate M365)
                                          │
                                          ▼
                          SMTP / Exchange interne  ──►  e-mail à l'agent (@grandlyon.fr)
```

**Aucune donnée ne sort du SI** : le relais et le serveur d'envoi sont internes.

## 3. Données transmises (minimisation)

À chaque notification, **uniquement** :
- l'**e-mail du destinataire** (l'agent concerné) ;
- le **libellé/message** de la tâche ;
- le **type d'événement** (création / modification) ;
- l'**identifiant** de la tâche.

Pas de pièce jointe, pas de contenu sensible, pas de donnée nominative au-delà de l'e-mail professionnel.

## 4. Demandes à la DSI

1. **Héberger le relais en interne** : soit **n8n auto-hébergé** sur l'infra de la collectivité, soit un **flux Power Automate** (si Microsoft 365 est déjà en place). *Un workflow prêt à l'emploi est fourni (fichiers `.json` + guide).*
2. **Autoriser le webhook côté Grist** : variable serveur **`ALLOWED_WEBHOOK_DOMAINS`** = le **domaine interne du relais** (ex. `n8n.intra.grandlyon.fr`). *(Aucune adresse externe.)*
3. **Donner accès au SMTP/Exchange interne** pour l'envoi, idéalement via un **compte d'envoi dédié** (ex. `notifications-projets@grandlyon.fr`).
4. **Flux réseau** : autoriser le flux **Grist → relais** (interne à interne).

## 5. Garanties sécurité / RGPD

- **Données dans le périmètre du SI** : relais + SMTP internes, **rien vers un service externe**.
- **Webhook restreint** : limité à la table dédiée `PM_Notifications`, événement `Add` uniquement.
- **Authentification du webhook** possible : ajout d'un **en-tête de sécurité** (secret partagé) vérifié par le relais, pour éviter tout appel non autorisé.
- **Traçabilité** : les exécutions/journaux du relais restent sous contrôle DSI.
- **Minimisation** : seuls les champs strictement nécessaires (cf. §3) sont transmis.

## 6. Effort / réutilisable

- Le **widget est déjà prêt** (il alimente `PM_Notifications` automatiquement) — rien à développer côté Grist.
- **Workflow d'envoi fourni** clé en main (n8n) + **guide pas-à-pas** (`EMAIL-NOTIFICATIONS.md`, `POWER-AUTOMATE-GUIDE.md`).
- Mise en place estimée : **quelques heures** côté DSI (déploiement relais + réglage webhook + compte SMTP).

---

*Documents joints : `EMAIL-NOTIFICATIONS.md` (procédure complète), workflows n8n prêts à importer (`n8n-grist-pm-email*.json`), guide Power Automate. Contact projet : [à compléter].*
