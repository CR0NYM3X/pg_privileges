# Base de datos
```sql







select 
	 current_database() as db_name, schema,object_name,object_type,grantor,grantee,privilege_type
	,case  
		WHEN  letter_privilege like  '%r*%' and privilege_type = 'SELECT'   then true
		WHEN  letter_privilege like  '%a*%' and privilege_type = 'INSERT'   then true
		WHEN  letter_privilege like  '%w*%' and privilege_type = 'UPDATE'   then true
		WHEN  letter_privilege like  '%d*%' and privilege_type = 'DELETE'  then true
		WHEN  letter_privilege like  '%D*%' and privilege_type = 'TRUNCATE'   then true
		WHEN  letter_privilege like  '%x*%' and privilege_type = 'REFERENCES'  then true
		WHEN  letter_privilege like  '%t*%' and privilege_type = 'TRIGGER'  then true
		WHEN  letter_privilege like  '%C*%' and privilege_type = 'CREATE'   then true
		WHEN  letter_privilege like  '%c*%' and privilege_type = 'CONNECT'   then true
		WHEN  letter_privilege like  '%T*%' and privilege_type = 'TEMPORARY'   then true
		WHEN  letter_privilege like  '%X*%' and privilege_type = 'EXECUTE'   then true
		WHEN  letter_privilege like  '%U*%' and privilege_type = 'USAGE'   then true
		WHEN  letter_privilege like  '%s*%' and privilege_type = 'SET'   then true
		WHEN  letter_privilege like  '%A*%' and privilege_type = 'ALTER SYSTEM'   then true
	else  
		false
	end as is_grantable 
from
(


	select 
			schema
			,object_name
			,object_type
			,grantor
			,grantee
			,letter_privilege
			,split_part(privilege_type, ',', generate_series(1, length(privilege_type) - length(replace(privilege_type, ',', '')) + 1))	as privilege_type	--- 
	from 
	(

		select 
			schema
			,object_name
			,object_type
			,grantor
			,case  when grantee = '' THEN 'PUBLIC' else grantee END as grantee
			,letter_privilege	
			,TRIM(
					BOTH ',' FROM
					( 
						CASE  WHEN letter_privilege like '%r%' THEN 'SELECT,' ELSE '' END || 
						CASE  WHEN letter_privilege like '%a%' THEN 'INSERT,' ELSE '' END || 
						CASE  WHEN letter_privilege like '%w%' THEN 'UPDATE,' ELSE '' END || 
						CASE  WHEN letter_privilege like '%d%' THEN 'DELETE,'ELSE '' END || 
						CASE  WHEN letter_privilege like '%D%' THEN 'TRUNCATE,' ELSE '' END || 
						CASE  WHEN letter_privilege like '%x%' THEN 'REFERENCES,'ELSE '' END || 
						CASE  WHEN letter_privilege like '%t%' THEN 'TRIGGER,'ELSE '' END || 
						CASE  WHEN letter_privilege like '%C%' THEN 'CREATE,' ELSE '' END || 
						CASE  WHEN letter_privilege like '%c%' THEN 'CONNECT,' ELSE '' END || 
						CASE  WHEN letter_privilege like '%T%' THEN 'TEMPORARY,' ELSE '' END || 
						CASE  WHEN letter_privilege like '%X%' THEN 'EXECUTE,' ELSE '' END || 
						CASE  WHEN letter_privilege like '%U%' THEN 'USAGE,' ELSE '' END || 
						CASE  WHEN letter_privilege like '%s%' THEN 'SET,' ELSE '' END || 
						CASE  WHEN letter_privilege like '%A%' THEN 'ALTER SYSTEM,' ELSE '' END   
					)
				) as privilege_type 
		from(

				SELECT 
				schema
				,object_name
				,object_type
				,split_part( split_part(array_to_string(relacl , ',') , ',', generate_series(1, length(array_to_string(relacl , ',') ) - length( replace(array_to_string(relacl , ',') , ',', '')) + 1))  , '/', 2) AS grantor
				,split_part(split_part( split_part(array_to_string(relacl , ',') , ',', generate_series(1, length(array_to_string(relacl , ',') ) - length(replace(array_to_string(relacl , ',') , ',', '')) + 1))  , '/', 1), '=', 1) as grantee 
				,split_part(split_part( split_part(array_to_string(relacl , ',') , ',', generate_series(1, length(array_to_string(relacl , ',') ) - length(replace(array_to_string(relacl , ',') , ',', '')) + 1))  , '/', 1), '=', 2) as letter_privilege  
				FROM 
				(  
				
				/* AQUI VAN LOS ACL DE LOS OBJETOS  */
				 
				 --- BASE DE DATOS 
				 select null as schema,datname as object_name,'DATABASE' AS object_type , datacl   AS relacl   from pg_database where datacl is not null  

				)  as v 

			) as a 
	) as a 

) as a 
where  not grantee in('postgres','PUBLIC')   
		and pg_catalog.set_config('client_encoding', current_setting('server_encoding'), true) is not null
;


```


