---
title: Java Spring App - Implementar el patrón repository
date: '2022-06-01'
tags: ['Java', 'Spring']
draft: false
summary: Tutorial de una aplicacion basica de Java Spring.
---

## Generar el proyecto

https://start.spring.io/

![Spring](/static/images/springinitializer.png)

link con el projecto con las dependencias iniciales.

[Link](https://start.spring.io/#!type=maven-project&language=java&platformVersion=2.7.0&packaging=jar&jvmVersion=18&groupId=ar.berserker&artifactId=spring%20app%20security&name=spring%20app%20security&description=Demo%20project%20with%20spring%20security&packageName=ar.berserker.spring%20app%20security&dependencies=web,data-jpa,mysql,data-jdbc,security))

## Descargar dependencias

Una vez abierto el proyecto, ejecutas la aplicación para que descargue las dependencias del repositorio de maven.

# Configurar el archivo application.properties

Springboot tiene con un mecanismo incorporado para la configuracion de la aplicación, que se utiliza a través del archivo application.properties.

El archivo se encuentra en la carpeta resources/application.properties.

![application.properties](/static/images/springapp/appproperties.png))

```
# Configuración de la aplicación

#link conexion con base de datos
spring.datasource.url = jdbc:mysql://localhost:3306/{{nombre base de datos}}

# nombre de usuario y contraseña
spring.datasource.username = usuario
spring.datasource.password = password123

# mostrar sentencias SQL en la consola
spring.jpa.show-sql = true

# actualizar base de datos y crear entidades
spring.jpa.hibernate.ddl-auto = update

# hibernate genera SQL optimizado
spring.jpa.properties.hibernate.dialect = org.hibernate.dialect.MySQL5Dialect

 # configuracion de seguridad
 security.basic.enabled = true
 security.basic.realm = BasicRealm
 security.basic.encoding = utf-8
 security.basic.name = usuario
 security.basic.password = password123

```

Ahora la App esta lista para conectarse a la base de datos y ejecutar las sentencias SQL.

Comando para ejecutar la aplicación

    ```
    mvn spring-boot:run
    ```

# El Patrón repositorio (repository pattern)

Creamos los siguientes paquetes:

    * entity
    * repository
    * service
    * controller

![repository](/static/images/springapp/repository.png)

# Crear una entidad

Entidad es una clase que representa una tabla en la base de datos.

En este caso creamos la clase Usuario.

```
    package ar.berserker.springapp.entity;

    import javax.persistence.Entity;
    import javax.persistence.GeneratedValue;
    import javax.persistence.GenerationType;
    import javax.persistence.Id;
    import javax.persistence.Table;

    @Entity
    @Table(name = "usuario")
    public class Usuario {

        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY)
        private Long id;

        @Column(name = "nombre")
        private String nombre;

        @Column(name = "apellido")
        private String apellido;

        @Column(name = "email")
        private String email;

        @Column(name = "password")
        private String password;

        public Long getId() {
            return id;
        }

        public void setId(Long id) {
            this.id = id;
        }

        public String getNombre() {
            return nombre;
        }

        public void setNombre(String nombre) {
            this.nombre = nombre;
        }

        public String getApellido() {
            return apellido;
        }

        public void setApellido(String apellido) {
            this.apellido = apellido;
        }

        public String getEmail() {
            return email;
        }

        public void setEmail(String email) {
            this.email = email;
        }

        public String getPassword() {
            return password;
        }

        public void setPassword(String password) {
            this.password = password;
        }

    }
```

Esta clase esta decorada con la anotación @Entity y @Table.

Tiene configurado el atributo id como primary key

```
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
```

, y el resto de atributos como columnas de la tabla.
Estos tienen configurado la anotación @Column, y sus respectivos getters y setters.

De esta forma se puede crear una entidad y persistirla en la base de datos.

# Crear el repositorio

El repositorio es una clase que se encarga de gestionar todas las operaciones de persistencia contra una tabla de la base de datos

En este caso creamos la clase UsuarioRepository.

Para hacerlo creamos una interfaz que herede de JpaRepository.

Para heredar de JpaRepository, hay que pasarle como parametro el tipo de la entidad que se va a persistir y el tipo de la clave primaria.

En este caso la clase Usuario tiene una clave primaria de tipo Long.

