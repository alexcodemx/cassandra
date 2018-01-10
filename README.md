# Cassandra
**Alejandro García Aparicio**

Basándonos en el modelo de datos de Tweetssandra visto en clase, implemente las siguientes operaciones utilizando CQL:

 0. Creación de la Base de Datos
```
    CREATE KEYSPACE tweetssandra WITH REPLICATION = { 'class' : 'SimpleStrategy', 'replication_factor' : '1' }; 
    
    USE tweetssandra; 
    
    CREATE TABLE users ( username text PRIMARY KEY, password text ); 
    
    CREATE TABLE friends ( username text, friend text, since timestamp, PRIMARY KEY (username, friend) );
    
    CREATE TABLE followers ( username text, follower text, since timestamp, PRIMARY KEY (username, follower) );
    
    CREATE TABLE tweets ( tweet_id uuid PRIMARY KEY, username text, body text );
    
    CREATE TABLE userline ( username text, time timeuuid, body text, tweet_id uuid, PRIMARY KEY (username, time) ) WITH CLUSTERING ORDER BY (time DESC); 
    
    CREATE TABLE timeline ( user_id text, tweet_id uuid, author text, body text, PRIMARY KEY (user_id, tweet_id) );
    
    CREATE TABLE page_view_counts (counter_value counter, url_name varchar, page_name varchar, PRIMARY KEY (url_name, page_name) ); 
    
    UPDATE page_view_counts SET counter_value = counter_value + 1 WHERE url_name='www.miPagina.com' AND page_name='home';
    
    INSERT INTO followers (username, follower, since) VALUES ('gwashington', 'gmason', '2013-01-28 15:30'); 
    
    INSERT INTO followers (username, follower, since) VALUES ('gwashington', 'ahamiltom', '2013-01-27 21:15');
    
    ALTER TABLE users ADD name text;
    
    INSERT INTO users (username, password, name) VALUES ('egarcia', 'pswegarcia', 'Eva Garcia'); 
    
    INSERT INTO users (username, password) VALUES ('gmason', 'pswgmason'); 
    
    UPDATE users SET name = 'George Mason' WHERE username='gmason';
    
    INSERT INTO users (username, password, name) VALUES ('pperez', 'pswpperez', 'Pepe Perez');
    
    INSERT INTO users (username, password, name) VALUES ('jgomez', 'pswjgomez', 'Juan Gomez');
    
    INSERT INTO tweets(tweet_id, username, body) VALUES(d38af38c-b2fa-4fc2-ad81-fd68eb0cd08b, 'pperez', 'Este es mi primer tweet');
    
    INSERT INTO userline(username, time, body, tweet_id) VALUES('pperez', 1d7d544c-f239-11e7-97bf-6f6e6c696e65, 'Este es mi primer tweet', d38af38c-b2fa-4fc2-ad81-fd68eb0cd08b);
    
    INSERT INTO tweets(tweet_id, username, body) VALUES(2f56264d-a865-407a-9fdf-f500dc41c624, 'pperez', 'Este es mi segundo tweet');
    
    INSERT INTO userline(username, time, body, tweet_id) VALUES('pperez', 7d51451a-f598-11e7-95a7-6f6e6c696e65, 'Este es mi segundo tweet', 2f56264d-a865-407a-9fdf-f500dc41c624);
    
    INSERT INTO followers(username, follower, since) VALUES('pperez', 'jgomez', '2018-01-15 21:00');
    
    INSERT INTO followers(username, follower, since) VALUES('pperez', 'egarcia', '2018-01-15 20:00');
    
    INSERT INTO friends(username, friend, since) VALUES('jgomez', 'pperez', '2018-01-15 21:00');
    
    INSERT INTO friends(username, friend, since) VALUES('egarcia', 'pperez', '2018-01-15 20:00');
```

 1. Actualizar la familia de columnas “users” para hacer incluir las direcciones de correo de los usuarios.
```
    ALTER TABLE users ADD emails set<text>;
```
 2. Realizar las modificaciones oportunas para permitir que un usuario
