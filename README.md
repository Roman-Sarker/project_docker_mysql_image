If you haven't pulled MySQL in your local Docker yet, follow these **step-by-step** instructions to **download, run, and verify** MySQL in Docker.

---

### **Step 1: Pull MySQL Docker Image**
Run the following command to pull MySQL **version 8** from Docker Hub:

```bash
docker pull mysql:8
```
- This will **download MySQL** into your local Docker environment.
- You can verify the image using:

  ```bash
  docker images
  ```

  You should see something like:

  ```
  REPOSITORY   TAG     IMAGE ID      CREATED       SIZE
  mysql        8       abc123xyz     2 days ago    500MB
  ```

---

### **Step 2: Run MySQL as a Docker Container**
Now, start a **MySQL container** with the following command:

```bash
docker run --name mysql_container -e MYSQL_ROOT_PASSWORD=root -e MYSQL_DATABASE=employee_db -p 3307:3306 -d mysql:8
```

**Explanation of flags:**
- `--name mysql_container` → Container name is `mysql_container`
- `-e MYSQL_ROOT_PASSWORD=root` → Set **root password**
- `-e MYSQL_DATABASE=employee_db` → Create a database named **`employee_db`**
- `-p 3307:3306` → Maps **MySQL's port 3306** to **host's port 3307**
- `-d` → Runs the container in **detached mode**
- `mysql:8` → Uses the **MySQL 8** image

**Verify if the container is running:**
```bash
docker ps
```
You should see:

```
CONTAINER ID   IMAGE    COMMAND                  PORTS                    STATUS        NAMES
abc123xyz      mysql:8  "docker-entrypoint..."   0.0.0.0:3307->3306/tcp    Up X mins     mysql_container
```

---

### **Step 3: Connect to MySQL in Docker**
To log into MySQL inside the container:

```bash
docker exec -it mysql_container mysql -uroot -proot
```

Now, run:

```sql
SHOW DATABASES;
```

You should see `employee_db` listed.

---

### **Step 4: Connect MySQL from Your Machine (Optional)**
If you want to connect to MySQL using a GUI tool like **MySQL Workbench**, use:

- **Host**: `localhost`
- **Port**: `3307`
- **Username**: `root`
- **Password**: `root`
- **Database**: `employee_db`

---

### **Step 5: Stop and Restart MySQL Container (If Needed)**
If MySQL is not running, start it again:

```bash
docker start mysql_container
```

To stop it:

```bash
docker stop mysql_container
```

To remove it completely:

```bash
docker rm mysql_container
docker rmi mysql:8  # (removes the image)
```

---

### **Next Step: Use MySQL in Your Spring Boot Application**
Since MySQL is now running in **Docker**, update `application.properties` to:

```properties
spring.datasource.url=jdbc:mysql://localhost:3307/employee_db
spring.datasource.username=root
spring.datasource.password=root
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.jpa.hibernate.ddl-auto=update
```

---

### **Final Step: Run Your Spring Boot Application**
Now, your MySQL database is ready! 🎉  
You can run:

```bash
mvn clean package
```

Then:

```bash
docker-compose up --build
```

This will start both **Spring Boot** and **MySQL** inside Docker.

Let me know if you face any issues! 🚀
