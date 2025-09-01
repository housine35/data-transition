/* ======================================================================
   📘 COURS : LES CONTRAINTES EN SQL (SQLite)
   ======================================================================

   ➤ NOT NULL
   - Empêche l’insertion de valeurs NULL dans la colonne.
   - Utile pour les colonnes obligatoires (ex. nom, slug).

   ➤ DEFAULT
   - Définit une valeur par défaut si aucune n’est fournie lors de l’INSERT.
   - Exemple : created_at TEXT DEFAULT (datetime('now'))

   ➤ UNIQUE
   - Garantit que chaque valeur dans la colonne (ou combinaison de colonnes)
     est unique dans la table.
   - Peut aussi être défini via CREATE UNIQUE INDEX.

   ➤ PRIMARY KEY
   - Identifie de manière unique chaque ligne.
   - Combine implicitement NOT NULL + UNIQUE.
   - En SQLite : INTEGER PRIMARY KEY → alias direct de ROWID.

   ➤ CHECK
   - Condition qui doit être respectée pour insérer/modifier une ligne.
   - Exemple : duration INTEGER CHECK(duration > 0)

   ➤ FOREIGN KEY
   - Contrainte d’intégrité référentielle : une valeur doit exister
     dans une autre table.
   - Exemple : FOREIGN KEY (recette_id) REFERENCES recette(id)
   - ⚠️ Dans SQLite, il faut activer : PRAGMA foreign_keys = ON;

   ➤ AUTOINCREMENT (optionnel, SQLite spécifique)
   - Sur une PRIMARY KEY INTEGER, force une incrémentation stricte
     (ne réutilise jamais d’anciens identifiants supprimés).
   - Lourd → rarement nécessaire.
*/

/* ======================================================================
   🧪 DÉMO : TABLE "recette" + DONNÉES + INDEX
   ====================================================================== */

PRAGMA foreign_keys = ON;

BEGIN;

-- Nettoyage
DROP INDEX  IF EXISTS idx_recette_title;
DROP INDEX  IF EXISTS idx_recette_slug_unique;
DROP INDEX  IF EXISTS idx_recette_online_duration;
DROP INDEX  IF EXISTS idx_recette_title_ci;
DROP INDEX  IF EXISTS idx_recette_recent_online;
DROP TABLE  IF EXISTS recette;

-- Création de la table avec contraintes
CREATE TABLE recette (
  id          INTEGER PRIMARY KEY,   -- alias ROWID
  title       TEXT       NOT NULL,   -- obligatoire
  content     TEXT       NOT NULL,   -- obligatoire
  slug        TEXT       NOT NULL,   -- identifiant lisible
  duration    INTEGER    NOT NULL DEFAULT 30
              CHECK (duration > 0 AND duration <= 1440),
  online      INTEGER    NOT NULL DEFAULT 1
              CHECK (online IN (0,1)),    -- booléen simulé
  created_at  TEXT       NOT NULL DEFAULT (datetime('now'))
);

-- Contrainte d’unicité sur slug
CREATE UNIQUE INDEX idx_recette_slug_unique ON recette(slug);

-- Index supplémentaires
CREATE INDEX idx_recette_title ON recette(title);
CREATE INDEX idx_recette_online_duration ON recette(online, duration);
CREATE INDEX idx_recette_title_ci ON recette( lower(title) );
CREATE INDEX idx_recette_recent_online ON recette(title) WHERE online = 1;

-- Données d’exemple
INSERT INTO recette (title, content, slug, duration, online, created_at) VALUES
('Couscous',      'Traditional Algerian dish made from steamed semolina.', 'couscous',       30, 1, '2023-01-01 12:00:00'),
('Chakhchoukha',  'A dish made with torn pieces of flatbread and a spicy sauce.', 'chakhchoukha',   45, 1, '2023-01-02 12:00:00'),
('Tajine Zitoun', 'Chicken or lamb stew with olives and preserved lemons.', 'tajine-zitoun',  60, 1, '2023-01-03 12:00:00'),
('Mhadjeb',       'Semolina flatbread stuffed with tomatoes, onions, and spices.', 'mhadjeb',        20, 1, '2023-01-04 12:00:00'),
('Rechta',        'Homemade pasta served with a chicken and chickpea sauce.', 'rechta',         50, 1, '2023-01-05 12:00:00');

COMMIT;

/* ======================================================================
   🔍 EXEMPLES DE REQUÊTES & INDEX
   ====================================================================== */

-- Recherche rapide via UNIQUE (slug)
EXPLAIN QUERY PLAN SELECT id FROM recette WHERE slug = 'couscous';

-- Recherche par titre exact
EXPLAIN QUERY PLAN SELECT id FROM recette WHERE title = 'Couscous';

-- Recherche insensible à la casse
EXPLAIN QUERY PLAN SELECT id FROM recette WHERE lower(title) = lower('COUSCOUS');

-- Filtrage composite : online + duration
EXPLAIN QUERY PLAN
SELECT id, title FROM recette
WHERE online = 1 AND duration <= 30
ORDER BY duration ASC;

/* ======================================================================
   ✍️ UPSERT (ON CONFLICT)
   ====================================================================== */

INSERT INTO recette (title, content, slug, duration, online)
VALUES ('Couscous (new)', 'Refreshed', 'couscous', 35, 1)
ON CONFLICT(slug) DO UPDATE SET
  title    = excluded.title,
  content  = excluded.content,
  duration = excluded.duration;

SELECT id, title, duration FROM recette WHERE slug='couscous';

/* ======================================================================
   🧭 INTROSPECTION
   ====================================================================== */

PRAGMA table_info('recette');
PRAGMA index_list('recette');
PRAGMA index_info('idx_recette_online_duration');
ANALYZE;
