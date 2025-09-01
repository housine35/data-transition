-- file: recipes.sql
-- ============================================================================
-- Mini-cours express : CREATE / ALTER TABLE / DROP / SELECT
-- ----------------------------------------------------------------------------
-- CREATE TABLE
--   Crée une table et ses colonnes + contraintes.
--   Ex : CREATE TABLE t (id bigserial PRIMARY KEY, name varchar(100) NOT NULL);
--
-- ALTER TABLE
--   Modifie le schéma : ajouter/supprimer/renommer/typer des colonnes.
--   Ex : ALTER TABLE t ADD COLUMN IF NOT EXISTS age smallint;
--        ALTER TABLE t ALTER COLUMN age TYPE integer;
--        ALTER TABLE t RENAME COLUMN name TO title;
--        ALTER TABLE t DROP COLUMN IF EXISTS age;
--
-- DROP TABLE
--   Supprime une table (facultatif : IF EXISTS).
--   Ex : DROP TABLE IF EXISTS t;
--
-- SELECT (base)
--   SELECT [DISTINCT] colonnes
--   FROM table
--   [WHERE ... (AND/OR, =, <>, IN, BETWEEN, LIKE/ILIKE, IS NULL)]
--   [ORDER BY col [ASC|DESC]]
--   [LIMIT n [OFFSET m]];
-- ============================================================================


-- ============================================================================
-- 1) Reset idempotent (rejouable sans erreur)
-- ============================================================================
DROP TABLE IF EXISTS recipes;


-- ============================================================================
-- 2) Schéma de la table
--    - id PK auto-incrémenté
--    - slug unique
--    - duration >= 0
--    - online par défaut à TRUE
--    - created_at par défaut à NOW()
-- ============================================================================
CREATE TABLE recipes (
  id         BIGSERIAL PRIMARY KEY,
  title      VARCHAR(150) NOT NULL,
  content    TEXT         NOT NULL,
  slug       VARCHAR(50)  NOT NULL UNIQUE,
  duration   SMALLINT     CHECK (duration >= 0),
  online     BOOLEAN      NOT NULL DEFAULT TRUE,
  created_at TIMESTAMP    NOT NULL DEFAULT NOW()
);

-- Index utiles (facultatifs)
CREATE INDEX IF NOT EXISTS idx_recipes_created_at ON recipes (created_at);
CREATE INDEX IF NOT EXISTS idx_recipes_online     ON recipes (online);


-- ============================================================================
-- 3) Jeux de données (Algérie)
-- ============================================================================
INSERT INTO recipes (title, content, slug, duration, online, created_at) VALUES
('Couscous',      'Traditional Algerian dish made from steamed semolina.',                    'couscous',       30, TRUE, '2023-01-01 12:00:00'),
('Chakhchoukha',  'A dish made with torn pieces of flatbread and a spicy sauce.',            'chakhchoukha',   45, TRUE, '2023-01-02 12:00:00'),
('Tajine Zitoun', 'Chicken or lamb stew with olives and preserved lemons.',                  'tajine-zitoun',  60, TRUE, '2023-01-03 12:00:00'),
('Mhadjeb',       'Semolina flatbread stuffed with tomatoes, onions, and spices.',           'mhadjeb',        20, TRUE, '2023-01-04 12:00:00'),
('Rechta',        'Homemade pasta served with a chicken and chickpea sauce.',                'rechta',         50, TRUE, '2023-01-05 12:00:00');


-- ============================================================================
-- 4) Exemples ALTER TABLE (modèle)
--    ⚠️ Laisse ces lignes commentées et décommente au besoin.
-- ============================================================================
-- -- Ajouter une colonne
-- ALTER TABLE recipes ADD COLUMN IF NOT EXISTS difficulty VARCHAR(20);
-- -- Modifier le type (ici on reste en SMALLINT, exemple illustratif)
-- ALTER TABLE recipes ALTER COLUMN duration TYPE SMALLINT;
-- -- Renommer une colonne
-- ALTER TABLE recipes RENAME COLUMN content TO body;
-- -- Supprimer une colonne
-- ALTER TABLE recipes DROP COLUMN IF EXISTS body;
-- -- Renommer la table
-- ALTER TABLE recipes RENAME TO recipes_archive;  -- puis renomme-la de nouveau si besoin


-- ============================================================================
-- 5) Exemples SELECT (toutes les variantes de base)
-- ============================================================================
-- a) Toutes les colonnes
SELECT * FROM recipes;

-- b) Colonnes spécifiques
SELECT title, duration FROM recipes;

-- c) WHERE
SELECT title FROM recipes WHERE online = TRUE;

-- d) Tri
SELECT title FROM recipes ORDER BY created_at DESC;

-- e) Limiter / paginer
SELECT title FROM recipes ORDER BY created_at DESC LIMIT 5 OFFSET 0;

-- f) BETWEEN
SELECT title FROM recipes WHERE duration BETWEEN 10 AND 30;

-- g) IN / NOT IN
SELECT title FROM recipes WHERE title IN ('Couscous', 'Chakhchoukha');
SELECT title FROM recipes WHERE title NOT IN ('Couscous', 'Chakhchoukha');

-- h) LIKE (sensible à la casse) / ILIKE (insensible à la casse, PostgreSQL)
SELECT title, content FROM recipes WHERE title LIKE  '%ou%';
SELECT title, content FROM recipes WHERE title ILIKE '%ou%';

-- i) DISTINCT (valeurs uniques)
SELECT DISTINCT duration FROM recipes ORDER BY duration;

-- j) Filtres combinés
SELECT title
FROM recipes
WHERE online = TRUE AND duration <= 45
ORDER BY created_at DESC;


-- ============================================================================
-- 6) UPDATE / DELETE (exemples pratiques)
--    ⚠️ Ces requêtes MODIFIENT les données. Exécute-les seulement si tu veux
--    l'effet correspondant.
-- ============================================================================
-- UPDATE : mettre à jour "Chakhchoukha"
UPDATE recipes
SET online = FALSE,
    duration = 60,
    created_at = TIMESTAMP '2025-07-04 12:00:00'
WHERE title = 'Chakhchoukha';

-- DELETE : supprimer "Tajine Zitoun"
-- DELETE FROM recipes WHERE title = 'Tajine Zitoun';
