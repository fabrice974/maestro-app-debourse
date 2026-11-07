# maestro-app-debourse

Application de sous-processus **Déboursé** de la PoC Maestro, écrite en ASP.NET Core (.NET 10,
minimal API, HTML rendu côté serveur). Elle prouve qu'une équipe .NET se branche sur le backbone
([maestro-v2](https://github.com/fabrice974/maestro-v2)) avec le seul contrat OpenAPI, copié dans
`contrat/openapi.yaml`.

## Lancer

Prérequis : SDK .NET 10 et la passerelle du backbone démarrée sur http://localhost:4000
(`npm run passerelle` dans maestro-v2, ou tout l'écosystème : `npm run ecosysteme -- demarrer`).

```sh
dotnet run
```

L'application écoute sur http://localhost:4103 (utilisateur par défaut : `hugo`, à changer avec le
sélecteur « Agir en tant que »). Configuration dans `appsettings.json`, section `Debourse` (URL de la
passerelle, jeton `demo-jeton-debourse`, utilisateur par défaut), surchargeable par variables
d'environnement (`Debourse__UrlPasserelle`, `Debourse__Jeton`, …).

Écrans : boîte de tâches (`/`), tâche (`/taches/{id}`), dossiers en post-investissement
(`/dossiers`, événement `demande-tirage`), audit d'un dossier (`/dossiers/{id}/audit`).
Webhook du backbone : `POST /webhooks/taches` (exige `Authorization: Bearer demo-jeton-debourse`).

## Contrat et client généré

- `contrat/openapi.yaml` est une **copie figée** du contrat de maestro-v2 : l'équipe choisit quand
  elle adopte une nouvelle version. Mise à jour depuis maestro-v2 : `npm run ecosysteme -- contrat`
  (copie le contrat ici et régénère le client), puis revue et PR dans ce dépôt.
- Le client C# `Passerelle/PasserelleClient.g.cs` est généré par NSwag (`nswag.json`) et **commité** :
  la compilation ne demande ni NSwag ni accès réseau. Régénération manuelle :

```sh
dotnet tool restore
dotnet nswag run nswag.json
```

Choix de génération : `System.Text.Json` (aucun paquet NuGet), `HttpClient` injecté (l'adresse et le
jeton sont posés par `Program.cs`), types optionnels nullables. Les erreurs 403/409/422 arrivent en
`ApiException<Erreur>` ; une réponse non décrite pour une opération (ex. 403 sur le dépôt de pièce)
est relue depuis le corps brut.

## Pourquoi aucune dépendance Camunda

- L'application ne connaît que des dossiers, des tâches, des documents et des événements métier :
  c'est le contrat de la passerelle, pas le BPMN.
- La passerelle est le seul client Camunda ; elle applique sécurité, séparation des tâches et audit
  avant d'informer le moteur.
- Changer de moteur (ou de version du BPMN) ne touche donc pas cette application, tant que le contrat
  OpenAPI tient.
