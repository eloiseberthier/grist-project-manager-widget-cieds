# 📧 Recevoir les notifications par e-mail avec Power Automate (Microsoft 365)

Ce guide explique comment envoyer un e-mail automatique à chaque personne concernée par une tâche, en utilisant **Power Automate** — l'outil d'automatisation **inclus dans Microsoft 365** (souvent déjà disponible en entreprise).

> **Le principe :** le widget écrit une ligne dans la table **`PM_Notifications`** de votre document Grist pour chaque utilisateur concerné. Un **webhook Grist** envoie cette ligne à Power Automate, qui envoie l'e-mail. Le widget ne peut pas envoyer d'e-mail lui-même (sécurité du navigateur).

> ⚠️ **Prérequis :** le déclencheur « Lorsqu'une requête HTTP est reçue » est une fonctionnalité **Premium** de Power Automate (incluse dans certaines licences M365, ex. E5, ou via un module complémentaire). Vérifiez avec votre service informatique.

---

## Étape 1 — Créer le flux Power Automate

1. Allez sur **https://make.powerautomate.com** et connectez-vous avec votre compte professionnel.
2. **Créer** → **Flux de cloud instantané**.
3. Donnez un nom (ex. « Grist — e-mail notifications ») et choisissez le déclencheur **« Lorsqu'une requête HTTP est reçue »** (*When an HTTP request is received*). Cliquez **Créer**.

## Étape 2 — Définir le format des données reçues

1. Dans le déclencheur HTTP, cliquez **« Utiliser un exemple de charge utile pour générer le schéma »** (*Use sample payload to generate schema*).
2. Collez cet exemple, puis **Terminé** :

```json
[
  {
    "id": 1,
    "User_Email": "prenom.nom@exemple.fr",
    "Message": "Nouvelle tâche vous concernant : Préparer le budget",
    "Type": "task_created",
    "Task_Id": 12
  }
]
```

> Grist envoie un **tableau** (parfois plusieurs notifications d'un coup). On va donc parcourir chaque élément.

## Étape 3 — Parcourir les notifications et envoyer l'e-mail

1. **+ Nouvelle étape** → action **« Appliquer à chacun »** (*Apply to each*).
2. Dans « Sélectionner une sortie », choisissez **`body`** (le corps de la requête HTTP).
3. À l'intérieur du « Appliquer à chacun », **+ Ajouter une action** → **« Envoyer un e-mail (V2) »** (*Office 365 Outlook — Send an email V2*).
4. Renseignez :
   - **À (To)** : champ dynamique **`User_Email`**
   - **Objet (Subject)** : `Tâche — ` puis le champ dynamique **`Type`**
   - **Corps (Body)** : le champ dynamique **`Message`**
5. **Enregistrer**.

## Étape 4 — Récupérer l'URL du webhook

1. Rouvrez le déclencheur **« Lorsqu'une requête HTTP est reçue »**.
2. Après le premier enregistrement, une **« URL HTTP POST »** apparaît. **Copiez-la** (c'est l'adresse à donner à Grist).

## Étape 5 — Connecter Grist

1. Ouvrez votre document Grist → menu **⚙️ Paramètres du document** → section **Webhooks** → **Ajouter un webhook**.
2. Réglez :
   - **Table** : `PM_Notifications`
   - **Événement** : **Add** (à chaque nouvelle notification)
   - **URL** : collez l'URL HTTP de Power Automate
   - **Colonnes** (optionnel) : `User_Email`, `Message`, `Type`, `Task_Id`
3. **Enregistrez** le webhook.

## Étape 6 — Tester

Dans le widget, **créez ou modifiez une tâche** en y associant un utilisateur (avec un e-mail valide dans la fiche utilisateur). → Une ligne apparaît dans `PM_Notifications` → Grist appelle Power Automate → l'e-mail part. ✅

---

## Points de vigilance (à voir avec votre informatique)
- **Webhooks Grist** : doivent être autorisés sur votre instance/plan (Grist auto-hébergé : variable `GRIST_ALLOWED_WEBHOOK_DOMAINS`).
- **Licence Premium** pour le déclencheur HTTP de Power Automate.
- **Confidentialité (RGPD, secteur public)** : les données de notification transitent par Power Automate (service Microsoft) — généralement acceptable si vous êtes déjà sur M365, mais à valider.

*Alternative sans Power Automate Premium : utilisez n8n (workflows .json fournis dans l'onglet Paramètres du widget).*
