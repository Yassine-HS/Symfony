# Cahier des charges — Projet Artounsi (Symfony)

**Version**: 1.0  • **Date**: 2026-01-06

---

## 1. Résumé exécutif 🎯
Le projet Artounsi est une plateforme web développée avec Symfony pour gérer utilisateurs, contenus (posts, commentaires), catalogue produits, panier et paiements. Ce cahier des charges vise à formaliser les fonctionnalités, les exigences non‑fonctionnelles, l'architecture technique, la sécurité, le déploiement, le plan de tests et le backlog priorisé pour le MVP.

**Objectifs immédiats**: sécuriser l'authentification, retirer secrets du dépôt, stabiliser roles/auth, mettre en place CI/CD et scans de sécurité.

---

## 2. Périmètre fonctionnel ✅
- Inscription / connexion (email + mot de passe).
- Gestion du profil utilisateur (photo, bio, préférences).
- CRUD posts, commentaires, likes, modération (ban).
- Catalogue produits, ajout au panier, checkout (paiement externe).
- Back‑office : gestion utilisateurs, produits, contenus.

---

## 3. Exigences fonctionnelles détaillées 🧩
- Authentification
  - Inscription avec validation email.
  - Connexion + possibilité 2FA (SMS via Twilio ou email).
  - Récupération / réinitialisation de mot de passe.
- Contenus
  - Créer/modifier/supprimer posts et commentaires.
  - Modération par rôle Admin (ban/unban).
- Produits & Panier
  - Affichage catalogue paginé, filtres par catégorie.
  - Gestion panier, checkout et confirmation de commande.

Chaque fonctionnalité doit avoir des critères d'acceptation (tests E2E) et exemples d'API (si applicables).

---

## 4. Exigences non‑fonctionnelles 🔧
- Performance: temps de réponse < 300ms sur pages standards, 50 RPS cible initial.
- Scalabilité: conteneurisation (Docker) et possibilité de déploiement en k8s.
- Disponibilité: SLA cible 99.5% en production.
- Sécurité: respecter OWASP Top 10, chiffrement des données sensibles, rotation des secrets.
- Accessibilité: WCAG AA.

---

## 5. Sécurité — Plan prioritaire 🔒
1. **Mots de passe**: migrer vers Symfony PasswordHasher (argon2id/bcrypt) et ré‑hacher les mots de passe sur prochaine connexion (lazy rehash). (Haute priorité)
2. **Secrets**: retirer `.env` du VCS (déjà fait), utiliser `secrets`/Vault/GH Secrets et créer `.env.example`. (Haute priorité)
3. **CSRF**: activer globalement `framework.csrf_protection: true` et vérifier tous les formulaires/endpoint de mutation. (Haute priorité)
4. **Compromised passwords**: activer `NotCompromisedPassword` ou intégration HaveIBeenPwned. (Haute priorité)
5. **Twilio & Webhooks**: sécuriser webhooks avec `Twilio RequestValidator`, vérifier rate limiting pour Verify. (Moyenne)
6. **Logging & Audit**: centraliser logs, rotation, et limiter accès aux fichiers `var/log`.
7. **SCA & SAST**: activer Dependabot + CodeQL / SonarQube dans pipeline CI.

---

## 6. Architecture & composants techniques 🏗️
- Backend: Symfony (PHP), Doctrine ORM.
- DB: MySQL/MariaDB.
- Services: Twilio (SMS/Verify), Mailer SMTP.
- Conteneurisation: Docker Compose (fichiers existants) → prévoir Dockerfile optimisé.
- Frontend: Webpack/Encore (assets/).

---

## 7. CI/CD & Déploiement 🚀
- Workflow CI: lint → tests unitaires → static analysis (PHPStan/Psalm) → SAST/SCA → build image → deploy staging → E2E → deploy prod.
- Secrets: GH Actions + GH Secrets / Vault.
- Monitoring: logs centralisés (ELK) + métriques (Prometheus/Grafana).

---

## 8. Tests & QA ✅
- Unitaires (PHPUnit) — couverture minimale 60% pour le coeur.
- Tests d’intégration DB pour endpoints critiques.
- E2E (Playwright/Cypress) pour parcours: inscription, login, checkout.
- Tests de sécurité: SAST, pentest périodique, tests d'injection/rate-limiting.

---

## 9. Planning estimatif (MVP) 📅
| Tâche | Priorité | Estimation (jours) |
|---|---:|---:|
| Migrer et ré-hasher mots de passe | Élevée | 3 |
| Retirer secrets & `.env.example` | Élevée | 0.5 |
| Activer CSRF global + audit | Élevée | 1 |
| Activer NotCompromisedPassword | Élevée | 1 |
| CI: PHPStan/PHPUnit/CodeQL | Moyenne | 2 |
| Rate-limiting endpoints auth | Moyenne | 1 |
| Sécuriser Twilio (webhooks, verify) | Moyenne | 1 |
| Tests E2E + Load test | Moyenne | 2 |
| Total (priorité sécurité + CI) | — | ~11.5 jours |

---

## 10. Critères d'acceptation 🏁
- Déploiement en staging OK et rollback testé.
- Toutes les tâches de sécurité critiques complétées et validées par tests.
- Pipeline CI vert sur branche principale.
- Liste d’issues créées pour les tâches secondaires.

---

## 11. Risques & mitigations ⚠️
- Perte d’accès lors de rotation secrets → mitigation: procédure pas‑à‑pas + backups.
- Régression login après migration hashing → mitigation: lazy rehash, tests automatisés.
- Exposition de clés Twilio → mitigation: rotation immédiate et suppressions dans repo.

---

## 12. Livrables & actions immédiates ✅
- `docs/cahier-des-charges.md` (ce document)
- Issues GitHub prioritaires (sécurité: hashing, CSRF, secrets)
- PRs: password migration, config CSRF, CI workflows

**Top 6 actions immédiates**:
- Migrer hashing passwords + tests (PR)
- Retirer secrets et ajouter `.env.example`
- Activer CSRF global et vérifier endpoints
- Activer NotCompromisedPassword
- Mettre en place Dependabot + CodeQL
- Ajouter rate-limiting sur endpoint d'auth

---

## 13. Annexes & références 🔗
- Fichiers à consulter: `config/packages/security.yaml`, `src/Entity/Allusers.php`, `src/Entity/AllusersProvider.php`, `config/services.yaml`, `.env.example`.

---

> Fait par l’équipe technique — prêt à être soumis en revue et transformé en issues/priorisé dans GitHub.

