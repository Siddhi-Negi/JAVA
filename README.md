# JAVA
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.*;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.context.annotation.Bean;

import javax.persistence.*;
import java.util.List;

@SpringBootApplication
public class InventoryCrudApp {
    public static void main(String[] args) {
        SpringApplication.run(InventoryCrudApp.class, args);
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}

@Entity
class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String username;
    private String password;
    // Getters and setters
}

@Entity
class Item {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    private String description;
    private int quantity;
    // Getters and setters
}

interface UserRepository extends JpaRepository<User, Long> {
    User findByUsername(String username);
}

interface ItemRepository extends JpaRepository<Item, Long> {}

@RestController
@RequestMapping("/auth")
class AuthController {
    @Autowired
    private UserRepository userRepository;
    @Autowired
    private PasswordEncoder passwordEncoder;
   
    @PostMapping("/signup")
    public ResponseEntity<String> signup(@RequestBody User user) {
        user.setPassword(passwordEncoder.encode(user.getPassword()));
        userRepository.save(user);
        return ResponseEntity.ok("User created");
    }
   
    @PostMapping("/login")
    public ResponseEntity<String> login(@RequestBody User user) {
        User foundUser = userRepository.findByUsername(user.getUsername());
        if (foundUser != null && passwordEncoder.matches(user.getPassword(), foundUser.getPassword())) {
            return ResponseEntity.ok("Login successful");
        }
        return ResponseEntity.status(401).body("Invalid credentials");
    }
}

@RestController
@RequestMapping("/items")
class ItemController {
    @Autowired
    private ItemRepository itemRepository;
   
    @PostMapping
    public ResponseEntity<Item> createItem(@RequestBody Item item) {
        return ResponseEntity.ok(itemRepository.save(item));
    }
   
    @GetMapping
    public ResponseEntity<List<Item>> getItems() {
        return ResponseEntity.ok(itemRepository.findAll());
    }
   
    @GetMapping("/{id}")
    public ResponseEntity<Item> getItem(@PathVariable Long id) {
        return itemRepository.findById(id)
                .map(ResponseEntity::ok)
                .orElse(ResponseEntity.notFound().build());
    }
   
    @DeleteMapping("/{id}")
    public ResponseEntity<String> deleteItem(@PathVariable Long id) {
        itemRepository.deleteById(id);
        return ResponseEntity.ok("Item deleted");
    }
}
