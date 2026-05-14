# agenda.arc.share

Petit site statique servi par **GitHub Pages** qui permet aux liens de partage d'**Agenda ARC** d'ouvrir directement l'événement dans l'app Android quand elle est installée, ou de rediriger vers le Play Store sinon.

## Fichiers

- `index.html` — page d'accueil (landing) servie sur la racine.
- `404.html` — page de fallback servie par GitHub Pages pour toute URL non-existante. Détecte les chemins `/event/{id}`, tente d'ouvrir l'app via le scheme custom `agendaarc://`, et propose un bouton manuel d'installation si l'app ne répond pas.
- `.well-known/assetlinks.json` — déclaration **Android App Links** : permet à Android de vérifier que les URLs `https://applications-arc.github.io/agenda.arc.share/event/...` sont bien associées à l'app `app.agendaarc.agendaarc` et d'ouvrir l'app **directement, sans passer par le navigateur**, au tap d'un lien partagé.

## Mise en place (manuelle, une seule fois)

1. **Créer un repo public** sur le compte / l'organisation : `applications-arc/agenda.arc.share`.
2. Y pousser les 3 fichiers ci-dessus tels quels (la structure de dossier compte : `.well-known/assetlinks.json` doit être à ce chemin exact).
3. **Settings → Pages** sur le repo : *Source* = « Deploy from a branch », *Branch* = `main`, *Folder* = `/ (root)` → **Save**.
4. Attendre ~1 min que le premier déploiement soit terminé (l'icône Pages affiche l'URL).
5. Vérifier que ces URLs répondent bien :
   - `https://applications-arc.github.io/agenda.arc.share/` → page d'accueil
   - `https://applications-arc.github.io/agenda.arc.share/.well-known/assetlinks.json` → JSON valide

## Vérifier les App Links côté Android

Une fois l'APK installée, demander à Android de re-vérifier les App Links :

```bash
adb shell pm verify-app-links --re-verify app.agendaarc.agendaarc
adb shell pm get-app-links app.agendaarc.agendaarc
```

La sortie attendue après quelques secondes :

```
Domain verification state:
  applications-arc.github.io: verified
```

## Mise à jour du SHA-256 (signature release)

L'empreinte actuelle (`1B:E5:84:2D:...`) correspond au keystore **debug** (utilisé par les builds locaux `flutter build apk --release` car aucune signing config production n'est encore configurée).

Quand l'app sera signée avec un keystore release dédié pour publication Play Store, **régénérer l'empreinte** et l'ajouter au tableau `sha256_cert_fingerprints` (on peut en lister plusieurs simultanément, debug + release, sans casser la vérif).

```bash
keytool -list -v -keystore <chemin-keystore-release> -alias <alias>
```

## Évolutions

- **iOS Universal Links** : ajouter un fichier `apple-app-site-association` à la racine quand l'app sera publiée sur l'App Store, avec le team ID + bundle ID.
- **Domaine custom** (`agendaarc.fr` par exemple) : si vous achetez le domaine, le pointer vers ces Pages (Settings → Pages → Custom domain). Les URLs deviennent `https://agendaarc.fr/event/...` (plus court, plus pro).