comente el tweet de otro.
> Creación de la tabla **comments**
```
  CREATE TABLE comments ( tweet_id uuid, comment_id uuid, tweet_username text, tweet_name text, tweet_body text, comment_username text, comment_name text, comment_body text, tweet_time timeuuid, PRIMARY KEY (tweet_name, tweet_time) );
```
> Inserción de comentarios
```
  INSERT INTO tweets(tweet_id, username, body) VALUES(203145bd-c7c5-4ccf-a561-aea62dd2a7cc, 'jgomez', 'Felicidades y bienvenido');
  INSERT INTO userline(username, time, body, tweet_id) VALUES('jgomez', eff08a42-f58c-11e7-8549-6f6e6c696e65, 'Felicidades y bienvenido', 203145bd-c7c5-4ccf-a561-aea62dd2a7cc);
  INSERT INTO comments( tweet_id, comment_id, tweet_username, tweet_name, tweet_body, comment_username, comment_name, comment_body, tweet_time) VALUES(d38af38c-b2fa-4fc2-ad81-fd68eb0cd08b, 203145bd-c7c5-4ccf-a561-aea62dd2a7cc, 'pperez', 'Pepe Perez', 'Este es mi primer tweet', 'jgomez', 'Juan Gomez', 'Felicidades y bienvenido', 1d7d544c-f239-11e7-97bf-6f6e6c696e65);

  INSERT INTO tweets(tweet_id, username, body) VALUES(fac512af-1dae-4903-a3c6-51f699a1ffc1, 'egarcia', 'Te has tardado');
  INSERT INTO userline(username, time, body, tweet_id) VALUES('egarcia', 8eb2487c-f598-11e7-b29f-6f6e6c696e65, 'Te has tardado', fac512af-1dae-4903-a3c6-51f699a1ffc1);
  INSERT INTO comments( tweet_id, comment_id, tweet_username, tweet_name, tweet_body, comment_username, comment_name, comment_body, tweet_time) VALUES(d38af38c-b2fa-4fc2-ad81-fd68eb0cd08b, fac512af-1dae-4903-a3c6-51f699a1ffc1, 'pperez', 'Pepe Perez', 'Este es mi segundo tweet', 'egarcia', 'Eva Garcia', 'Gracias por mantenernos informados!', 7d51451a-f598-11e7-95a7-6f6e6c696e65);
```
 3. Visualizar todos los tweets que Pepe Pérez haya realizado durante el mes de enero junto con sus respectivos comentarios y autores de los mismos.
```
  SELECT todate(tweet_time) AS fecha, tweet_name AS usuario_tweet, tweet_body AS Tweet, comment_name AS usuario_comentario, comment_body AS comentario FROM comments WHERE tweet_name = 'Pepe Perez' AND tweet_time <= minTimeuuid('2018-01-31 23:59+0000') AND tweet_time >= maxTimeuuid('2018-01-01 00:00+0000');
```
 4. Mostrar todos los followers de un determinado usuario.
> Modificacion de la tabla **followers**
``` 
  ALTER TABLE followers ADD username_name text;
  ALTER TABLE followers ADD follower_name text;
  UPDATE followers SET username_name = 'Pepe Perez', follower_name = 'Juan Gomez' WHERE username = 'pperez' AND follower='jgomez';
  UPDATE followers SET username_name = 'Pepe Perez', follower_name = 'Eva Garcia' WHERE username = 'pperez' AND follower='egarcia';
```
> Query para seleccionar los seguidores de Pepe Perez
```
  SELECT follower_name, follower FROM followers WHERE username= 'pperez';
```
 5. Mostrar el nombre de usuario de todas las personas a las que sigue Juan Gómez (realice las modificaciones oportunas en el modelo de datos, si fueran necesarias).
> Modificacion de la tabla **friends**
```
  ALTER TABLE friends ADD username_name text;
  ALTER TABLE friends ADD friend_name text;
  UPDATE friends SET username_name = 'Juan Gomez', friend_name = 'Pepe Perez' WHERE username = 'jgomez' AND friend='pperez';
  UPDATE friends SET username_name = 'Eva Garcia', friend_name = 'Pepe Perez' WHERE username = 'egarcia' AND friend='pperez';
```
> Query para seleccionar los amigos de Juan Gomez
```
  SELECT friend_name, friend FROM Friends WHERE username= 'jgomez';
```
