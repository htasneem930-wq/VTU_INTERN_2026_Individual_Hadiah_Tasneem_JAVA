# Chapter 3: User Authentication (Login & Sign Up)

## 3.1 Introduction

Authentication is an important feature in any web application. It ensures that only authorized users can access the system.

In the **EduTrack Student Management System**, authentication is implemented using **Spring Boot**, **Spring Security**, **MySQL**, and **HTML + JavaScript**.

This module allows users to:

* Register (Sign Up)
* Login (Sign In)
* Access the dashboard after successful login

Passwords are securely stored using **BCrypt hashing**.

---

# 3.2 Technologies Used

| Technology           | Purpose                              |
| -------------------- | ------------------------------------ |
| Spring Boot          | Backend framework                    |
| Spring Security      | Authentication & password encryption |
| MySQL                | Database                             |
| HTML/CSS             | Frontend interface                   |
| JavaScript Fetch API | API communication                    |
| JPA/Hibernate        | Database interaction                 |

---

# 3.3 Database Design

A **users table** is created to store user credentials.

## SQL Table Creation

```sql
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    email VARCHAR(100) UNIQUE NOT NULL,
    password VARCHAR(255) NOT NULL,
    role VARCHAR(20) DEFAULT 'STUDENT'
);
```

### Table Description

| Column   | Description                 |
| -------- | --------------------------- |
| id       | Unique user ID              |
| email    | User email (unique)         |
| password | Encrypted password          |
| role     | User role (STUDENT / ADMIN) |

---

# 3.4 Project Structure

```
edutrack
│
├── controller
│   └── AuthController.java
│
├── model
│   └── Users.java
│
├── repository
│   └── UserRepository.java
│
├── config
│   └── SecurityConfig.java
│
└── resources
    └── static
        └── index.html
```

---
You just need to add the **Spring Security dependency** in your `pom.xml`.

## Maven Dependency for Spring Security

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

### Where to add it

Open **`pom.xml`** and place it inside the `<dependencies>` section.

Example:

```xml
<dependencies>

    <!-- Spring Boot Web -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <!-- Spring Data JPA -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>

    <!-- MySQL Driver -->
    <dependency>
        <groupId>com.mysql</groupId>
        <artifactId>mysql-connector-j</artifactId>
    </dependency>

    <!-- Spring Security -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>

</dependencies>
```

### After adding dependency

Do this once:

1. **Save `pom.xml`**
2. **Reload Maven project** in IntelliJ / Eclipse
3. Or run:

```bash
mvn clean install
```

This will download Spring Security automatically.

### Why we use it

Spring Security provides:

* Authentication
* Authorization
* Password encryption using **BCrypt**
* Protection against attacks like **CSRF**

Your code already uses it here:

```java
BCryptPasswordEncoder passwordEncoder = new BCryptPasswordEncoder();
```

So this dependency is required.

---

If you want, I can also give you a **perfect Chapter-3 GitHub documentation (very clean, screenshot ready, teacher-impressing)** for your **EduTrack project**.


# 3.5 User Model

The **Users entity** represents the `users` table in the database.

```java
package com.edutrack.edutrack.model;

import jakarta.persistence.*;

@Entity
@Table(name = "users")
public class Users {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;

    @Column(unique = true, nullable = false)
    private String email;

    @Column(nullable = false)
    private String password;

    @Column(nullable = false)
    private String role;

    public Users() {}

    public Users(String email, String password, String role) {
        this.email = email;
        this.password = password;
        this.role = role;
    }

    public int getId() { return id; }
    public void setId(int id) { this.id = id; }

    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }

    public String getPassword() { return password; }
    public void setPassword(String password) { this.password = password; }

    public String getRole() { return role; }
    public void setRole(String role) { this.role = role; }
}
```

---

# 3.6 User Repository

The repository interacts with the database using **Spring Data JPA**.

```java
package com.edutrack.edutrack.repository;

import org.springframework.data.jpa.repository.JpaRepository;
import com.edutrack.edutrack.model.Users;
import java.util.Optional;

public interface UserRepository extends JpaRepository<Users, Integer> {
    Optional<Users> findByEmail(String email);
}
```

