# Guía de Conexión MySQL con DataFlex 2025 (64 bits) e Importación de Tablas

Esta guía documenta el proceso para conectar **DataFlex 2025 (64 bits)** con **MySQL** durante la creación de un workspace, importar tablas existentes y configurar los DataDictionaries básicos. Basada en una experiencia real de desarrollo.

---

## 🔧 Parte 1: Configurar la conexión ODBC al crear el workspace

Al crear un nuevo workspace en DataFlex Studio, el asistente (**New Workspace Wizard**) ofrece la posibilidad de configurar la conexión a una base de datos SQL.

### 1.1. Acceder a la configuración de conexión
- En la pantalla **Database Connection** del wizard, haz clic en **"Set Managed Database Connection..."**.
- Se abrirá la ventana **"Add a Connection"**.

### 1.2. Configurar la conexión ODBC
En la ventana **Add a Connection**:

- **Database Type**: selecciona `ODBC_DRV`.
- **Connection Id**: pon un nombre identificativo (por ejemplo, `MySQL_Connection`).
- **ODBC Server Connect**: elige la opción **DSN Connection String**.
- A continuación, necesitas seleccionar un DSN (Data Source Name) ya creado o crear uno nuevo.

#### 1.2.1. Crear un DSN de sistema (64 bits)
1. Haz clic en el botón **"ODBC Administrator (64-bit)"**. Se abrirá el administrador ODBC de Windows (64 bits).
2. Ve a la pestaña **"System DSN"**.
3. Haz clic en **"Agregar..."**.
4. Selecciona el controlador **MySQL ODBC 8.0 Unicode Driver** (o el que corresponda a tu versión).
5. Completa los datos de conexión:
   - **Data Source Name**: pon un nombre descriptivo (ej. `MySQL_DataFlex`).
   - **TCP/IP Server**: `localhost`
   - **Port**: `3306` (o el puerto que uses)
   - **User**: `df_user` (o tu usuario MySQL)
   - **Password**: tu contraseña
   - **Database**: selecciona la base de datos (ej. `dataflex`)
   - Marca **"Save password"** si deseas guardarla.
6. Haz clic en **"Test"** para verificar que la conexión es correcta. Si todo está bien, pulsa **OK** para guardar el DSN.
7. Cierra el administrador ODBC.

#### 1.2.2. Seleccionar el DSN en DataFlex
De vuelta en la ventana **Add a Connection**:

- En el campo **"Select or enter an ODBC Data Source"**, elige el DSN que acabas de crear (aparecerá en el desplegable). Asegúrate de que esté marcado **"Show system data sources"**.
- En el campo **"Extra"**, puedes añadir parámetros adicionales si es necesario, por ejemplo: `UID=df_user;PWD=tu_password;` (aunque ya estén en el DSN, a veces DataFlex los necesita explícitamente).
- Haz clic en **"Test Connection (64-bit)"** para comprobar que DataFlex puede conectar mediante ese DSN.
- Si la prueba es exitosa, pulsa **OK** para guardar la conexión.

Ya tienes la conexión configurada. Continúa con el wizard del workspace (siguiente, siguiente...) hasta finalizar.

> **Nota:** Si en lugar de usar un DSN prefieres una cadena de conexión directa (DSN-less), puedes marcar la opción **"Custom (DSN-less) Connection String"** y pegar la cadena completa, por ejemplo:  
> `DRIVER={MySQL ODBC 8.0 Unicode Driver};SERVER=localhost;PORT=3306;DATABASE=dataflex;UID=df_user;PWD=tu_password;`

---

## Parte 2: Importar tablas desde MySQL usando Connect/Repair Wizard

Una vez que el workspace está creado y conectado a MySQL, necesitas importar las tablas de la base de datos para que DataFlex las reconozca. Esto se hace con el **SQL Connect/Repair Wizard**.

### 2.1. Acceder al wizard
- En el menú principal de DataFlex Studio, ve a **Database → SQL Connect/Repair Wizard**.

### 2.2. Elegir la acción
Aparece la pantalla **Connect/Repair Options**:

