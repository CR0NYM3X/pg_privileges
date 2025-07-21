# .........EN PROCESO DE DESARROLLO DE FUNCIONES.........

# pg_privileges
🔐 pg_privileges — Función personalizada en PostgreSQL para consultar los permisos de usuarios sobre cualquier objetos. Permite revisar privilegios específicos o globales con facilidad, ideal para auditorías, administración y diagnóstico de roles en bases de datos.

```sql
- te permitira consultar información de los privilegios de otras DB en local usando dblink y te permitira guardar los resultados en una tabla temporal 

pg_privileges( p_object_name, p_user_name , p_db_name, p_save_tmp_table = false ) --- retornara en formato TABLA 
pg_privileges_acl( p_object_name, p_user_name , p_db_name, p_save_tmp_table = false ) --- Retornara solo los ACL 

```


```sql
1.- DB 
	select null as schema,datname as object_name,'DATABASE' AS object_type , datacl   AS relacl   from pg_database where datacl is not null  


2.- Schemas 

	select null as schema, nspname as object_name, 'SCHEMA' as object_type, nspacl  AS relacl FROM pg_namespace where nspacl is not null ;



3.- Fun y Proc 

	select null as schema, nspname as object_name, 'SCHEMA' as object_type, nspacl  AS relacl FROM pg_namespace where nspacl is not null ;

4.- Secuencias



5.- Types 

	SELECT
		NSPNAME AS SCHEMA,
		RELNAME AS OBJECT_NAME,
		CASE
		--WHEN (nc.oid = pg_my_temp_schema()) THEN 'LOCAL TEMPORARY'::text
			WHEN RELKIND = 'r' THEN 'TABLE'
			WHEN RELKIND = 'p' THEN 'TABLE'
			WHEN RELKIND = 'i' THEN 'INDEX'
			WHEN RELKIND = 'S' THEN 'SEQUENCE'
			WHEN RELKIND = 'v' THEN 'VIEW'
			WHEN RELKIND = 'm' THEN 'MATERIALIZED VIEW'
			WHEN RELKIND = 'c' THEN 'type'
			WHEN RELKIND = 't' THEN 'TOAST TABLE'
			WHEN RELKIND = 'f' THEN 'FOREIGN TABLE'
			WHEN RELKIND = 'p' THEN 'PARTITIONED FOREIGN'
			WHEN RELKIND = 'I' THEN 'PARTITIONED INDEX'
			ELSE 'other'
		END AS OBJECT_TYPE,
		RELACL AS RELACL
	FROM
		PG_CLASS AS CL
		LEFT JOIN PG_TYPE AS PTY ON CL.OID = PTY.TYPRELID
		LEFT JOIN PG_NAMESPACE AS NC ON CL.RELNAMESPACE = NC.OID
	WHERE
		RELKIND IN ('c')
		




6.- Tablas y Vistas privileges
7.- DEFAULT  privileges
8.- Columns privileges. 
9.- fdw

---
  select   ' SELECT '|| string_agg(column_name,',')  ||  ' FROM ' || table_schema || '.' || table_name|| ' where ' || string_agg(column_name,',') || ' IS NOT NULL ;'   from information_schema.columns where column_name ilike '%acl' group by table_schema,table_name;
+-----------------------------------------------------------------------------------+
|                                     ?column?                                      |
+-----------------------------------------------------------------------------------+
|  SELECT attacl FROM pg_catalog.pg_attribute where attacl IS NOT NULL ;            |
|  SELECT relacl FROM pg_catalog.pg_class where relacl IS NOT NULL ;                |
|  SELECT datacl FROM pg_catalog.pg_database where datacl IS NOT NULL ;             |
|  SELECT defaclacl FROM pg_catalog.pg_default_acl where defaclacl IS NOT NULL ;    |
|  SELECT fdwacl FROM pg_catalog.pg_foreign_data_wrapper where fdwacl IS NOT NULL ; |
|  SELECT srvacl FROM pg_catalog.pg_foreign_server where srvacl IS NOT NULL ;       |
|  SELECT lanacl FROM pg_catalog.pg_language where lanacl IS NOT NULL ;             |
|  SELECT lomacl FROM pg_catalog.pg_largeobject_metadata where lomacl IS NOT NULL ; |
|  SELECT nspacl FROM pg_catalog.pg_namespace where nspacl IS NOT NULL ;            |
|  SELECT paracl FROM pg_catalog.pg_parameter_acl where paracl IS NOT NULL ;        |
|  SELECT proacl FROM pg_catalog.pg_proc where proacl IS NOT NULL ;                 |
|  SELECT spcacl FROM pg_catalog.pg_tablespace where spcacl IS NOT NULL ;           |
|  SELECT typacl FROM pg_catalog.pg_type where typacl IS NOT NULL ;                 |
+-----------------------------------------------------------------------------------+
```
