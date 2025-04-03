Here's a **step-by-step guide** to setting up a **Spring Boot application with MySQL in Docker using Docker Compose**, from **pulling the MySQL image** to **testing with Postman**.

---

## **Step 1: Install Docker in Local PC**
 - Download Docker
 - Install and Setup Docker Desktop in local

---

## **Step 2: Pull MySQL Docker Image into local docker**
First, pull the latest **MySQL Docker image** from Docker Hub.

```bash
docker pull mysql
```
> This downloads the latest **MySQL** image from Docker Hub.

---

## **Step 3: Run MySQL Container in Docker**
After pulling the image, run the MySQL container using the following command:

```bash
docker run --name mysql_container -e MYSQL_ROOT_PASSWORD=root -e MYSQL_DATABASE=employee_db -p 3307:3306 -d mysql
```

### **Explanation of Flags:**
- `--name mysql_container` → Names the container as **mysql_container**.
- `-e MYSQL_ROOT_PASSWORD=root` → Sets the **root password** to `root`.
- `-e MYSQL_DATABASE=employee_db` → Creates a database named **employee_db**.
- `-p 3307:3306` → Maps **container port 3306** to **host port 3307**.
- `-d mysql` → Runs the container in **detached mode**.

### **Verify MySQL Container is Running**
Run:
```bash
docker ps
```
Expected Output:
```
CONTAINER ID   IMAGE    PORTS                    STATUS        NAMES
abc123xyz      mysql    0.0.0.0:3307->3306/tcp    Up X mins    mysql_container
```

---

## **Step 4: Connect to MySQL in Docker**
Now, let's connect to MySQL inside the container.

```bash
docker exec -it mysql_container mysql -uroot -proot
```

Check the available databases:
```sql
SHOW DATABASES;
```
You should see:
```
+--------------------+
| Database          |
+--------------------+
| employee_db       |
| information_schema |
| mysql            |
| performance_schema |
| sys              |
+--------------------+

```
```sql
use employee_db;
```

```sql
select * from employees;
```
---

## **Step 5: Create a Spring Boot Project**
Create a new **Spring Boot** project using **Spring Initializr** (or manually).

### **Dependencies to Add:**
- **Spring Web**
- **Spring Data JPA**
- **MySQL Driver**
- **Lombok**

---

## **Step 6: Create an Employee Entity**
Create a new **`Employee` entity** inside your Spring Boot project.

📌 **File:** `Employee.java`
```java
package com.example.demo.model;

import jakarta.persistence.*;
import lombok.Data;

@Entity
@Data
@Table(name = "employees")
public class Employee {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String name;
    private String email;
}
```

---

## **Step 7: Create an Employee Repository**
📌 **File:** `EmployeeRepository.java`
```java
package com.example.demo.repository;

import com.example.demo.model.Employee;
import org.springframework.data.jpa.repository.JpaRepository;

public interface EmployeeRepository extends JpaRepository<Employee, Long> {
}
```

---

## **Step 8: Create an Employee Service**
📌 **File:** `EmployeeService.java`
```java
package com.example.demo.service;

import com.example.demo.model.Employee;
import com.example.demo.repository.EmployeeRepository;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class EmployeeService {

    private final EmployeeRepository employeeRepository;

    public EmployeeService(EmployeeRepository employeeRepository) {
        this.employeeRepository = employeeRepository;
    }

    public Employee saveEmployee(Employee employee) {
        return employeeRepository.save(employee);
    }

    public List<Employee> getAllEmployees() {
        return employeeRepository.findAll();
    }
}
```

---

## **Step 9: Create an Employee Controller**
📌 **File:** `EmployeeController.java`
```java
package com.example.demo.controller;

import com.example.demo.model.Employee;
import com.example.demo.service.EmployeeService;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/api/employees")
public class EmployeeController {

    private final EmployeeService employeeService;

    public EmployeeController(EmployeeService employeeService) {
        this.employeeService = employeeService;
    }

    @PostMapping
    public Employee saveEmployee(@RequestBody Employee employee) {
        return employeeService.saveEmployee(employee);
    }

    @GetMapping
    public List<Employee> getAllEmployees() {
        return employeeService.getAllEmployees();
    }
}
```

---

## **Step 10: Configure `application.properties`**
📌 **File:** `src/main/resources/application.properties`
```properties
spring.datasource.url=jdbc:mysql://mysql:3306/employee_db
spring.datasource.username=root
spring.datasource.password=root
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
```

---

## **Step 11: Create `docker-compose.yml`**
📌 **File:** `docker-compose.yml`
```yaml
version: '3.8'
services:
  mysql:
    image: mysql:latest
    container_name: mysql_container
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: employee_db
    ports:
      - "3307:3306"
    networks:
      - my_network

  spring-app:
    build: .
    container_name: spring_boot_app
    restart: always
    ports:
      - "8080:8080"
    depends_on:
      - mysql
    environment:
      SPRING_DATASOURCE_URL: jdbc:mysql://mysql:3306/employee_db
      SPRING_DATASOURCE_USERNAME: root
      SPRING_DATASOURCE_PASSWORD: root
    networks:
      - my_network

networks:
  my_network:
    driver: bridge
```
> This file ensures both MySQL and Spring Boot run in the same **Docker network**.

---

## **Step 12: Build and Run the Containers**
Inside your project directory, execute:

```bash
docker-compose up --build
```
This will:
1. **Build the Spring Boot application**.
2. **Start MySQL**.
3. **Run both services inside Docker containers**.

---

## **Step 13: Verify Containers Are Running**
Run:
```bash
docker ps
```
Expected Output:
```
CONTAINER ID   IMAGE            PORTS                     NAMES
abc123xyz      mysql:latest     0.0.0.0:3307->3306/tcp    mysql_container
def456xyz      spring-boot-app  0.0.0.0:8080->8080/tcp    spring_boot_app
```

---

## **Step 14: Test with Postman**
### **Create Employee (POST Request)**
- **URL:** `http://localhost:8080/api/employees`
- **Method:** `POST`
- **Headers:** `Content-Type: application/json`
- **Body (JSON):**
```json
{
  "name": "John Doe",
  "email": "john@example.com"
}
```
- **Expected Response:**
```json
{
  "id": 1,
  "name": "John Doe",
  "email": "john@example.com"
}
```

---

### **Fetch All Employees (GET Request)**
- **URL:** `http://localhost:8080/api/employees`
- **Method:** `GET`
- **Expected Response:**
```json
[
  {
    "id": 1,
    "name": "John Doe",
    "email": "john@example.com"
  }
]
```

---

## **Step 15: Stop Containers**
To stop the running containers:
```bash
docker-compose down
```
To restart:
```bash
docker-compose up -d
```

---

## **Conclusion**
You have successfully:
✅ Pulled MySQL Docker image  
✅ Set up a Spring Boot project with MySQL  
✅ Created a **REST API** for Employee management  
✅ Connected Spring Boot with MySQL using **Docker Compose**  
✅ Tested API endpoints using **Postman** 🎉  


