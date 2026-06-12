# 🔔📧 Notifications & e-mails — Guide complet

Comment recevoir une **notification dans l'application** ET, en option, un **e-mail** lorsqu'un utilisateur est concerné par l'**ajout ou la modification** d'une tâche.

> Ce guide vaut pour **toutes les instances Grist** : auto-hébergé (votre propre serveur) comme **grist.com** (hébergé).

---

## 🧭 Le principe en 30 secondes

1. Le **widget** écrit automatiquement une ligne dans la table **`PM_Notifications`** pour **chaque personne concernée** (rôles R / A / C / I) à la **création** et à la **modification** d'une tâche. → *Déjà actif, rien à faire.*
2. La **notification 🔔 dans l'app** apparaît immédiatement pour ces personnes. → *Déjà actif.*
3. Pour l'**e-mail** : un **webhook Grist** posé sur `PM_Notifications` envoie chaque nouvelle ligne à un **relais** (n8n ou Power Automate) qui, lui, **envoie l'e-mail**.

> ⚠️ **Un widget ne peut pas envoyer d'e-mail directement** (bac à sable du navigateur), et Grist non plus nativement. Le relais + webhook est **la voie officielle**.

| Étape | Qui le fait | Automatique ? |
|-------|-------------|---------------|
| Écrire la notification dans `PM_Notifications` | Le widget | ✅ Oui |
| Afficher l'alerte 🔔 dans l'app | Le widget | ✅ Oui |
| Autoriser les webhooks (serveur) | Admin / DSI | ❌ Une fois |
| Créer le webhook (document) | Vous | ❌ Une fois |
| Envoyer l'e-mail | Le relais (n8n / Power Automate) | ✅ après config |

---

## Étape 1 — Activer les webhooks côté Grist

### Vous êtes sur **grist.com** (hébergé)
Les webhooks sont disponibles (selon le plan/équipe). En général **rien à configurer côté serveur** : passez directement à l'étape 2.

### Vous êtes en **auto-hébergé** (votre serveur)
Par sécurité, Grist n'envoie de webhook **que vers des domaines autorisés**. Définissez la variable d'environnement du serveur :

```bash
# Autoriser un domaine précis (recommandé)
GRIST_ALLOWED_WEBHOOK_DOMAINS=mon-n8n.mon-domaine.fr

# ou tout autoriser (à éviter en production)
GRIST_ALLOWED_WEBHOOK_DOMAINS=*
```

Redémarrez Grist après modification. *(C'est généralement une action pour l'administrateur / la DSI.)*

---

## Étape 2 — Choisir et configurer le relais

Choisissez **une** des deux solutions ci-dessous.

### Option A — n8n *(gratuit, auto-hébergeable)*

Téléchargez le workflow correspondant à votre messagerie, puis dans n8n : **menu ⋮ (en haut à droite) → Import from File**.

| Messagerie | Fichier |
|------------|---------|
| **Gmail** (OAuth) | [n8n-grist-pm-email-gmail.json](n8n-grist-pm-email-gmail.json) |
| **Outlook / Microsoft 365** (OAuth) | [n8n-grist-pm-email-outlook.json](n8n-grist-pm-email-outlook.json) |
| **Zimbra & tout serveur SMTP** | [n8n-grist-pm-email.json](n8n-grist-pm-email.json) |

> Liens de téléchargement direct (clic droit → Enregistrer) :
> `https://raw.githubusercontent.com/isaytoo/grist-project-manager-widget/main/n8n-grist-pm-email-gmail.json`
> `https://raw.githubusercontent.com/isaytoo/grist-project-manager-widget/main/n8n-grist-pm-email-outlook.json`
> `https://raw.githubusercontent.com/isaytoo/grist-project-manager-widget/main/n8n-grist-pm-email.json`

Une fois importé (le Webhook, la normalisation des données et l'envoi d'e-mail sont **déjà reliés** et **déjà mappés** sur `User_Email`, `Type`, `Message`) :

1. Ouvrez le nœud **« Envoyer l'e-mail »** → associez vos **identifiants** (SMTP, ou OAuth Gmail/Outlook) et votre **adresse expéditeur**.
2. **Activez** le workflow.
3. Copiez l'**URL de production** du nœud **Webhook** (vous la collerez à l'étape 3).

### Option B — Power Automate *(Microsoft 365, rien à installer)*

Si votre organisation est déjà sur Microsoft 365, suivez le pas-à-pas : **[POWER-AUTOMATE-GUIDE.md](POWER-AUTOMATE-GUIDE.md)**.
Vous obtiendrez une **URL HTTP** à coller à l'étape 3.
*(Le déclencheur HTTP de Power Automate est une fonction **Premium** — vérifiez votre licence.)*

---

## Étape 3 — Créer le webhook dans le document Grist

1. Ouvrez votre document → menu **⚙️ Paramètres du document** → section **Webhooks** → **Ajouter un webhook**.
2. Réglez :
   - **Table** : `PM_Notifications`
   - **Événement** : **Add** *(à chaque nouvelle notification)*
   - **URL** : collez l'URL obtenue à l'étape 2 (n8n ou Power Automate)
   - **Colonnes** (optionnel) : `User_Email`, `Message`, `Type`, `Task_Id`
3. Enregistrez.

---

## Étape 4 — Tester

Dans le widget, **créez ou modifiez une tâche** en y associant un utilisateur **dont la fiche contient un e-mail valide**.
→ Une ligne apparaît dans `PM_Notifications` → Grist appelle le relais → l'e-mail part. ✅

---

## 🛠️ Dépannage

| Symptôme | Piste |
|----------|-------|
| Aucune ligne dans `PM_Notifications` | Vérifiez que le toggle **« Notifier les utilisateurs concernés »** est activé (widget → Paramètres → Notifications & e-mail) et que la personne a bien un rôle R/A/C/I sur la tâche. |
| La ligne est créée mais pas d'e-mail | Le webhook n'est pas créé, l'URL est fausse, ou (auto-hébergé) le domaine n'est pas dans `GRIST_ALLOWED_WEBHOOK_DOMAINS`. |
| n8n reçoit mais n'envoie pas | Identifiants SMTP/OAuth manquants, ou le workflow n'est pas **activé**. |
| Les données arrivent sous `body[0]` | Grist envoie un tableau ; les workflows fournis gèrent déjà ce cas (nœud « Normaliser »). |

## 🔒 Points de vigilance (organisations)

- **RGPD / secteur public** : les notifications transitent par le relais. Préférez un **relais interne** (n8n auto-hébergé, ou Power Automate de votre tenant M365) plutôt qu'un SaaS externe, et validez avec votre DSI.
- **Envoi d'e-mail** : votre informatique doit parfois autoriser un compte d'envoi / un « mot de passe d'application ».
- L'auteur d'une action **n'est pas notifié** de sa propre saisie (anti-spam).

---

*Le widget se charge de la partie « notification ». Le webhook + relais (config unique) se charge de l'e-mail. Aucune limite Grist n'est contournée : c'est le fonctionnement officiel.*
