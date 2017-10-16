# Spring Data

---

### Start with simple project dependencies

- Web
- JPA
- H2

+++

### Creating an entity

```java
@Entity
class Greeting{

	@Id
	@GeneratedValue(strategy = GenerationType.AUTO)
	Long id;

	String greeting;

	public Greeting(){} // Why, JPA, Why?

```

+++

```java
	public Greeting(String greeting){

		this.greeting = greeting;

	}

	public String getGreeting() {
		return greeting;
	}

	public void setGreeting(String greeting) {
		this.greeting = greeting;
	}
	
```

+++

```java
	@Override
	public String toString() {
		return "Greeting{" +
				"greeting='" + greeting + '\'' +
				'}';
	}
	
	@Override
	public boolean equals(Object o) {
		if (this == o) return true;
		if (o == null || getClass() != o.getClass()) return false;

		Greeting greeting1 = (Greeting) o;

		return greeting.equals(greeting1.greeting);
	}
}

```

+++

### Creating a CRUD repository

```java
interface GreetingRepository extends CrudRepository<Greeting, Long> {

}
```

+++

### Alternatively use a PagingAndSortingRepository

```java
interface GreetingRepository extends PagingAndSortingRepository<Greeting, Long> {

}
```

+++

### Interacting with the repository

```java
@Autowired
GreetingRepository greetingRepository;
```

---

# Operations

+++

### READ Operation

```java
@GetMapping("/readAll")
String readAll(){

	return greetingRepository.findAll().toString();

}
```

+++

### DELETE Operation

```java
@DeleteMapping("/deleteAll")
String deleteAll(){
	
	greetingRepository.deleteAll();
	return "Cleared all";
	
}
```

+++

### CREATE Operation

```java
@PostMapping("/add/{greeting}")
String create(@PathVariable String greeting){

	greetingRepository.save(new Greeting(greeting));
	return "Added: "+greeting;

}
```

---

# Database in Containers

+++

### Pulling the MySQL image

```bash
$ docker run mysql
Unable to find image 'mysql:latest' locally
latest: Pulling from library/mysql
85b1f47fba49: Pulling fs layer
27dc53f13a11: Pulling fs layer
095c8ae4182d: Pulling fs layer
0972f6b9a7de: Pulling fs layer
1b199048e1da: Pulling fs layer
159de3cf101e: Pulling fs layer
...
```

+++

### Running the MySQL container (easy)

```bash
$ docker run --name my-mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=password -e MYSQL_DATABASE=mydb -e MYSQL_USER=matthias -e MYSQL_PASSWORD=matthias -d mysql
```

+++

### Running the MySQL container (properly)

```bash
$  docker run --name my-mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=password -d mysql
c08307a68e95067dc5db02b98197ecc8eb40be8d508c8058383f0f8078c29235
```

+++

### Connecting to the container

```bash
$ docker exec -it mymysql bash
the input device is not a TTY.  If you are using mintty, try prefixing the command with 'winpty'

$ winpty docker exec -it mymysql bash
root@81e94de64c12:/###
```

+++

### Login to MySQL

```bash
root@81e94de64c12:/### mysql -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 4
Server version: 5.7.19 MySQL Community Server (GPL)

Copyright (c) 2000, 2017, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```

---

### Creating the database

```bash
mysql> create database mydb;
Query OK, 1 row affected (0.00 sec)

mysql> create user 'matthias'@'localhost' identified by
'matthias';
Query OK, 0 rows affected (0.00 sec)

mysql> grant all privileges on mydb.* to  'matthias'@'localhost';
Query OK, 0 rows affected, 1 warning (0.00 sec)
```

+++

### Validate

```bash
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mydb               |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.00 sec)
```

+++

### Validate grants

```
mysql> show grants;
+---------------------------------------------------------------------+
| Grants for root@localhost                                           |
+---------------------------------------------------------------------+
| GRANT ALL PRIVILEGES ON *.* TO 'root'@'localhost' WITH GRANT OPTION |
| GRANT PROXY ON ''@'' TO 'root'@'localhost' WITH GRANT OPTION        |
+---------------------------------------------------------------------+
2 rows in set (0.00 sec)

mysql> show grants for matthias;
+----------------------------------------------------+
| Grants for matthias@%                              |
+----------------------------------------------------+
| GRANT USAGE ON *.* TO 'matthias'@'%'               |
| GRANT ALL PRIVILEGES ON `mydb`.* TO 'matthias'@'%' |
+----------------------------------------------------+
2 rows in set (0.00 sec)
```

---

# Application side configuration

+++

### application.properties

```
spring.jpa.hibernate.ddl-auto=create
spring.datasource.url=jdbc:mysql://192.168.99.100:3306/mydb
spring.datasource.username=matthias
spring.datasource.password=matthias
```

+++

### Options for spring.jpa.hibernate.ddl-auto

validate: validate the schema, makes no changes to the database.
update: update the schema.
create: creates the schema, destroying previous data.
create-drop: drop the schema when the SessionFactory is closed explicitly, typically when the application is stopped.