### Funciones o Procedimientos
```sql

SELECT  
current_database() as db_name 
 ,grantor
 ,grantee
 ,b.routine_type as type 
,a.routine_schema  as schema
 ,a.routine_name as object_name
 ,privilege_type
 ,case when is_grantable = 'NO'::varchar then false else true end as is_grantable 
FROM information_schema.routine_privileges as a
left join information_schema.routines  as b on a.routine_name=b.routine_name
where not a.grantee in('PUBLIC','postgres')  
AND pg_catalog.set_config('client_encoding', current_setting('server_encoding'), true) is not null  ;
 
```

### Secuencias
```sql
select 
	 current_database() as db_name, schema,object_name,object_type,grantor,grantee,privilege_type
	,case  
		WHEN  letter_privilege like  '%r*%' and privilege_type = 'SELECT'   then true
		WHEN  letter_privilege like  '%a*%' and privilege_type = 'INSERT'   then true
		WHEN  letter_privilege like  '%w*%' and privilege_type = 'UPDATE'   then true
		WHEN  letter_privilege like  '%d*%' and privilege_type = 'DELETE'  then true
		WHEN  letter_privilege like  '%D*%' and privilege_type = 'TRUNCATE'   then true
		WHEN  letter_privilege like  '%x*%' and privilege_type = 'REFERENCES'  then true
		WHEN  letter_privilege like  '%t*%' and privilege_type = 'TRIGGER'  then true
		WHEN  letter_privilege like  '%C*%' and privilege_type = 'CREATE'   then true
		WHEN  letter_privilege like  '%c*%' and privilege_type = 'CONNECT'   then true
		WHEN  letter_privilege like  '%T*%' and privilege_type = 'TEMPORARY'   then true
		WHEN  letter_privilege like  '%X*%' and privilege_type = 'EXECUTE'   then true
		WHEN  letter_privilege like  '%U*%' and privilege_type = 'USAGE'   then true
		WHEN  letter_privilege like  '%s*%' and privilege_type = 'SET'   then true
		WHEN  letter_privilege like  '%A*%' and privilege_type = 'ALTER SYSTEM'   then true
	else  
		false
	end as is_grantable 
from
(


	select 
			schema
			,object_name
			,object_type
			,grantor
			,grantee
			,letter_privilege
			,split_part(privilege_type, ',', generate_series(1, length(privilege_type) - length(replace(privilege_type, ',', '')) + 1))	as privilege_type	--- 
	from 
	(

		select 
			schema
			,object_name
			,object_type
			,grantor
			,case  when grantee = '' THEN 'PUBLIC' else grantee END as grantee
			,letter_privilege	
			,TRIM(
					BOTH ',' FROM
					( 
						CASE  WHEN letter_privilege like '%r%' THEN 'SELECT,' ELSE '' END || 
						CASE  WHEN letter_privilege like '%a%' THEN 'INSERT,' ELSE '' END || 
						CASE  WHEN letter_privilege like '%w%' THEN 'UPDATE,' ELSE '' END || 
						CASE  WHEN letter_privilege like '%d%' THEN 'DELETE,'ELSE '' END || 
						CASE  WHEN letter_privilege like '%D%' THEN 'TRUNCATE,' ELSE '' END || 
						CASE  WHEN letter_privilege like '%x%' THEN 'REFERENCES,'ELSE '' END || 
						CASE  WHEN letter_privilege like '%t%' THEN 'TRIGGER,'ELSE '' END || 
						CASE  WHEN letter_privilege like '%C%' THEN 'CREATE,' ELSE '' END || 
						CASE  WHEN letter_privilege like '%c%' THEN 'CONNECT,' ELSE '' END || 
						CASE  WHEN letter_privilege like '%T%' THEN 'TEMPORARY,' ELSE '' END || 
						CASE  WHEN letter_privilege like '%X%' THEN 'EXECUTE,' ELSE '' END || 
						CASE  WHEN letter_privilege like '%U%' THEN 'USAGE,' ELSE '' END || 
						CASE  WHEN letter_privilege like '%s%' THEN 'SET,' ELSE '' END || 
						CASE  WHEN letter_privilege like '%A%' THEN 'ALTER SYSTEM,' ELSE '' END   
					)
				) as privilege_type 
		from(

				SELECT 
				schema
				,object_name
				,object_type
				,split_part( split_part(array_to_string(relacl , ',') , ',', generate_series(1, length(array_to_string(relacl , ',') ) - length( replace(array_to_string(relacl , ',') , ',', '')) + 1))  , '/', 2) AS grantor
				,split_part(split_part( split_part(array_to_string(relacl , ',') , ',', generate_series(1, length(array_to_string(relacl , ',') ) - length(replace(array_to_string(relacl , ',') , ',', '')) + 1))  , '/', 1), '=', 1) as grantee 
				,split_part(split_part( split_part(array_to_string(relacl , ',') , ',', generate_series(1, length(array_to_string(relacl , ',') ) - length(replace(array_to_string(relacl , ',') , ',', '')) + 1))  , '/', 1), '=', 2) as letter_privilege  
				FROM 
				(  
				
				/* AQUI VAN LOS ACL DE LOS OBJETOS  */
				 
				 --- SEQUENCE  
				  
				SELECT 
				nspname as schema
				,relname AS object_name
				,CASE 
				--WHEN (nc.oid = pg_my_temp_schema()) THEN 'LOCAL TEMPORARY'::text
				WHEN relkind= 'r' THEN 'TABLE'
				WHEN relkind= 'p' THEN 'TABLE'
				WHEN relkind= 'i' THEN 'INDEX'
				WHEN relkind= 'S' THEN 'SEQUENCE'
				WHEN relkind= 'v' THEN 'VIEW'
				WHEN relkind= 'm' THEN 'MATERIALIZED VIEW'
				WHEN relkind= 'c' THEN 'type'
				WHEN relkind= 't' THEN 'TOAST TABLE'
				WHEN relkind= 'f' THEN 'FOREIGN TABLE'
				WHEN relkind= 'p' THEN 'PARTITIONED FOREIGN'
				WHEN relkind= 'I' THEN 'PARTITIONED INDEX'
				ELSE 'other'
				END AS object_type
				, relacl  AS relacl
				FROM pg_class as cl
				left join pg_type as pty on  cl.oid = pty.typrelid 
				left join pg_namespace as nc on   cl.relnamespace= nc.oid

				 WHERE relacl is not null and not nspname in( 'information_schema','pg_catalog','pg_toast')  and not nspname ilike 'pg_temp%' 
				 and relkind in('S')
				 
 

				)  as v 

			) as a 
	) as a 

) as a 
where  not grantee in('postgres','PUBLIC')   
		and pg_catalog.set_config('client_encoding', current_setting('server_encoding'), true) is not null
;


```

