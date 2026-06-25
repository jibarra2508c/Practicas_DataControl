# Guía MySQL: Crear un usuario exclusivo para una base de datos

Esta guía te muestra cómo crear un usuario en MySQL que tenga acceso solamente a una base de datos específica, garantizando que ningún otro usuario (ni siquiera administradores) pueda acceder a ella.

---

## Conceptos clave

| Concepto | Descripción |
| :--- | :--- |
| `'usuario'@'host'` | Formato de identificación de un usuario. `localhost` = mismo servidor, `%` = cualquier host. |
| `GRANT ... ON bd.*` | **Obligatorio** el `.*` después del nombre de la base de datos. Indica "todas las tablas de esa BD". |
| `FLUSH PRIVILEGES` | Recarga los privilegios en memoria. Asegura que los cambios sean efectivos de inmediato. |

---

## 📋 Paso a paso

### 1. Crear el usuario
```sql
CREATE USER 'nombre_usuario'@'localhost' IDENTIFIED BY 'contraseña_segura';
```
**Ejemplo:**
```sql
CREATE USER 'admin_futbol'@'localhost' IDENTIFIED BY 'Futbol2024$';
```

### 2. Otorgar permisos sobre la base de datos (¡con `.*`!)
```sql
GRANT ALL PRIVILEGES ON nombre_bd.* TO 'nombre_usuario'@'localhost';
```
**Ejemplo:**
```sql
GRANT ALL PRIVILEGES ON db_Futbol.* TO 'admin_futbol'@'localhost';
```

### 3. Aplicar los cambios
```sql
FLUSH PRIVILEGES;
```

### 4. Verificar los permisos
```sql
SHOW GRANTS FOR 'nombre_usuario'@'localhost';
```

### 5. (Opcional) Aislar la base de datos – revocar acceso de otros usuarios
```sql
-- Ver qué usuarios tienen acceso actualmente
SELECT User, Host FROM mysql.db WHERE Db = 'nombre_bd';

-- Revocar permisos a cada usuario no deseado
REVOKE ALL PRIVILEGES ON nombre_bd.* FROM 'otro_usuario'@'localhost';
FLUSH PRIVILEGES;
```

---

## ✅ Verificación final

```sql
-- Listar todos los usuarios
SELECT user, host FROM mysql.user;

-- Ver permisos específicos del usuario creado
SHOW GRANTS FOR 'admin_futbol'@'localhost';

-- Confirmar que solo tu usuario aparece en los accesos a la BD
SELECT User, Host FROM mysql.db WHERE Db = 'db_Futbol';
```

---

## ❗ Errores comunes y soluciones

| Error | Causa | Solución |
| :--- | :--- | :--- |
| **No database selected** | Falta `.*` en GRANT | Usa `nombre_bd.*` en lugar de solo `nombre_bd` |
| **Can't find any matching row** | El usuario no existe | Créalo primero con `CREATE USER` |
| **Access denied for user** | El usuario no tiene permisos | Verifica con `SHOW GRANTS` |
| **DROP USER failed** | El usuario no existe o tiene sesiones activas | Usa `DROP USER IF EXISTS` o termina sus conexiones |

---

## Resumen de comandos esenciales

```sql
-- Crear usuario y darle acceso completo a una BD
CREATE USER 'usuario'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON basedatos.* TO 'usuario'@'localhost';
FLUSH PRIVILEGES;

-- Verificar
SHOW GRANTS FOR 'usuario'@'localhost';

-- Eliminar usuario (si es necesario)
REVOKE ALL PRIVILEGES ON basedatos.* FROM 'usuario'@'localhost';
DROP USER 'usuario'@'localhost';
FLUSH PRIVILEGES;
```
