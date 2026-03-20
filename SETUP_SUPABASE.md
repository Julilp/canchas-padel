# MATCHVIEW — Setup Supabase

## 1. Crear proyecto en Supabase
- Ir a https://supabase.com → New Project
- Anotar la **Project URL** y la **anon public key** (Settings → API)

## 2. Crear las tablas

Ir a **SQL Editor** y ejecutar esto:

```sql
-- Tabla de configuración del evento
CREATE TABLE config (
  id INT PRIMARY KEY DEFAULT 1,
  club_nombre TEXT NOT NULL DEFAULT 'GRAN WILLY',
  club_subtitulo TEXT NOT NULL DEFAULT 'PADEL CLUB',
  organizador TEXT NOT NULL DEFAULT 'WPE PADEL',
  CHECK (id = 1)
);

INSERT INTO config (club_nombre, club_subtitulo, organizador)
VALUES ('GRAN WILLY', 'PADEL CLUB', 'WPE PADEL');

-- Tabla de partidos / tarjetas
CREATE TABLE partidos (
  id SERIAL PRIMARY KEY,
  slot INT NOT NULL UNIQUE CHECK (slot BETWEEN 1 AND 16),
  tag TEXT NOT NULL DEFAULT '---',
  jugador TEXT NOT NULL DEFAULT '---',
  categoria TEXT DEFAULT '',
  es_aviso BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Insertar 16 slots vacíos
INSERT INTO partidos (slot, tag, jugador, categoria, es_aviso)
SELECT s, '---', '---', '', (s IN (8, 16))
FROM generate_series(1, 16) AS s;

-- Habilitar Realtime en la tabla partidos
ALTER PUBLICATION supabase_realtime ADD TABLE partidos;
ALTER PUBLICATION supabase_realtime ADD TABLE config;
```

## 3. Configurar Row Level Security (RLS)

```sql
-- Partidos: lectura pública, escritura solo autenticados
ALTER TABLE partidos ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Lectura pública partidos"
  ON partidos FOR SELECT
  USING (true);

CREATE POLICY "Escritura admin partidos"
  ON partidos FOR ALL
  USING (auth.role() = 'authenticated');

-- Config: lectura pública, escritura solo autenticados
ALTER TABLE config ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Lectura pública config"
  ON config FOR SELECT
  USING (true);

CREATE POLICY "Escritura admin config"
  ON config FOR ALL
  USING (auth.role() = 'authenticated');
```

## 4. Crear usuario admin

- Ir a **Authentication → Users → Add User**
- Crear un usuario con email y contraseña (ej: admin@matchview.com / tu-password)
- Ese será el login del panel admin

## 5. Configurar los HTMLs

En ambos archivos (`display.html` y `admin.html`), reemplazar estas dos líneas al inicio del script:

```js
const SUPABASE_URL = 'https://TU-PROYECTO.supabase.co';
const SUPABASE_KEY = 'tu-anon-key-publica';
```

## 6. Deploy

Podés subir ambos archivos a GitHub Pages, Netlify, o cualquier hosting estático.
- `display.html` → se abre en la TV/monitor del evento
- `admin.html` → se abre en el celular/notebook del staff
```
