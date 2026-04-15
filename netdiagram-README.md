# NetDiagram — Guide de déploiement SaaS

## Structure des fichiers

```
netdiagram/
├── index.html              ← Landing page marketing
├── schema-reseau-v2.html   ← Application éditeur
└── README.md               ← Ce fichier
```

---

## Déploiement en 10 minutes sur Vercel (gratuit)

### Étape 1 — Créer un compte GitHub
1. Allez sur https://github.com
2. Créez un compte gratuit
3. Cliquez sur **New repository**
4. Nommez-le `netdiagram`
5. Cochez **Public**
6. Cliquez **Create repository**

### Étape 2 — Uploader vos fichiers
1. Dans votre nouveau repository, cliquez **uploading an existing file**
2. Glissez-déposez `index.html` et `schema-reseau-v2.html`
3. Cliquez **Commit changes**

### Étape 3 — Déployer sur Vercel
1. Allez sur https://vercel.com
2. Cliquez **Sign up** → **Continue with GitHub**
3. Cliquez **New Project**
4. Sélectionnez votre repository `netdiagram`
5. Cliquez **Deploy** — votre site est en ligne sur `netdiagram.vercel.app`

### Étape 4 — Domaine personnalisé (optionnel)
1. Dans Vercel → Settings → Domains
2. Ajoutez votre domaine (ex: `netdiagram.fr`)
3. Suivez les instructions DNS chez votre registrar

---

## Ajouter l'authentification réelle (Supabase — gratuit)

### 1. Créer un projet Supabase
Allez sur https://supabase.com → New project. Notez votre URL et anon key.

### 2. Remplacer dans index.html (avant </head>)
```html
<script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2"></script>
<script>
const { createClient } = supabase;
const sb = createClient('VOTRE_URL', 'VOTRE_ANON_KEY');

async function handleSignup() {
  const email = document.querySelector('#modal-form-signup input[type=email]').value;
  const password = document.querySelector('#modal-form-signup input[type=password]').value;
  const { error } = await sb.auth.signUp({ email, password });
  if (error) alert(error.message);
  else { closeModal(); window.open('schema-reseau-v2.html', '_blank'); }
}

async function handleLogin() {
  const email = document.querySelector('#modal-form-login input[type=email]').value;
  const password = document.querySelector('#modal-form-login input[type=password]').value;
  const { error } = await sb.auth.signInWithPassword({ email, password });
  if (error) alert(error.message);
  else { closeModal(); window.open('schema-reseau-v2.html', '_blank'); }
}
</script>
```

### 3. Table Supabase pour sauvegarder les schémas
```sql
create table schemas (
  id uuid default gen_random_uuid() primary key,
  user_id uuid references auth.users not null,
  title text,
  data jsonb,
  created_at timestamp with time zone default now()
);
alter table schemas enable row level security;
create policy "Users can manage own schemas" on schemas
  for all using (auth.uid() = user_id);
```

---

## Ajouter les paiements (Stripe)

1. Créez un compte sur https://stripe.com
2. Créez deux produits : Pro (12€/mois) et Équipe (29€/mois)
3. Intégrez Stripe Checkout :

```javascript
async function startCheckout(plan) {
  const stripe = Stripe('VOTRE_CLE_PUBLIQUE');
  await stripe.redirectToCheckout({
    lineItems: [{ price: plan === 'pro' ? 'price_PRO_ID' : 'price_TEAM_ID', quantity: 1 }],
    mode: 'subscription',
    successUrl: window.location.origin + '/schema-reseau-v2.html',
    cancelUrl: window.location.origin,
  });
}
```

---

## Personnalisation rapide

### Couleurs (dans :root de index.html)
- `--acc: #d4f53c` → couleur accent
- `--bg: #0a0a09` → fond
- `--tx: #f0efe8` → texte

### Nom du produit
Rechercher/remplacer "NetDiagram" dans index.html

### Tarifs
Modifier les valeurs 12€ et 29€ dans la section #pricing

---

## Checklist avant mise en ligne

- [ ] Remplacer les textes placeholder (CGU, confidentialité, contact)
- [ ] Vérifier que schema-reseau-v2.html est dans le même dossier
- [ ] Tester le formulaire sur mobile
- [ ] Configurer le domaine personnalisé sur Vercel
- [ ] Ajouter Google Analytics si souhaité
- [ ] Connecter Supabase pour les vrais comptes utilisateurs
- [ ] Connecter Stripe pour les paiements