### Types
```sql
select 
	 current_database() as db_name, schema,object_name,object_type,grantor,grantee,privilege_type
	,case  
		WHEN  letter_privilege like  '%r*%' and privilege_type = 'SELECT'   then true
		WHEN  letter_privilege like  '%a*%' and privilege_type = 'INSERT'   then true
		WHEN  letter_privilege like  '%w*%' and privilege_type = 'UPDATE'   then true
		WHEN  letter_privilege like  '%d*%' and privilege_type = 'DELETE'  then true
		WHEN  letter_privilege like  '%D*%' and privilege_type = 'TRUNCATE'   then true
		WHEN  letter_privilege like  '%x*%' and privilege_type = 'REFERENCES'  then true
		WHEN  letter_privilege like  '%t*%' and privilege_type = 'TRIGGER'  then true
		WHEN  letter_privilege like  '%C*%' and privilege_type = 'CREATE'   then true
		WHEN  letter_privilege like  '%c*%' and privilege_type = 'CONNECT'   then true
		WHEN  letter_privilege like  '%T*%' and privilege_type = 'TEMPORARY'   then true
		WHEN  letter_privilege like  '%X*%' and privilege_type = 'EXECUTE'   then true
		WHEN  letter_privilege like  '%U*%' and privilege_type = 'USAGE'   then true
		WHEN  letter_privilege like  '%s*%' and privilege_type = 'SET'   then true
		WHEN  letter_privilege like  '%A*%' and privilege_type = 'ALTER SYSTEM'   then true
	else  
		false
	end as is_grantable 
from
(


	select 
			schema
			,object_name
			,object_type
			,grantor
			,grantee
			,letter_privilege
			,split_part(privilege_type, ',', generate_series(1, length(privilege_type) - length(replace(privilege_type, ',', '')) + 1))	as privilege_type	--- 
	from 
	(

		select 
			schema
			,object_name
			,object_type
			,grantor
			,case  when grantee = '' THEN 'PUBLIC' else grantee END as grantee
			,letter_privilege	
			,TRIM(
					BOTH ',' FROM
					( 
						CASE  WHEN letter_privilege like '%r%' THEN 'SELECT,' ELSE '' END || 
						CASE  WHEN letter_privilege like '%a%' THEN 'INSERT,' ELSE '' END || 
						CASE  WHEN letter_privilege like '%w%' THEN 'UPDATE,' ELSE '' END || 
						CASE  WHEN letter_privilege like '%d%' THEN 'DELETE,'ELSE '' END || 
						CASE  WHEN letter_privilege like '%D%' THEN 'TRUNCATE,' ELSE '' END || 
						CASE  WHEN letter_privilege like '%x%' THEN 'REFERENCES,'ELSE '' END || 
						CASE  WHEN letter_privilege like '%t%' THEN 'TRIGGER,'ELSE '' END || 
						CASE  WHEN letter_privilege like '%C%' THEN 'CREATE,' ELSE '' END || 
						CASE  WHEN letter_privilege like '%c%' THEN 'CONNECT,' ELSE '' END || 
						CASE  WHEN letter_privilege like '%T%' THEN 'TEMPORARY,' ELSE '' END || 
						CASE  WHEN letter_privilege like '%X%' THEN 'EXECUTE,' ELSE '' END || 
						CASE  WHEN letter_privilege like '%U%' THEN 'USAGE,' ELSE '' END || 
						CASE  WHEN letter_privilege like '%s%' THEN 'SET,' ELSE '' END || 
						CASE  WHEN letter_privilege like '%A%' THEN 'ALTER SYSTEM,' ELSE '' END   
					)
				) as privilege_type 
		from(

				SELECT 
				schema
				,object_name
				,object_type
				,split_part( split_part(array_to_string(relacl , ',') , ',', generate_series(1, length(array_to_string(relacl , ',') ) - length( replace(array_to_string(relacl , ',') , ',', '')) + 1))  , '/', 2) AS grantor
				,split_part(split_part( split_part(array_to_string(relacl , ',') , ',', generate_series(1, length(array_to_string(relacl , ',') ) - length(replace(array_to_string(relacl , ',') , ',', '')) + 1))  , '/', 1), '=', 1) as grantee 
				,split_part(split_part( split_part(array_to_string(relacl , ',') , ',', generate_series(1, length(array_to_string(relacl , ',') ) - length(replace(array_to_string(relacl , ',') , ',', '')) + 1))  , '/', 1), '=', 2) as letter_privilege  
				FROM 
				(  
				
				/* AQUI VAN LOS ACL DE LOS OBJETOS  */
				 
				 --- types  
				  
					SELECT 
					nspname as schema
					,relname AS object_name
					,CASE 
					--WHEN (nc.oid = pg_my_temp_schema()) THEN 'LOCAL TEMPORARY'::text
					WHEN relkind= 'r' THEN 'TABLE'
					WHEN relkind= 'p' THEN 'TABLE'
					WHEN relkind= 'i' THEN 'INDEX'
					WHEN relkind= 'S' THEN 'SEQUENCE'
					WHEN relkind= 'v' THEN 'VIEW'
					WHEN relkind= 'm' THEN 'MATERIALIZED VIEW'
					WHEN relkind= 'c' THEN 'type'
					WHEN relkind= 't' THEN 'TOAST TABLE'
					WHEN relkind= 'f' THEN 'FOREIGN TABLE'
					WHEN relkind= 'p' THEN 'PARTITIONED FOREIGN'
					WHEN relkind= 'I' THEN 'PARTITIONED INDEX'
					ELSE 'other'
					END AS object_type
					, relacl  AS relacl
					FROM pg_class as cl
					left join pg_type as pty on  cl.oid = pty.typrelid 
					left join pg_namespace as nc on   cl.relnamespace= nc.oid

					 WHERE  relacl is not null and not nspname in( 'information_schema','pg_catalog','pg_toast')  and not nspname ilike 'pg_temp%' and 
					 relkind in('c')
 

				)  as v 

			) as a 
	) as a 

) as a 
where  not grantee in('postgres','PUBLIC')   
		and pg_catalog.set_config('client_encoding', current_setting('server_encoding'), true) is not null
;
```

