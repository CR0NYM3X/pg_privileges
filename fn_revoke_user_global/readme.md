 ### 🛠 Batería de Pruebas: Stress-Test del Script de Seguridad
Para asegurar que la función no "rompa" nada y se comporte de forma predecible, se ejecutaron los siguientes casos de uso:

#### 1. Gestión de Identidades (Filtro de Usuarios)

* **Prueba de "Usuarios Fantasma":** Le pasamos puros nombres de usuarios que no existen.
* *Qué pasó:* El script detectó que no había nadie en `pg_roles`, saltó la fase de conexión a las DBs y terminó limpio, avisando que no había nada que procesar.

* **Mix de Usuarios (Existentes + Inexistentes):** Mandamos una lista combinada (ej. 'admin_viejo' que sí está y 'user_test' que no).
* *Qué pasó:* Filtró los que no existen, los mandó al log de errores y siguió el proceso de revocación únicamente con los usuarios válidos.

#### 2. Flujos de Ejecución (Permisos vs. Borrado)

* **Solo Limpieza (Soft Revoke):** Se ejecutó con `p_drop_user_final => FALSE`.
* *Qué pasó:* El script entró a todas las DBs, quitó permisos, reasignó dueños y al final dejó al usuario vivo pero "desarmado".

* **Purga Total (Hard Revoke + Drop):** Se ejecutó con `p_drop_user_final => TRUE`.
* *Qué pasó:* Hizo todo el barrido de permisos y, una vez que el usuario quedó sin dependencias, le tiró el `DROP USER` sin errores de "role is being used".

#### 3. Control de Errores (Sintaxis y Resiliencia)

* **Inyección de Error de Sintaxis (Revoke Corrupto):** Modificamos un comando del array (ej. pusimos `REVOKEE` en vez de `REVOKE`) para forzar el fallo.
* *Escenario A (Solo Permisos):* El script falló en ese comando específico, lo guardó en la tabla de auditoría con el mensaje de error de Postgres y **siguió con los demás comandos**. No se detuvo.
* *Escenario B (Permisos + Borrado):* Igual que el anterior, pero al final el veredicto detectó que hubo fallos en los revokes y nos avisó que el proceso fue "exitoso con advertencias".

 