- **Select Type of Connect/Repair**: elige **Connect New** (es la primera vez que conectas estas tablas).
  - **Connect New**: crea las definiciones `.INT` necesarias para acceder a las tablas SQL.
  - **Repair Existing**: útil cuando ya tienes tablas conectadas pero has realizado cambios en la estructura de la base de datos (nuevos campos, cambios de tipo, etc.). Esta opción actualiza las definiciones `.INT` sin perder la conexión.
  - **Custom**: para casos avanzados (no suele ser necesario).
- **When Table is already connected**: elige **"Skip tables already connected"** (para evitar sobrescribir si ya están). Si estás reparando, puedes elegir la opción de sincronizar.

Haz clic en **Next >**.

### 2.3. Seleccionar las tablas a importar
En la pantalla **Table Selection**, verás la lista de todas las tablas de tu base de datos MySQL. Marca las que quieras importar. Normalmente, si es la primera vez, selecciona todas las tablas de tu aplicación (por ejemplo: `liga`, `equipo`, `jugador`, etc.). Puedes usar **"Select All in Group"**.

Haz clic en **Next >**.

### 2.4. Configurar el mapeo (Dummy Assign)
En la siguiente pantalla (posiblemente **Column Mapping** o similar), se te presentarán opciones para definir cómo se asignan los campos. **Es importante marcar la opción "Dummy Assign"** (asignación ficticia) para los campos que son claves foráneas o relaciones. Esto le indica a DataFlex que debe mantener las relaciones entre tablas (los `ID` que se conectan entre sí). Sin esta opción, las relaciones podrían no funcionar correctamente.

Revisa que para cada campo de tipo `ID` (o clave foránea) esté marcado **Dummy Assign**. Normalmente el wizard lo hace automáticamente si detecta las relaciones en la base de datos.

Haz clic en **Next >** y luego en **Finish** para completar la importación.

### 2.5. Resultado de la importación
Una vez finalizado, en el explorador del workspace aparecerán:

- En **Other Files**: los archivos `.fd` (file definition) para cada tabla.
- En **Data Dictionaries**: los DataDictionaries básicos (`.dd`) con el mismo nombre que las tablas (prefijados con `c` mayúscula, por ejemplo `cLigaDataDictionary.dd`). Estos DD son funcionales pero muy simples; los ajustaremos después.

---

## Parte 3: Configuración básica del DataDictionary

Los DataDictionaries (DD) son clases que definen el comportamiento de las tablas: relaciones, validaciones, valores por defecto, etc. Puedes modificarlos directamente en el código o usando el asistente del IDE.

Aquí te mostramos un ejemplo completo para la tabla **`liga`**, con las opciones más comunes. Este código está en el archivo `cLigaDataDictionary.dd` (generado automáticamente, pero lo ampliamos).

```dataflex
Use DataDict.pkg

Open liga
Open equipo

Class cLigaDataDictionary is a DataDictionary

    Procedure Construct_Object
        Forward Send Construct_Object
        Set Main_File to liga.File_Number   // Tabla principal

        // Relación con la tabla 'equipo' (uno a muchos)
        Set Add_Client_File to equipo.File_Number

        // Configuración de campos foráneos (relaciones)
        Set Foreign_Field_Option DD_KEYFIELD   DD_NOPUT to True
        Set Foreign_Field_Option DD_KEYFIELD   DD_FINDREQ to True
        Set Foreign_Field_Option DD_INDEXFIELD DD_NOPUT to True
        Set Foreign_Field_Option DD_DEFAULT    DD_DISPLAYONLY to True

        // Configuración de campos individuales

        // El campo ID es autogenerado, no se debe modificar manualmente
        Set Field_Option Field liga.ID DD_NOPUT to True
        Set Field_Option Field liga.ID DD_DISPLAYONLY to True

        // Nombre: obligatorio y en mayúsculas
        Set Field_Option Field liga.Nombre DD_REQUIRED to True
        Set Field_Option Field liga.Nombre DD_CAPSLOCK to True

        // País: obligatorio
        Set Field_Option Field liga.Pais DD_REQUIRED to True

        // Equipos: campo obligatorio (número de equipos actuales)
        Set Field_Option Field liga.Equipos DD_REQUIRED to True

        // EquiposMax: obligatorio, con etiqueta personalizada y solo lectura
        Set Field_Option Field liga.EquiposMax DD_REQUIRED to True
        Set Field_Label_Long Field liga.EquiposMax to "Máximo de Equipos"
        Set Field_Option Field liga.EquiposMax DD_NOPUT to True   // No se puede modificar

    End_Procedure

    // Valores por defecto (se ejecuta al crear un nuevo registro)
    Procedure Field_Defaults
        Forward Send Field_Defaults
        Set Field_Changed_Value Field liga.EquiposMax to 20
    End_Procedure

End_Class
```
### 3.1. Explicación de las instrucciones principales

