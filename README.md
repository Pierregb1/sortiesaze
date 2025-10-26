# Ideas Minimal (GitHub Pages + Firestore, no user tracking)

Une app ultra-simple : **Donner une idée** (Titre + Description) ou **Voir les idées** (liste de titres cliquables pour voir la description).  
100% statique pour GitHub Pages. Données stockées dans **Firestore**. **Aucune info d'auteur, aucune IP stockée dans Firestore**.

## 🚀 Mise en route rapide (5–10 min)

1) Crée un projet **Firebase** → ajoute une **app Web** et copie la **config** (apiKey, projectId, etc.).
2) **Firestore** → créer la base (Production).
3) **Règles Firestore** (sécurité + pas d’info auteur) :
```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    function hasOnlyKeys(keys) {
      return keys.hasOnly(['title','description','createdAt']);
    }
    function isCleanString(field, min, max) {
      return request.resource.data[field] is string
        && request.resource.data[field].size() >= min
        && request.resource.data[field].size() <= max;
    }
    match /ideas/{id} {
      allow read: if true;
      // Autoriser la création SANS auth : uniquement les champs autorisés
      allow create: if
        hasOnlyKeys(request.resource.data.keys())
        && isCleanString('title', 3, 120)
        && isCleanString('description', 10, 4000)
        && request.resource.data['createdAt'] is timestamp;
      allow update, delete: if false; // Pas de modif/suppression côté client
    }
  }
}
```
> Ces règles **n’enregistrent aucune info d’auteur** (pas d'UID, pas d'IP). Firebase peut logger l’IP en interne pour la sécurité, mais **aucune IP n’est écrite dans Firestore**.

4) Ouvre `index.html` et **remplace** la config Firebase :
   - Cherche `// TODO: Firebase config` et colle la tienne.

5) Déploie sur **GitHub Pages** :
   - Crée un repo → ajoute `index.html`, `README.md`, `favicon.svg` → push.
   - Settings → Pages → Deploy from a branch → `main` → `/ (root)`.
   - Ton site est en ligne 🎉

---

## 📦 Schéma des données
- Collection `ideas`
  - doc auto-ID
  - `title`: string (3–120)
  - `description`: string (10–4000)
  - `createdAt`: timestamp (serverTimestamp côté client)

---

## 🧼 Anti-spam basique (optionnel)
- Ajoute un délai min. entre deux envois via un **throttle** client.
- Active **reCAPTCHA Enterprise / Turnstile** si besoin (pas inclus ici pour rester minimal).

---

## 📄 Licence
MIT
