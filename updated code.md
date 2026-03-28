# 📘 EduTrack - Attendance Module (Spring Boot)

This module handles student attendance management including:

* Adding attendance
* Fetching attendance
* Calculating attendance percentage

---

# 🚀 Step-by-Step Implementation

## 🧩 Step 1: Create Attendance Entity

Create a model class to map the attendance table.

```java
@Entity
@Table(name = "attendance")
public class Attendance {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;

    @Column(name = "student_id", nullable = false)
    private int studentId;

    private LocalDate date;

    private String status;

    // Getters and Setters
}
```

---

## 🧩 Step 2: Create Repository Layer

Create a repository interface to interact with the database.

```java
public interface AttendanceRepository extends JpaRepository<Attendance, Integer> {

    List<Attendance> findByStudentId(int studentId);
}
```

---

## 🧩 Step 3: Create Controller Layer

Expose REST APIs for attendance operations.

```java
@RestController
@RequestMapping("/attendance")
@CrossOrigin
public class AttendanceController {

    @Autowired
    private AttendanceRepository repo;

    @PostMapping("/add")
    public Attendance addAttendance(@RequestBody Attendance attendance){
        return repo.save(attendance);
    }

    @GetMapping("/all")
    public List<Attendance> getAll(){
        return repo.findAll();
    }

    @GetMapping("/student/{studentId}")
    public List<Attendance> getAttendanceByStudent(@PathVariable int studentId){
        return repo.findByStudentId(studentId);
    }

    @GetMapping("/percentage/{studentId}")
    public double getAttendancePercentage(@PathVariable int studentId){

        List<Attendance> list = repo.findByStudentId(studentId);

        if(list.isEmpty()){
            return 0;
        }

        long presentCount = list.stream()
                .filter(a -> "Present".equalsIgnoreCase(a.getStatus()))
                .count();

        return (presentCount * 100.0) / list.size();
    }
}
```

---

## 🧩 Step 4: Configure Security

Allow API access using Spring Security configuration.

```java
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

---

# 📡 API Endpoints

| Method | Endpoint                      | Description            |
| ------ | ----------------------------- | ---------------------- |
| POST   | `/attendance/add`             | Add attendance         |
| GET    | `/attendance/all`             | Get all records        |
| GET    | `/attendance/student/{id}`    | Get student attendance |
| GET    | `/attendance/percentage/{id}` | Get attendance %       |

---

# 🧪 Sample JSON

### Add Attendance

```json
{
  "studentId": 1,
  "date": "2026-03-29",
  "status": "Present"
}
```

---

# ✅ Output Example

```
Attendance Percentage: 80.0%
```

---

# 🔧 Future Improvements

* Add Service Layer (Best Practice)
* Use Enum for status (Present/Absent)
* Add Validation (`@NotNull`, `@NotBlank`)
* Implement JWT Authentication
* Add Swagger for API documentation

---

# 👨‍💻 Tech Stack

* Spring Boot
* Spring Data JPA
* MySQL
* Spring Security

---

# 📌 Author

Developed as part of EduTrack Student Management System.