```
       package ar.berserker.springapp.repository;

       import ar.berserker.springapp.entity.Usuario;
       import org.springframework.data.jpa.repository.JpaRepository;

       @Repository
       public interface UsuarioRepository extends JpaRepository<Usuario, Long> {

       }
```

# Creamos el servicio

El servicio es una clase que se encarga de gestionar todas las operaciones de negocio.

Esta lleva los decoradores @Service y @Transactional.

@Service indica que la clase es un servicio.
@Transactional indica que la clase se encarga de gestionar transacciones.

A través de @Autowired se puede inyectar la dependencia del repositorio.

y con @Transactional se hace una transacción contra la base de datos.

```
        package ar.berserker.springapp.service;

        import ar.berserker.springapp.entity.Usuario;
        import ar.berserker.springapp.repository.UsuarioRepository;
        import org.springframework.beans.factory.annotation.Autowired;
        import org.springframework.stereotype.Service;
        import org.springframework.transaction.annotation.Transactional;

        @Service
        public class UsuarioService {

            @Autowired
            private UsuarioRepository usuarioRepository;

            @Transactional
            public Usuario findByEmail(String email) {
                return usuarioRepository.findByEmail(email);
            }

            @Transactional
            public Usuario save(Usuario usuario) {
                return usuarioRepository.save(usuario);
            }

        }
```

# Crear el controller

Controller es una clase que se encarga de gestionar las peticiones HTTP.

El controlador tiene los decoradores @Controller y @RequestMapping.

@Controller indica que la clase es un controlador.
@RequestMapping indica que la clase se encarga de gestionar las peticiones HTTP.

@Crossorigin es una anotación que permite que la aplicación pueda ser accedida desde cualquier origen.

@CrossOrigin(origins = "http://localhost:4200") indica que la clase se puede acceder desde el origen http://localhost:4200.

@RequestMapping(value = "/usuarios") indica que la clase se encarga de gestionar las peticiones HTTP que empiezan con /usuarios.

Creamos la clase UsuarioController.

UsuarioController se encarga de gestionar las peticiones HTTP que empiezan con /usuarios.

```
        package ar.berserker.springapp.controller;

        import ar.berserker.springapp.entity.Usuario;
        import ar.berserker.springapp.service.UsuarioService;
        import org.springframework.beans.factory.annotation.Autowired;
        import org.springframework.web.bind.annotation.CrossOrigin;
        import org.springframework.web.bind.annotation.RequestMapping;
        import org.springframework.web.bind.annotation.RestController;

        @RestController
        @CrossOrigin(origins = "http://localhost:4200")
        public class UsuarioController {

            @Autowired
            private UsuarioService usuarioService;

            @GetMapping(value = "/usuarios")
            public List<Usuario> findAll() {
                return usuarioService.findAll();
            }

            @GetMapping(value = "/usuarios/{id}")
            public Usuario findById(@PathVariable Long id) {
                return usuarioService.findById(id);
            }

            @PostMapping(value = "/usuarios")
            public Usuario save(@RequestBody Usuario usuario) {
                return usuarioService.save(usuario);
            }

            @DeleteMapping(value = "/usuarios/{id}")
            public void delete(@PathVariable Long id) {
                usuarioService.delete(id);
            }

            @PutMapping(value = "/usuarios/{id}")
            public Usuario update(@PathVariable Long id, @RequestBody Usuario usuario) {
                return usuarioService.update(id, usuario);
            }

        }
```

Peticiones HTTP

```
    GET /usuarios
    GET /usuarios/{id}
    POST /usuarios
    DELETE /usuarios/{id}
    PUT /usuarios/{id}
```

@GetMapping ejecuta una petición HTTP GET.

@PostMaping ejecuta una petición HTTP POST.

@DeleteMapping ejecuta una petición HTTP DELETE.

@PutMapping ejecuta una petición HTTP PUT.

Con estos decoradores podemos ejecutar las peticiones HTTP que queramos.

y con @PathVariable indicamos que valor es la variable que se encuentra en la petición HTTP
en este caso el valor es el id.

Este valor se pasa como parametro entre llaves.

```

        @GetMapping(value = "/usuarios/{id}")
        public Usuario findById(@PathVariable Long id) {
            return usuarioService.findById(id);
        }

```

De esta forma podemos ejecutar todas las peticiones que queramos.

# Conclusión

Ahora la aplicación está lista para ejecutar peticiones HTTP y gestionar las transacciones.
Si quisieras agregar mas entidades, seguirías el mismo proceso.