### Purpose

* Retrieve user by email
* Save new users
* Perform database operations

---

# 3.7 Authentication Controller

The **AuthController** handles signup and login requests.

```java
package com.edutrack.edutrack.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.web.bind.annotation.*;

import com.edutrack.edutrack.model.Users;
import com.edutrack.edutrack.repository.UserRepository;

import java.util.Optional;

@RestController
@CrossOrigin
@RequestMapping("/auth")
public class AuthController {

    @Autowired
    private UserRepository userRepository;

    @Autowired
    private BCryptPasswordEncoder passwordEncoder;

    @PostMapping("/signup")
    public String signup(@RequestBody Users user) {

        Optional<Users> existingUser = userRepository.findByEmail(user.getEmail());

        if (existingUser.isPresent()) {
            return "Email already registered!";
        }

        user.setPassword(passwordEncoder.encode(user.getPassword()));
        user.setRole("STUDENT");

        userRepository.save(user);

        return "User registered successfully!";
    }

    @PostMapping("/signin")
    public String signin(@RequestBody Users user) {

        Optional<Users> existingUser = userRepository.findByEmail(user.getEmail());

        if (existingUser.isEmpty()) {
            return "User not found!";
        }

        boolean matches = passwordEncoder.matches(
                user.getPassword(),
                existingUser.get().getPassword()
        );

        if (matches) {
            return "Login successful!";
        } else {
            return "Invalid password!";
        }
    }
}
```

---

# 3.8 Spring Security Configuration

Spring Security is configured to allow authentication endpoints.

```java
package com.edutrack.edutrack.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
public class SecurityConfig {

    @Bean
    public BCryptPasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {

        http
            .csrf(csrf -> csrf.disable())
            .authorizeHttpRequests(auth -> auth
                    .requestMatchers("/auth/**").permitAll()
                    .anyRequest().permitAll()
            );

        return http.build();
    }
}
```

### Configuration Explanation

| Feature               | Description                                      |
| --------------------- | ------------------------------------------------ |
| CSRF Disabled         | Allows API requests from frontend                |
| /auth/** permitted    | Signup & login accessible without authentication |
| BCryptPasswordEncoder | Encrypts passwords                               |

---

# 3.9 Frontend Implementation

The frontend uses **HTML, CSS, and JavaScript** with the **Fetch API**.

### Key Features

* User Sign Up
* User Login
* Dashboard access after login
* Student list display
* Add new student

---

## index.html

```html
<!-- shortened explanation since your code is long -->
<!-- keep your full HTML code here exactly -->
```

*(Insert your full HTML code here exactly as used in the project.)*

---

# 3.10 Authentication Flow

```
User enters email and password
        │
        ▼
Frontend sends request to backend API
        │
        ▼
Spring Boot AuthController processes request
        │
        ▼
Password encrypted using BCrypt
        │
        ▼
Data stored or validated from MySQL database
        │
        ▼
Response sent to frontend
```

---

# 3.11 API Endpoints

| Method | Endpoint     | Description           |
| ------ | ------------ | --------------------- |
| POST   | /auth/signup | Register new user     |
| POST   | /auth/signin | Login user            |
| GET    | /students    | Retrieve student list |
| POST   | /students    | Add new student       |

---

# 3.12 Output

### Login Page

Users enter email and password to access the system.

### Signup

New users can register by providing email and password.

### Output Snapshot 1:

<img width="1208" height="685" alt="image" src="https://github.com/user-attachments/assets/22ca7dcd-32a1-4707-ae4d-c12de32d958b" />

### Dashboard

After successful login, users can view and add student details.


### After Successful LOGIN/SIGNUP users are directed to Dashboard

### Output Snapshot 2:

<img width="698" height="527" alt="image" src="https://github.com/user-attachments/assets/659f5284-2a83-4dc3-831b-531641450099" />


---

# 3.13 Summary

In this chapter, the **User Authentication System** was implemented using Spring Boot and Spring Security.

The module provides secure signup and login functionality with encrypted password storage using BCrypt. The frontend communicates with backend APIs using the Fetch API to authenticate users and provide access to the EduTrack dashboard.