### Permisos Granulares
```sql





select 

 current_database() as db
,* from 
 
(
 

select grantor,grantee,object_type,schema,object_name,privilege_type,is_grantable from 
(select * ,
case  
WHEN  letter_privilege like  '%r*%' and privilege_type = 'SELECT'   then true
WHEN  letter_privilege like  '%a*%' and privilege_type = 'INSERT'   then true
WHEN  letter_privilege like  '%w*%' and privilege_type = 'UPDATE'   then true
WHEN  letter_privilege like  '%d*%' and privilege_type = 'DELETE'  then true
WHEN  letter_privilege like  '%D*%' and privilege_type = 'TRUNCATE'   then true
WHEN  letter_privilege like  '%x*%' and privilege_type = 'REFERENCES'  then true
WHEN  letter_privilege like  '%t*%' and privilege_type = 'TRIGGER'  then true
WHEN  letter_privilege like  '%C*%' and privilege_type = 'CREATE'   then true
WHEN  letter_privilege like  '%c*%' and privilege_type = 'CONNECT'   then true
WHEN  letter_privilege like  '%T*%' and privilege_type = 'TEMPORARY'   then true
WHEN  letter_privilege like  '%X*%' and privilege_type = 'EXECUTE'   then true
WHEN  letter_privilege like  '%U*%' and privilege_type = 'USAGE'   then true
WHEN  letter_privilege like  '%s*%' and privilege_type = 'SET'   then true
WHEN  letter_privilege like  '%A*%' and privilege_type = 'ALTER SYSTEM'   then true
else  false
end as is_grantable 
from 
(select * from 
(select  

grantor
,grantee
,object_type
,schema
,object_name
,letter_privilege
,unnest(string_to_array(( 
CASE  WHEN letter_privilege like '%r%' THEN 'SELECT' ELSE '' END || ',' || 
CASE  WHEN letter_privilege like '%a%' THEN 'INSERT' ELSE '' END || ',' || 
CASE  WHEN letter_privilege like '%w%' THEN 'UPDATE' ELSE '' END || ',' || 
CASE  WHEN letter_privilege like '%d%' THEN 'DELETE'ELSE '' END || ',' || 
CASE  WHEN letter_privilege like '%D%' THEN 'TRUNCATE' ELSE '' END || ',' || 
CASE  WHEN letter_privilege like '%x%' THEN 'REFERENCES'ELSE '' END || ',' || 
CASE  WHEN letter_privilege like '%t%' THEN 'TRIGGER'ELSE '' END || ',' || 
CASE  WHEN letter_privilege like '%C%' THEN 'CREATE' ELSE '' END || ',' || 
CASE  WHEN letter_privilege like '%c%' THEN 'CONNECT' ELSE '' END || ',' || 
CASE  WHEN letter_privilege like '%T%' THEN 'TEMPORARY' ELSE '' END || ',' || 
CASE  WHEN letter_privilege like '%X%' THEN 'EXECUTE' ELSE '' END || ',' || 
CASE  WHEN letter_privilege like '%U%' THEN 'USAGE' ELSE '' END || ',' || 
CASE  WHEN letter_privilege like '%s%' THEN 'SET' ELSE '' END || ',' || 
CASE  WHEN letter_privilege like '%A%' THEN 'ALTER SYSTEM' ELSE '' END   
) , ',')) as privilege_type

  
from (
SELECT 
schema
,object_name
,object_type
,split_part( split_part(array_to_string(relacl , ', ') , ', ', generate_series(1, length(array_to_string(relacl , ', ') ) - length( replace(array_to_string(relacl , ', ') , ', ', '')) + 1))  , '/', 2) AS grantor
,split_part(split_part( split_part(array_to_string(relacl , ', ') , ', ', generate_series(1, length(array_to_string(relacl , ', ') ) - length(replace(array_to_string(relacl , ', ') , ', ', '')) + 1))  , '/', 1), '=', 1) as grantee 
,split_part(split_part( split_part(array_to_string(relacl , ', ') , ', ', generate_series(1, length(array_to_string(relacl , ', ') ) - length(replace(array_to_string(relacl , ', ') , ', ', '')) + 1))  , '/', 1), '=', 2) as letter_privilege  
FROM 
( select * from (
 
 
 ---- ESQUEMAS 
select null as schema, nspname as object_name, 'SCHEMA' as object_type, nspacl  AS relacl FROM pg_namespace where not nspname in( 'information_schema','pg_catalog') and nspacl is not null 
 
 
 union all 
 
 --- BASE DE DATOS 
 select null as schema,datname as object_name,'DATABASE' AS object_type , datacl    from pg_database where datacl is not null  
 
 
 union all 

select * from 
(

------ TABLAS,VIEW,INDEX,SEQUENCES,TYPE,ETC
SELECT 
nspname as schema
,relname AS object_name
,CASE 
--WHEN (nc.oid = pg_my_temp_schema()) THEN 'LOCAL TEMPORARY'::text
WHEN relkind= 'r' THEN 'TABLE'
WHEN relkind= 'p' THEN 'TABLE'
WHEN relkind= 'i' THEN 'INDEX'
WHEN relkind= 'S' THEN 'SEQUENCE'
WHEN relkind= 'v' THEN 'VIEW'
WHEN relkind= 'm' THEN 'MATERIALIZED VIEW'
WHEN relkind= 'c' THEN 'type'
WHEN relkind= 't' THEN 'TOAST TABLE'
WHEN relkind= 'f' THEN 'FOREIGN TABLE'
WHEN relkind= 'p' THEN 'PARTITIONED FOREIGN'
WHEN relkind= 'I' THEN 'PARTITIONED INDEX'
ELSE 'other'
END AS object_type
, relacl  as privileges 
FROM pg_class as cl
left join pg_type as pty on  cl.oid = pty.typrelid 
left join pg_namespace as nc on   cl.relnamespace= nc.oid

 WHERE relacl is not null and not nspname in( 'information_schema','pg_catalog','pg_toast')  and not nspname ilike 'pg_temp%' 
 and relkind in('r' ,'p' ,'i' ,'S' ,'v' ,'m' ,'t' ,'f' ,'p' ,'I')
 
 union all 
 
 ------ FUNCIONES 
  SELECT 
--p.oid,
n.nspname as  schema,
p.proname as object_name ,

(
CASE
  WHEN p.proisagg THEN 'agg'
  WHEN p.proiswindow THEN 'window'
  WHEN p.prorettype = 'pg_catalog.trigger'::pg_catalog.regtype THEN 'trigger'
  ELSE 'FUNCTION'
END  )::information_schema.character_data AS object_type,
p.proacl as privileges
   FROM pg_proc as p
   left join pg_namespace as n on    p.pronamespace = n.oid where  p.proacl is not null and  not nspname in( 'information_schema','pg_catalog','pg_toast')  and not nspname ilike 'pg_temp%'
 
 
 
 ) as a    
 
 
 
) as g   
)  as v

) as f where not grantee in('',  'postgres','PUBLIC','pg_database_owner') 
) as a where privilege_type != ''
) as b ) as x ) as d where  pg_catalog.set_config('client_encoding', current_setting('server_encoding'), true) is not null  order by grantee ;
```