| Instrucción | Significado |
| :--- | :--- |
| `Set Main_File to tabla.File_Number` | Define la tabla principal que maneja este DataDictionary (DD). |
| `Set Add_Client_File to tabla.File_Number` | Establece una relación **uno a muchos**: la tabla actual tiene muchos registros en la tabla destino (ej: una liga tiene muchos equipos). |
| `Set Add_Server_File to tabla.File_Number` | Establece una relación **muchos a uno**: la tabla actual tiene un "padre" (ej: el equipo pertenece a una liga). |
| `Set Foreign_Field_Option ...` | Configura el comportamiento de claves foráneas: `DD_NOPUT` (no modificable), `DD_FINDREQ` (requerido para buscar), etc. |
| `Set Field_Option ...` | Propiedades de campos: `DD_REQUIRED` (obligatorio), `DD_CAPSLOCK` (mayúsculas), `DD_DISPLAYONLY` (solo lectura). |
| `Set Field_Label_Long` | Cambia la etiqueta (label) que se muestra en la interfaz para ese campo. |
| `Procedure Field_Defaults` | Evento que se ejecuta al crear un nuevo registro; ideal para `Set Field_Changed_Value`. |

---

### 3.2. Relaciones en el DataDictionary

#### `Set Add_Server_File`
Se usa en el DD de la tabla **hija** (muchos a uno) para enlazar con el padre. 
*Ejemplo en `cEquipoDataDictionary`:*

```dataflex
Set Add_Server_File to liga.File_Number
Set Foreign_Field_Option Field equipo.LigaId DD_RELATED_TO to (oLiga_DD)
Set Foreign_Field_Option Field equipo.LigaId DD_RELATED_FIELD to liga.Nombre
```
> **Nota:** Esto permite que al mostrar el campo `equipo.LigaId`, el usuario vea el nombre de la liga en lugar del ID numérico.

#### `Set Add_Client_File`
Se usa en el DD de la tabla **padre** (uno a muchos) para indicar que tiene hijos. Facilita operaciones en cascada (como borrar una liga y que se borren sus equipos automáticamente) si se configura con `DD_DELETECASCADE`.

---

### 3.3. Modificar el DataDictionary
Puedes editar el archivo `.dd` de dos formas:
1.  **Editor de código:** Modificando directamente las instrucciones DataFlex.
2.  **DataDictionary Editor:** Haciendo doble clic sobre el DD en el explorador de componentes dentro de la tabla. Es la forma visual de configurar validaciones, máscaras y relaciones.

---

## Parte 4: Siguientes pasos

Una vez que las tablas están importadas y los DataDictionaries configurados, el flujo de desarrollo continúa con:

* **Crear Zooms (Lookups):** Usar el *Web Mobile Zoom View Wizard* para crear listas de selección (ej: seleccionar una liga desde una lista al crear un equipo).
* **Crear Web Views:** Usar el *Web View Wizard* para generar las pantallas de mantenimiento (CRUD) de cada tabla.
* **Ajustar Navegación:** Configurar el menú principal, añadir botones de acción y pulir la lógica de negocio.

---

## Resumen del flujo de trabajo

1.  **Infraestructura:** Crear workspace y configurar conexión ODBC (DSN de 64 bits).
2.  **Importación:** Usar *SQL Connect/Repair Wizard* (opción *Connect New* + *Dummy Assign*).
3.  **Modelado:** Ajustar DataDictionaries (relaciones, campos requeridos, etiquetas).
4.  **Interfaz:** Desarrollar vistas y zooms según las necesidades del usuario.

---

## Enlaces útiles

* [MySQL ODBC Connector (64-bit)](https://dev.mysql.com/downloads/connector/odbc/)
* [Documentación Oficial de DataFlex](https://docs.dataaccess.com/)
* [Foro de la Comunidad DataFlex](https://support.dataaccess.com/Forums/)