### Esquemas
```sql
select 
	 current_database() as db_name, schema,object_name,object_type,grantor,grantee,privilege_type
	,case  
		WHEN  letter_privilege like  '%r*%' and privilege_type = 'SELECT'   then true
		WHEN  letter_privilege like  '%a*%' and privilege_type = 'INSERT'   then true
		WHEN  letter_privilege like  '%w*%' and privilege_type = 'UPDATE'   then true
		WHEN  letter_privilege like  '%d*%' and privilege_type = 'DELETE'  then true
		WHEN  letter_privilege like  '%D*%' and privilege_type = 'TRUNCATE'   then true
		WHEN  letter_privilege like  '%x*%' and privilege_type = 'REFERENCES'  then true
		WHEN  letter_privilege like  '%t*%' and privilege_type = 'TRIGGER'  then true
		WHEN  letter_privilege like  '%C*%' and privilege_type = 'CREATE'   then true
		WHEN  letter_privilege like  '%c*%' and privilege_type = 'CONNECT'   then true
		WHEN  letter_privilege like  '%T*%' and privilege_type = 'TEMPORARY'   then true
		WHEN  letter_privilege like  '%X*%' and privilege_type = 'EXECUTE'   then true
		WHEN  letter_privilege like  '%U*%' and privilege_type = 'USAGE'   then true
		WHEN  letter_privilege like  '%s*%' and privilege_type = 'SET'   then true
		WHEN  letter_privilege like  '%A*%' and privilege_type = 'ALTER SYSTEM'   then true
	else  
		false
	end as is_grantable 
from
(


	select 
			schema
			,object_name
			,object_type
			,grantor
			,grantee
			,letter_privilege
			,split_part(privilege_type, ',', generate_series(1, length(privilege_type) - length(replace(privilege_type, ',', '')) + 1))	as privilege_type	--- 
	from 
	(

		select 
			schema
			,object_name
			,object_type
			,grantor
			,case  when grantee = '' THEN 'PUBLIC' else grantee END as grantee
			,letter_privilege	
			,TRIM(
					BOTH ',' FROM
					( 
						CASE  WHEN letter_privilege like '%r%' THEN 'SELECT,' ELSE '' END || 
						CASE  WHEN letter_privilege like '%a%' THEN 'INSERT,' ELSE '' END || 
						CASE  WHEN letter_privilege like '%w%' THEN 'UPDATE,' ELSE '' END || 
						CASE  WHEN letter_privilege like '%d%' THEN 'DELETE,'ELSE '' END || 
						CASE  WHEN letter_privilege like '%D%' THEN 'TRUNCATE,' ELSE '' END || 
						CASE  WHEN letter_privilege like '%x%' THEN 'REFERENCES,'ELSE '' END || 
						CASE  WHEN letter_privilege like '%t%' THEN 'TRIGGER,'ELSE '' END || 
						CASE  WHEN letter_privilege like '%C%' THEN 'CREATE,' ELSE '' END || 
						CASE  WHEN letter_privilege like '%c%' THEN 'CONNECT,' ELSE '' END || 
						CASE  WHEN letter_privilege like '%T%' THEN 'TEMPORARY,' ELSE '' END || 
						CASE  WHEN letter_privilege like '%X%' THEN 'EXECUTE,' ELSE '' END || 
						CASE  WHEN letter_privilege like '%U%' THEN 'USAGE,' ELSE '' END || 
						CASE  WHEN letter_privilege like '%s%' THEN 'SET,' ELSE '' END || 
						CASE  WHEN letter_privilege like '%A%' THEN 'ALTER SYSTEM,' ELSE '' END   
					)
				) as privilege_type 
		from(

				SELECT 
				schema
				,object_name
				,object_type
				,split_part( split_part(array_to_string(relacl , ',') , ',', generate_series(1, length(array_to_string(relacl , ',') ) - length( replace(array_to_string(relacl , ',') , ',', '')) + 1))  , '/', 2) AS grantor
				,split_part(split_part( split_part(array_to_string(relacl , ',') , ',', generate_series(1, length(array_to_string(relacl , ',') ) - length(replace(array_to_string(relacl , ',') , ',', '')) + 1))  , '/', 1), '=', 1) as grantee 
				,split_part(split_part( split_part(array_to_string(relacl , ',') , ',', generate_series(1, length(array_to_string(relacl , ',') ) - length(replace(array_to_string(relacl , ',') , ',', '')) + 1))  , '/', 1), '=', 2) as letter_privilege  
				FROM 
				(  
				
				/* AQUI VAN LOS ACL DE LOS OBJETOS  */
				 
				  ---- ESQUEMAS 
				select null as schema, nspname as object_name, 'SCHEMA' as object_type, nspacl  AS relacl FROM pg_namespace where not nspname in( 'information_schema','pg_catalog') and nspacl is not null 
				 
 

				)  as v 

			) as a 
	) as a 

) as a 
where  not grantee in('postgres','PUBLIC')   
		and pg_catalog.set_config('client_encoding', current_setting('server_encoding'), true) is not null
;
```

###
```sql

```

###
```sql

```

###
```sql

```

###
```sql

```

###
```sql

```
