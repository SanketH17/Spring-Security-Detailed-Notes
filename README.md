# Spring-Security — Enhanced Notes (With Analogies & Code Snippets)

> 💡 **How to use these notes:** Every major concept has a real-world analogy in a `> 🏫 Analogy` block so you can understand it intuitively, followed by theory and code. Perfect for revision and GitHub reference.

---

# 🔵 Overview of Spring Security

**Spring Security** is a **powerful and highly customizable framework** for **authentication, authorization, and security management** in Java applications, especially Spring-based applications. It provides a complete **security infrastructure** for web and method-level security.

---

## 🟢 1. Purpose of Spring Security

1. **Authentication** – Verifying who the user is (login).
2. **Authorization** – Checking if the user has access to specific resources or actions (roles/permissions).
3. **Protection** – Guarding against common security threats like:

   * CSRF (Cross-Site Request Forgery)
   * Session Fixation
   * Clickjacking
   * HTTP header injection
   * Brute force attacks
4. **Extensibility** – Can integrate with **LDAP, OAuth2, JWT, in-memory, and database-based authentication**.

> 🏫 **Analogy — The School Building:**
> Think of your application as a school building.
> - **Authentication** = Showing your school ID card at the gate to prove who you are.
> - **Authorization** = Even after entering, you can only go into rooms you're allowed in (students can't enter the principal's office without permission).
> - **Protection** = The security guard stops strangers from tailgating behind you.

---

## 🟡 2. Key Concepts

* **Authentication** – Process of validating credentials.
* **Authorization (Access Control)** – Granting/denying access to endpoints based on roles/authorities.
* **Security Context** – Holds details about the authenticated user (`SecurityContextHolder`).
* **Filter Chain** – All requests go through a **chain of filters** before reaching the controller.
* **UserDetailsService** – Interface to load user-specific data.
* **PasswordEncoder** – Interface to encode and match passwords securely.

> 🏫 **Analogy — Airport Security:**
> - **Authentication** = Passport check (proving your identity).
> - **Authorization** = Boarding pass check (proving you're allowed on *this specific* flight).
> - **Filter Chain** = The series of checkpoints: ticket counter → security scanner → gate check. Each checkpoint has one job, and you must pass all of them.
> - **SecurityContextHolder** = The tag on your luggage that travels with you throughout the airport, telling everyone who you are.
> - **PasswordEncoder** = A locked safe where passwords are stored scrambled — even airport staff can't read them directly.

---

## 🟠 3. Components

| Component                | Role                                                                                   |
| ------------------------ | -------------------------------------------------------------------------------------- |
| `FilterChainProxy`       | Delegates requests to the correct security filters.                                    |
| `Security Filter Chain`  | Collection of filters handling authentication, authorization, exception handling, etc. |
| `AuthenticationManager`  | Entry point for authentication; delegates to providers.                                |
| `AuthenticationProvider` | Implements authentication logic (e.g., database, in-memory).                           |
| `UserDetailsService`     | Loads user data from DB or memory.                                                     |
| `PasswordEncoder`        | Encodes and validates passwords.                                                       |
| `SecurityContextHolder`  | Stores the authenticated user for the current thread.                                  |

> 🏫 **Analogy — Hotel Reception Desk:**
> - **FilterChainProxy** = The hotel receptionist who decides which department handles your request.
> - **AuthenticationManager** = The shift supervisor who decides which staff member verifies your ID.
> - **AuthenticationProvider** = The specific staff member (or identity verification method) used — they might check a government ID, or a loyalty card, or a digital key.
> - **UserDetailsService** = The hotel database that looks up your reservation.
> - **PasswordEncoder** = The hotel safe — stores your room PIN scrambled so only the system can verify it.

**Quick code reference — wiring the key components:**

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    // 1️⃣ Define how users are loaded
    @Bean
    public UserDetailsService userDetailsService(UserRepository repo) {
        return username -> repo.findByUsername(username)
                .orElseThrow(() -> new UsernameNotFoundException("User not found: " + username));
    }

    // 2️⃣ Define the password encoder
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder(); // BCrypt hashes passwords with a salt
    }

    // 3️⃣ Wire the AuthenticationProvider
    @Bean
    public DaoAuthenticationProvider authenticationProvider(
            UserDetailsService uds, PasswordEncoder pe) {
        DaoAuthenticationProvider provider = new DaoAuthenticationProvider();
        provider.setUserDetailsService(uds);
        provider.setPasswordEncoder(pe);
        return provider;
    }
}
```

---

## 🔴 4. How It Works (High-Level Flow)

```
Client → Server (Tomcat) → Servlet Filter Chain → FilterChainProxy → Security Filters →
AuthenticationManager → AuthenticationProvider → UserDetailsService + PasswordEncoder →
SecurityContextHolder → DispatcherServlet → Controller → Response
```

* **Step 1:** Client sends login request.
* **Step 2:** Filters intercept request; UsernamePasswordAuthenticationFilter extracts credentials.
* **Step 3:** AuthenticationManager delegates to AuthenticationProvider.
* **Step 4:** UserDetailsService loads user, PasswordEncoder matches password.
* **Step 5:** Authenticated object stored in SecurityContextHolder.
* **Step 6:** Controller can access authenticated user.

> 🏫 **Analogy — A Job Application Process:**
> 1. You (the client) walk into the HR office (server).
> 2. The receptionist (Filter Chain) checks if you have an appointment.
> 3. The HR manager (AuthenticationManager) looks at your CV.
> 4. A specialist (AuthenticationProvider) verifies your references (UserDetailsService) and checks your background (PasswordEncoder).
> 5. If cleared, your badge (token) is stored in the system (SecurityContextHolder).
> 6. You can now enter your designated floor (Controller).

---

## 🟢 5. Advantages of Spring Security

* Highly **customizable and extensible**.
* Provides **built-in protection** for common web vulnerabilities.
* Can secure **web applications, REST APIs, and microservices**.
* Integrates with **OAuth2, JWT, LDAP, and SSO** easily.
* Follows **best practices** for password storage and session management.

---

## 🟡 6. Common Use Cases

* Login and registration systems.
* Role-based access control (RBAC).
* JWT-based stateless authentication for APIs.
* OAuth2 and SSO (Single Sign-On) integration.
* Protecting sensitive endpoints in microservices.

---

# 🔵 ① Spring Security Authentication Flow

When a client hits `/login`, Spring Security **does not immediately hit your controller**. Instead, the framework intercepts the request **at the filter layer**, checks credentials, authenticates, stores the user in context, and only then passes the request to your controller.

> 🏫 **Analogy — Entering a Government Office:**
> You don't walk straight to the officer's desk. First, security at the door scans your bag (Filter Chain). Then a receptionist checks your appointment (FilterChainProxy). Then a clerk verifies your documents (AuthenticationProvider). Only once fully verified are you shown to the officer's desk (Controller).

**High-level flow:**

```
Client (/login)
        ↓
Tomcat
        ↓
Servlet Filter Chain
        ↓
FilterChainProxy
        ↓
Security Filter Chain
        ↓
UsernamePasswordAuthenticationFilter
        ↓
AuthenticationManager (ProviderManager)
        ↓
AuthenticationProvider (DaoAuthenticationProvider)
        ↓
UserDetailsService → Fetch User
        ↓
PasswordEncoder → Match Password
        ↓
Authenticated Token Created
        ↓
SecurityContextHolder (ThreadLocal Storage)
        ↓
DispatcherServlet
        ↓
Controller
        ↓
Response

```

---

# 🟢 ② Step 1 – Client Sends Login Request

When the user submits a login form:

```
POST /login
username=rabbani
password=1234
```

* Tomcat receives the request.
* The request enters the **Servlet Filter Chain**, which is a standard chain of filters applied to every incoming request.
* **Why filters?** Security must happen **before your controller logic**. Filters intercept requests at the container level, allowing pre-processing and post-processing.

> 🏫 **Analogy:** Think of filters as layers of security at a concert venue. Before you reach the stage (controller), you pass through: parking check → ticket scan → bag search → wristband check. Each layer has one job.

**Minimal Spring Boot setup to see the filter chain in action:**

```java
// application.properties
logging.level.org.springframework.security=DEBUG
// This prints every filter that processes each request — great for learning!
```

```java
// A simple custom filter to see where your code sits in the chain
@Component
@Order(1)
public class LoggingFilter implements Filter {

    @Override
    public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain)
            throws IOException, ServletException {
        HttpServletRequest request = (HttpServletRequest) req;
        System.out.println("➡️  Incoming request: " + request.getRequestURI());
        chain.doFilter(req, res); // pass to next filter
        System.out.println("⬅️  Response sent for: " + request.getRequestURI());
    }
}
```

---

# 🟣 ③ Step 2 – FilterChainProxy & Security Filter Chain

`FilterChainProxy` is **the main entry point for Spring Security**.

* It acts as a **delegator**, sending the request to the correct `SecurityFilterChain` based on URL patterns.
* For `/login`, the chain typically includes:

```
SecurityContextPersistenceFilter → UsernamePasswordAuthenticationFilter → 
ConcurrentSessionFilter → ExceptionTranslationFilter → FilterSecurityInterceptor
```

**Key Points:**

* **SecurityContextPersistenceFilter:** Loads the `SecurityContext` (authentication info) from session or other storage.
* **UsernamePasswordAuthenticationFilter:** Extracts username/password and triggers authentication.
* **ExceptionTranslationFilter:** Converts Spring Security exceptions into proper HTTP responses (401, 403).
* **FilterSecurityInterceptor:** Checks authorization for protected endpoints.

> 🏫 **Analogy — Post Sorting Office:**
> FilterChainProxy is like a head postmaster. When mail (requests) arrives, they don't sort it themselves — they assign it to the right department (SecurityFilterChain) based on the address (URL pattern). Each department then passes the mail through its own internal checklist before delivering it.

![Filter Chain Flow Diagram](images/Spring_Security_Filter_Chain.png)

**Diagram:**

```
FilterChainProxy
        ↓
Security Filter Chain
   ┌──────────────┐
   │ Multiple     │
   │ Filters      │
   └──────────────┘
```

**Code — Configuring multiple SecurityFilterChains for different URL patterns:**

```java
@Configuration
@EnableWebSecurity
public class MultiChainSecurityConfig {

    // Chain 1: Public API — no authentication required
    @Bean
    @Order(1)
    public SecurityFilterChain publicChain(HttpSecurity http) throws Exception {
        http
            .securityMatcher("/public/**")          // applies only to /public/**
            .authorizeHttpRequests(auth -> auth.anyRequest().permitAll());
        return http.build();
    }

    // Chain 2: Admin API — requires ADMIN role
    @Bean
    @Order(2)
    public SecurityFilterChain adminChain(HttpSecurity http) throws Exception {
        http
            .securityMatcher("/admin/**")           // applies only to /admin/**
            .authorizeHttpRequests(auth -> auth.anyRequest().hasRole("ADMIN"))
            .httpBasic(Customizer.withDefaults());
        return http.build();
    }
}
```

---

# 🟡 ④ Step 3 – UsernamePasswordAuthenticationFilter

This filter is **responsible for login form authentication**.

* Extracts `username` and `password`.
* Creates an **Authentication request token**:

```java
Authentication authRequest = 
    new UsernamePasswordAuthenticationToken(username, password);
```

* Delegates authentication to **AuthenticationManager**.

**Important:** At this stage:

```
authenticated = false
```

The token only represents a **login attempt**, not a verified user.

> 🏫 **Analogy — Filling a Visitor Form at the Security Desk:**
> When you arrive at a corporate building, you fill in a visitor form (UsernamePasswordAuthenticationToken) with your name and purpose. You've handed it over, but you're NOT yet verified. The form is passed to the verification team (AuthenticationManager) — your `authenticated` flag is still `false` until they confirm your identity.

**Code — What happens inside UsernamePasswordAuthenticationFilter:**

```java
// Spring does this internally — shown for understanding
public Authentication attemptAuthentication(HttpServletRequest request,
                                             HttpServletResponse response) {

    // 1. Read credentials from the request body or form params
    String username = request.getParameter("username"); // "rabbani"
    String password = request.getParameter("password"); // "1234"

    // 2. Create an UNAUTHENTICATED token (authenticated = false)
    UsernamePasswordAuthenticationToken authRequest =
            new UsernamePasswordAuthenticationToken(username, password);

    // 3. Hand it over to the AuthenticationManager
    return this.getAuthenticationManager().authenticate(authRequest);
    // ↑ If this succeeds, authenticated = true is set on the returned token
}
```

**Code — Customizing the login URL (default is `/login`):**

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http
        .formLogin(form -> form
            .loginProcessingUrl("/auth/signin")  // 👈 custom login URL
            .defaultSuccessUrl("/dashboard")
            .failureUrl("/login?error=true")
        );
    return http.build();
}
```

---

# 🟠 ⑤ Step 4 – AuthenticationManager (ProviderManager)

Spring's `AuthenticationManager` is implemented by **ProviderManager**:

* Holds a list of `AuthenticationProvider`s.
* Loops through each provider to check `supports(authentication.getClass())`.
* Delegates authentication to the first provider that supports the token.

**Why this design?**

* Allows multiple authentication mechanisms (DB, in-memory, OAuth2, JWT, LDAP) to coexist.
* Follows **Strategy Pattern**: each provider is a separate strategy.

**Diagram:**

```
ProviderManager
    ├─ DaoAuthenticationProvider
    ├─ InMemoryAuthenticationProvider
    ├─ OAuth2AuthenticationProvider
```

> 🏫 **Analogy — Bank Loan Approval:**
> The loan manager (ProviderManager) doesn't personally verify everything. They pass your application to different specialists:
> - Home loan specialist (DaoAuthenticationProvider) for standard login
> - Fast-track specialist (InMemoryAuthenticationProvider) for test accounts
> - Partner bank specialist (OAuth2AuthenticationProvider) for social login
> The first specialist who says "I can handle this type" takes over.

**Code — How ProviderManager loops through providers internally:**

```java
// Simplified version of what ProviderManager does internally
public Authentication authenticate(Authentication authentication)
        throws AuthenticationException {

    for (AuthenticationProvider provider : this.providers) {

        // 👇 Skip providers that don't support this token type
        if (!provider.supports(authentication.getClass())) {
            continue;
        }

        try {
            // 👇 Found a matching provider — delegate to it
            Authentication result = provider.authenticate(authentication);
            if (result != null) {
                return result; // ✅ Authentication succeeded
            }
        } catch (AuthenticationException ex) {
            // Try next provider
        }
    }

    throw new ProviderNotFoundException("No provider found for: "
            + authentication.getClass().getName());
}
```

---

# 🔵 ⑥ Step 5 – DaoAuthenticationProvider

Used for database-backed login:

* Calls `UserDetailsService` to fetch user data.
* Uses `PasswordEncoder` to verify password.
* Returns a fully authenticated `UsernamePasswordAuthenticationToken` with roles and authorities.

**Code Snippet (UserDetailsService):**

```java
UserDetails user = userDetailsService.loadUserByUsername(username);
if (!passwordEncoder.matches(rawPassword, user.getPassword())) {
    throw new BadCredentialsException("Invalid password");
}
return new UsernamePasswordAuthenticationToken(user, null, user.getAuthorities());
```

**Why it exists:**

* Decouples authentication logic from the database.
* Adds abstraction for password encoding and authority mapping.

> 🏫 **Analogy — Library Card Verification:**
> When you want to borrow a book (access a resource), the librarian (DaoAuthenticationProvider) first looks up your card number in their system (UserDetailsService → DB). Then they check if your PIN matches what's stored (PasswordEncoder). If both match, they hand you a stamped library card (Authenticated Token) that lets you borrow books.

**Code — Full custom UserDetailsService:**

```java
@Service
@RequiredArgsConstructor
public class CustomUserDetailsService implements UserDetailsService {

    private final UserRepository userRepository;

    @Override
    public UserDetails loadUserByUsername(String username)
            throws UsernameNotFoundException {

        // 1️⃣ Fetch user from the database
        User user = userRepository.findByUsername(username)
                .orElseThrow(() -> new UsernameNotFoundException(
                        "No user found with username: " + username));

        // 2️⃣ Map roles to GrantedAuthority (Spring requires "ROLE_" prefix)
        List<GrantedAuthority> authorities = user.getRoles().stream()
                .map(role -> new SimpleGrantedAuthority("ROLE_" + role.name()))
                .collect(Collectors.toList());

        // 3️⃣ Return a UserDetails object Spring Security understands
        return new org.springframework.security.core.userdetails.User(
                user.getUsername(),
                user.getPassword(),   // ← must be BCrypt-encoded
                authorities
        );
    }
}
```

**Code — Password encoding at signup:**

```java
@Service
@RequiredArgsConstructor
public class UserService {

    private final UserRepository userRepository;
    private final PasswordEncoder passwordEncoder; // injected BCryptPasswordEncoder

    public User registerUser(String username, String rawPassword) {
        User user = new User();
        user.setUsername(username);

        // ✅ NEVER store plain text password
        // BCrypt automatically generates a salt and produces a 60-char hash
        user.setPassword(passwordEncoder.encode(rawPassword));
        // e.g. "$2a$10$N9qo8uLOickgx2ZMRZoMyeIjZAgcfl7p92ldGxad68LJZdL17lhWy"

        return userRepository.save(user);
    }
}
```

---

# 🟢 ⑦ Step 6 – SecurityContextHolder

Once authenticated:

* The token is stored in `SecurityContextHolder`:

```java
SecurityContextHolder.getContext().setAuthentication(authenticatedToken);
```

* `SecurityContextHolder` is a **ThreadLocal storage**, meaning each request/thread has its own authentication context.
* Controllers and other beans can access the current user:

```java
Authentication auth = SecurityContextHolder.getContext().getAuthentication();
String username = auth.getName();
```

**Advanced note:** For async or reactive applications, the context must be propagated manually.

> 🏫 **Analogy — Your Wristband at a Theme Park:**
> Once you pass the entrance check, you get a wristband (authentication token in SecurityContextHolder). For the rest of your visit, staff at every ride (controller endpoint) can just look at your wristband to know who you are and what you're allowed on — they don't re-verify your ticket each time. But importantly, each visitor has their **own wristband** — they don't share one (ThreadLocal isolation).

**Code — Accessing the current user anywhere in your app:**

```java
// 📌 Method 1: From SecurityContextHolder directly (works anywhere)
public String getCurrentUsername() {
    Authentication auth = SecurityContextHolder.getContext().getAuthentication();

    if (auth == null || !auth.isAuthenticated()) {
        return "anonymous";
    }
    return auth.getName(); // returns the username
}

// 📌 Method 2: In a @RestController via @AuthenticationPrincipal (cleaner)
@GetMapping("/profile")
public ResponseEntity<String> getProfile(
        @AuthenticationPrincipal UserDetails userDetails) {
    return ResponseEntity.ok("Logged in as: " + userDetails.getUsername());
}

// 📌 Method 3: Inject Authentication directly into the method
@GetMapping("/dashboard")
public String dashboard(Authentication authentication) {
    return "Welcome, " + authentication.getName()
           + "! Your roles: " + authentication.getAuthorities();
}
```

**Code — Propagating SecurityContext to async threads:**

```java
// ⚠️ Without this, @Async methods lose the SecurityContext
@Configuration
@EnableAsync
public class AsyncConfig implements AsyncConfigurer {

    @Override
    public Executor getAsyncExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.initialize();

        // 🔑 Wrap executor to propagate SecurityContext to child threads
        return new DelegatingSecurityContextAsyncTaskExecutor(executor);
    }
}
```

---

# 🟣 ⑧ Step 7 – DispatcherServlet → Controller

After authentication:

* The request proceeds through the remaining filters.
* Hits the **DispatcherServlet** and then the target controller.
* You can access the authenticated user either via `SecurityContextHolder` or method injection:

```java
@GetMapping("/dashboard")
public String dashboard(@AuthenticationPrincipal UserDetails user) {
    return "Welcome " + user.getUsername();
}
```

> 🏫 **Analogy:** After all security checks, you're finally shown to your seat in the cinema (controller). The usher (DispatcherServlet) escorts you to the right screen (method) based on your ticket (URL). Once seated, you can identify yourself using your ticket stub (`@AuthenticationPrincipal`).

**Code — Role-based response in controller:**

```java
@RestController
@RequestMapping("/api")
public class DashboardController {

    @GetMapping("/dashboard")
    public String dashboard(@AuthenticationPrincipal UserDetails user) {
        // Get roles of the current user
        boolean isAdmin = user.getAuthorities().stream()
                .anyMatch(a -> a.getAuthority().equals("ROLE_ADMIN"));

        if (isAdmin) {
            return "Welcome Admin " + user.getUsername() + "! You have full access.";
        }
        return "Welcome " + user.getUsername() + "! Standard access granted.";
    }
}
```

---

# 🟡 ⑨ Step 8 – Optional In-Memory Authentication

For testing or small applications:

```java
@Bean
public UserDetailsService userDetailsService() {
    UserDetails user = User.builder()
        .username("rabbani")
        .password(passwordEncoder().encode("1234"))
        .roles("USER")
        .build();
    return new InMemoryUserDetailsManager(user);
}
```

* Eliminates database dependency.
* Still uses the same authentication pipeline: filter → manager → provider → context.

> 🏫 **Analogy — A Small Office Building:**
> In a tiny 5-person office, the receptionist keeps a written list of all employees in a notepad (InMemoryUserDetailsManager) instead of a full database. Still checks the same things — name and PIN — but much simpler. Fine for a family business, not for a large corporation.

**Code — Multiple in-memory users with different roles:**

```java
@Bean
public UserDetailsService userDetailsService() {
    // 👤 Regular user
    UserDetails normalUser = User.builder()
            .username("rabbani")
            .password(passwordEncoder().encode("pass123"))
            .roles("USER")
            .build();

    // 👑 Admin user
    UserDetails adminUser = User.builder()
            .username("admin")
            .password(passwordEncoder().encode("admin123"))
            .roles("ADMIN", "USER") // Admin also has USER role
            .build();

    // 🩺 Doctor user
    UserDetails doctorUser = User.builder()
            .username("dr_smith")
            .password(passwordEncoder().encode("doc456"))
            .roles("DOCTOR")
            .build();

    return new InMemoryUserDetailsManager(normalUser, adminUser, doctorUser);
}
```

---

# 🟠 ⑩ Advanced Design Insights

1. **Filter-based security** ensures authentication before business logic.
2. **Strategy pattern in ProviderManager** makes Spring extensible.
3. **SecurityContextHolder** isolation ensures thread-safe per-request context.
4. **Delegation to DaoAuthenticationProvider** allows flexible user loading & password validation.
5. **ExceptionTranslationFilter** cleanly handles security exceptions and translates them into HTTP responses.
6. **ThreadLocal Storage** for authentication ensures performance but requires care in async threads.

> 🏫 **Analogy — Design Patterns in Spring Security:**
> - **Strategy Pattern (ProviderManager):** Like a Swiss Army knife — you pick the right tool (provider) for the job without changing how you hold the knife (the interface).
> - **Chain of Responsibility (Filter Chain):** Like an assembly line — each station (filter) does its one job and passes the item forward.
> - **ThreadLocal:** Like each cashier at a supermarket having their own cash drawer — they don't share, so there's no confusion about whose money is whose.

---

# 🔴 ⑪ Complete Flow Diagram (Expert Level)

```
Client (/login)
        ↓
Tomcat / Servlet Container
        ↓
Servlet Filter Chain
        ↓
FilterChainProxy
        ↓
Security Filter Chain
   ├─ SecurityContextPersistenceFilter (load/save)
   ├─ UsernamePasswordAuthenticationFilter (extract credentials)
   ├─ ExceptionTranslationFilter
   ├─ FilterSecurityInterceptor
        ↓
AuthenticationManager (ProviderManager)
        ↓
AuthenticationProvider (DaoAuthenticationProvider)
        ↓
UserDetailsService → fetch user from DB
        ↓
PasswordEncoder → match password
        ↓
Authenticated UsernamePasswordAuthenticationToken
        ↓
SecurityContextHolder (ThreadLocal)
        ↓
DispatcherServlet → Controller
        ↓
Response
```

---

✅ **Key Takeaways for 2+ Years Experienced Developers**

* Spring Security is **filter-based and pre-controller**.
* **AuthenticationManager + Providers** form a **strategy-based extensible system**.
* **SecurityContextHolder** is **ThreadLocal**, crucial for thread safety and async.
* Understanding **filter order** and **provider selection** is essential for debugging login/authorization issues.
* You can customize almost every layer: filters, providers, context storage, password encoding.

---

# 🔵 Expert-Level Spring Security Flow Diagram

```
┌───────────────┐
│   Client      │
│ (Browser/API) │
└───────┬───────┘
        │ POST /login
        ▼
┌───────────────┐
│   Tomcat      │
│  (Embedded)   │
└───────┬───────┘
        │
        ▼
┌───────────────────────────────┐
│ Servlet Filter Chain          │
│ (Standard Servlet Filters)    │
└───────┬──────────────────────┘
        │
        ▼
┌───────────────────────────────┐
│   FilterChainProxy             │
│  (Spring Security Delegator)   │
└───────┬──────────────────────┘
        │
        ▼
┌───────────────────────────────┐
│ Security Filter Chain          │
│ 1. SecurityContextPersistence  │
│ 2. UsernamePasswordAuthenticationFilter  │
│ 3. ExceptionTranslationFilter  │
│ 4. ConcurrentSessionFilter     │
│ 5. FilterSecurityInterceptor   │
└───────┬──────────────────────┘
        │
        ▼
┌───────────────────────────────┐
│ UsernamePasswordAuthenticationFilter │
│ - extract credentials                 │
│ - create UsernamePasswordAuthenticationToken │
│ - delegate to AuthenticationManager  │
└───────┬──────────────────────┘
        │
        ▼
┌───────────────────────────────┐
│ AuthenticationManager (ProviderManager) │
│ - holds List<AuthenticationProvider>    │
│ - loops providers:                      │
│     supports(token) → authenticate(token) │
└───────┬──────────────────────┘
        │
        ▼
┌───────────────────────────────┐
│ AuthenticationProvider          │
│ (DaoAuthenticationProvider)     │
│ - fetch UserDetailsService       │
│ - passwordEncoder.matches()      │
│ - returns authenticated token   │
└───────┬──────────────────────┘
        │
        ▼
┌───────────────────────────────┐
│ UserDetailsService / Custom    │
│ - loadUserByUsername()         │
│ - fetch user from DB / memory  │
└───────┬──────────────────────┘
        │
        ▼
┌───────────────────────────────┐
│ PasswordEncoder (BCrypt/Custom) │
│ - encode(rawPassword)           │
│ - matches(raw, encoded)         │
└───────┬──────────────────────┘
        │
        ▼
┌───────────────────────────────┐
│ SecurityContextHolder           │
│ - ThreadLocal storage           │
│ - setAuthentication(authenticatedToken) │
│ - getAuthentication()           │
└───────┬──────────────────────┘
        │
        ▼
┌───────────────────────────────┐
│ DispatcherServlet → Controller │
│ - @AuthenticationPrincipal     │
│ - SecurityContextHolder access │
└──────────────┬────────────────┘
               ▼
           Response to Client
```

---

# 🔴 Key Features in the Diagram for Experienced Developers

1. **Thread Context**

   * `SecurityContextHolder` uses **ThreadLocal**, which ensures each request has isolated authentication info.
   * In async tasks, reactive streams, or `@Async`, you need to manually propagate context.

2. **Filter Order & Responsibility**

   * SecurityContextPersistenceFilter → loads context
   * UsernamePasswordAuthenticationFilter → authentication
   * ExceptionTranslationFilter → converts exceptions to HTTP errors
   * FilterSecurityInterceptor → final authorization check

3. **ProviderManager & Strategy Pattern**

   * Holds multiple `AuthenticationProvider`s.
   * Each provider can be a separate strategy (DB, LDAP, OAuth2, JWT, Custom).

4. **PasswordEncoder**

   * Critical for secure password handling.
   * Supports BCrypt, Argon2, PBKDF2, or custom encoders.
   * DaoAuthenticationProvider calls it **after user fetch** for flexibility.

5. **Extensibility Points**

   * Filters: extend UsernamePasswordAuthenticationFilter for MFA.
   * Providers: implement custom AuthenticationProvider.
   * Context storage: can override SecurityContextRepository for stateless JWT.

6. **Exception Handling**

   * ExceptionTranslationFilter ensures Spring Security exceptions do not propagate as raw stack traces.
   * Maps exceptions to 401 (Unauthorized) or 403 (Forbidden).

**Code — Custom exception handling (401 vs 403):**

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http
        .exceptionHandling(ex -> ex
            // 🔴 401 Unauthorized — user not authenticated at all
            .authenticationEntryPoint((request, response, authException) -> {
                response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
                response.setContentType("application/json");
                response.getWriter().write("""
                    { "error": "Unauthorized", "message": "Please log in first" }
                """);
            })
            // 🔴 403 Forbidden — user authenticated but lacks permission
            .accessDeniedHandler((request, response, accessDeniedException) -> {
                response.setStatus(HttpServletResponse.SC_FORBIDDEN);
                response.setContentType("application/json");
                response.getWriter().write("""
                    { "error": "Forbidden", "message": "You don't have permission" }
                """);
            })
        );
    return http.build();
}
```

---

# 🟢 Optional: Visual Representation (ASCII + Flow Arrows)

```
[Client] 
   │
   ▼
[Tomcat] 
   │
   ▼
[Servlet Filter Chain] ──> [FilterChainProxy] 
   │                              │
   ▼                              ▼
[Security Filters] ─────────> [UsernamePasswordAuthenticationFilter]
                                       │
                                       ▼
                           [AuthenticationManager / ProviderManager]
                                       │
                                       ▼
                            [DaoAuthenticationProvider / Other Providers]
                                       │
               ┌───────────────────────┴───────────────────────┐
               ▼                                               ▼
      [UserDetailsService]                              [PasswordEncoder]
               │                                               │
               └───────────── Authenticated Token ─────────────┘
                                       │
                                       ▼
                          [SecurityContextHolder (ThreadLocal)]
                                       │
                                       ▼
                           [DispatcherServlet → Controller]
                                       │
                                       ▼
                                  Response
```

---

# 🔵 ① What is JWT (JSON Web Token)?

A **JWT (JSON Web Token)** is a **compact, self-contained, secure token format** used to transmit information between two parties as a JSON object.

It is commonly used for:

* Stateless authentication
* Authorization in REST APIs
* Microservices communication
* OAuth2 token representation

A JWT is:

* Digitally signed
* URL-safe
* Stateless (server does not store session)

> 🏫 **Analogy — A Digitally Signed Visitor Pass:**
> Traditional session auth = the office keeps a logbook and gives you a numbered paper slip. Every time you enter a room, staff call the front desk to check if your number is valid.
> JWT = the office gives you a tamper-proof digitally printed badge with your name, clearance level, and expiry baked in. Staff can verify it on the spot by checking the holographic seal — no phone call needed. The badge is **self-contained**.

---

# 🟢 ② Structure of JWT (3 Parts)

A JWT consists of **three parts**, separated by dots (`.`):

```
xxxxx.yyyyy.zzzzz
```

```
HEADER.PAYLOAD.SIGNATURE
```

Example:

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
.
eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ
.
XbPfbIHMI6arZ3Y922BhjWgQzWXcXNrz0ogtVhfEd20
```

> 🏫 **Analogy — A Sealed Envelope:**
> - **Header** = The envelope type and postage method (tells you how the letter is secured).
> - **Payload** = The letter inside (the actual message / user data).
> - **Signature** = The wax seal on the back — if anyone opens and re-seals it, the seal looks different. The server can instantly detect tampering.

---

# 🟡 ③ JWT Part 1 – Header

The header typically contains:

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

Meaning:

* `alg` → Algorithm used to sign token (HS256 = HMAC SHA-256)
* `typ` → Token type (JWT)

Then it is:

```
Base64URL Encoded
```

So:

```
Encoded Header = Base64UrlEncode(header JSON)
```

**Code — Decoding a JWT header manually:**

```java
import java.util.Base64;

public class JwtDecoder {
    public static void main(String[] args) {
        String jwt = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJyYWJiYW5pIn0.signature";

        String[] parts = jwt.split("\\.");

        // 🔍 Decode header (Base64URL → JSON)
        String headerJson = new String(Base64.getUrlDecoder().decode(parts[0]));
        System.out.println("Header: " + headerJson);
        // Output: {"alg":"HS256","typ":"JWT"}

        // 🔍 Decode payload
        String payloadJson = new String(Base64.getUrlDecoder().decode(parts[1]));
        System.out.println("Payload: " + payloadJson);
        // Output: {"sub":"rabbani"}
    }
}
```

---

# 🟠 ④ JWT Part 2 – Payload

Payload contains **claims** (data).

Example:

```json
{
  "sub": "1234567890",
  "name": "John Doe",
  "iat": 1516239022,
  "exp": 1741651200
}
```

Common claims:

| Claim | Meaning           |
| ----- | ----------------- |
| sub   | Subject (user id) |
| name  | Username          |
| iat   | Issued At         |
| exp   | Expiration Time   |
| role  | User roles        |

This is also:

```
Base64Url Encoded
```

Important:

Payload is NOT encrypted.
It is only encoded. Anyone can decode it.

> ⚠️ **Important:** Because the payload is only Base64-encoded (not encrypted), **never put sensitive data like passwords or credit card numbers in the JWT payload**. Think of it like writing on a postcard — anyone can read it, but they can't forge your signature.

**Code — Adding custom claims to JWT:**

```java
// Using io.jsonwebtoken (JJWT library)
public String generateTokenWithCustomClaims(User user) {
    return Jwts.builder()
            .subject(user.getUsername())           // "sub" claim
            .claim("userId", user.getId())         // custom claim
            .claim("roles", user.getRoles())       // custom claim — list of roles
            .claim("email", user.getEmail())       // custom claim
            .issuedAt(new Date())                  // "iat" claim
            .expiration(new Date(
                    System.currentTimeMillis() + 1000L * 60 * 60 * 24)) // 24h
            .signWith(getSecretKey())
            .compact();
}
```

---

# 🔴 ⑤ JWT Part 3 – Signature

Signature ensures **integrity and authenticity**.

For HS256:

```
HMACSHA256(
    Base64UrlEncode(header) + "." +
    Base64UrlEncode(payload),
    secretKey
)
```

Then signature is Base64Url encoded.

This guarantees:

* If payload is changed → signature changes
* If secret is wrong → signature verification fails

> 🏫 **Analogy — A Tamper-Evident Sticker:**
> Imagine a sticker placed over the seal of a package. If anyone opens the package and changes the contents, the sticker tears and can't be put back perfectly. The server's secret key is the sticker factory — only the factory can produce a valid sticker. So if someone modifies the payload and tries to re-sign, without the secret key they produce a fake sticker that the server immediately rejects.

---

# 🔵 ⑥ Visual JWT Creation Flow

```
Header JSON
     ↓
Base64UrlEncode
     ↓
Encoded Header

Payload JSON
     ↓
Base64UrlEncode
     ↓
Encoded Payload

Encoded Header + "." + Encoded Payload
     ↓
HMAC-SHA256 using Secret Key
     ↓
Base64UrlEncode
     ↓
Encoded Signature

Final JWT:
EncodedHeader.EncodedPayload.EncodedSignature
```

---

# 🟢 ⑦ JWT Authentication Flow (Client-Server)

Now let's understand the complete login + token flow.

---

## 🔐 Step 1 – Client Sends Credentials

```
POST /login
username + password
```

---

## 🔐 Step 2 – Server Validates Credentials

Spring Security:

* Uses AuthenticationManager
* Validates via DaoAuthenticationProvider
* If valid → generate JWT

---

## 🔐 Step 3 – Server Generates JWT

Server creates token:

```
Header
Payload (sub, role, iat, exp)
Secret Key
```

Signs token → sends to client.

```
Response:
{
   "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

---

## 🔐 Step 4 – Client Uses Token

For future requests:

```
GET /dashboard
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

---

## 🔐 Step 5 – Server Validates Token

Server:

1. Extract token
2. Split by `.`
3. Recalculate signature
4. Compare signatures
5. Check expiration (exp)
6. If valid → allow access

> 🏫 **Analogy — Token Flow as a Concert Wristband:**
> 1. You buy a ticket online (login) and show ID at the gate.
> 2. Staff scan your ID and give you an RFID wristband (JWT).
> 3. You keep the wristband on all night.
> 4. Every time you enter a new zone (API endpoint), the scanner reads your wristband.
> 5. The scanner checks: Is this wristband from our venue? (signature check) Is it still valid? (expiry check)
> 6. If yes → you're in. No need to go back to the gate each time.

---

# 🔴 ⑧ JWT Validation Flow (Important)

```
Incoming Request
       ↓
Extract Authorization Header
       ↓
Split token (.)
       ↓
Decode header + payload
       ↓
Recreate signature using secret
       ↓
Compare with token signature
       ↓
Check exp (expiration)
       ↓
Valid ? YES → Allow
         NO → Reject (401)
```

**Code — JWT validation logic:**

```java
@Component
public class JwtValidator {

    @Value("${jwt.secret}")
    private String secret;

    // ✅ Validate token — checks signature + expiry + username match
    public boolean validateToken(String token, UserDetails userDetails) {

        try {
            Claims claims = extractAllClaims(token);

            String username = claims.getSubject();
            Date expiration = claims.getExpiration();

            boolean usernameMatch = username.equals(userDetails.getUsername());
            boolean notExpired = expiration.after(new Date());

            return usernameMatch && notExpired;

        } catch (SignatureException e) {
            System.err.println("❌ Invalid JWT signature");
        } catch (ExpiredJwtException e) {
            System.err.println("❌ JWT token has expired");
        } catch (MalformedJwtException e) {
            System.err.println("❌ Invalid JWT token");
        }

        return false;
    }

    private Claims extractAllClaims(String token) {
        return Jwts.parser()
                .verifyWith(Keys.hmacShaKeyFor(secret.getBytes()))
                .build()
                .parseSignedClaims(token)
                .getPayload();
    }
}
```

---

# 🟣 ⑨ Access Token + Refresh Token Flow

JWT systems often use:

* Access Token (short-lived)
* Refresh Token (long-lived)

Flow:

```
Client logs in
   ↓
Server sends:
   - Access Token (15 min)
   - Refresh Token (7 days)
   ↓
Access Token expires
   ↓
Client sends refresh token
   ↓
Server verifies refresh token
   ↓
Generates new access token
```

This avoids frequent re-login.

> 🏫 **Analogy — Day Pass + Season Pass:**
> Your **access token** is like a day pass at a gym — it expires every day, so if it falls into the wrong hands, the damage is limited to one day.
> Your **refresh token** is like a season pass stored safely at home — you use it to get a new day pass each morning without having to prove your full identity again.
> If your day pass is stolen, you can revoke the season pass to completely cut off access.

**Code — Returning both tokens on login:**

```java
@PostMapping("/auth/login")
public ResponseEntity<AuthResponse> login(@RequestBody LoginRequest request) {

    Authentication auth = authenticationManager.authenticate(
            new UsernamePasswordAuthenticationToken(
                    request.getUsername(), request.getPassword()));

    User user = (User) auth.getPrincipal();

    // 🎫 Short-lived access token (15 minutes)
    String accessToken = jwtService.generateAccessToken(user);

    // 🔄 Long-lived refresh token (7 days) — stored in DB for revocation support
    String refreshToken = jwtService.generateRefreshToken(user);
    refreshTokenRepository.save(new RefreshToken(user, refreshToken));

    return ResponseEntity.ok(new AuthResponse(accessToken, refreshToken));
}

@PostMapping("/auth/refresh")
public ResponseEntity<AuthResponse> refresh(@RequestBody RefreshRequest request) {

    // 1. Find refresh token in DB
    RefreshToken stored = refreshTokenRepository
            .findByToken(request.getRefreshToken())
            .orElseThrow(() -> new RuntimeException("Refresh token not found"));

    // 2. Check it hasn't expired
    if (stored.getExpiresAt().before(new Date())) {
        refreshTokenRepository.delete(stored);
        throw new RuntimeException("Refresh token expired. Please log in again.");
    }

    // 3. Issue a new access token
    String newAccessToken = jwtService.generateAccessToken(stored.getUser());

    return ResponseEntity.ok(new AuthResponse(newAccessToken, request.getRefreshToken()));
}
```

---

# 🔵 ⑩ Stateless Nature of JWT

Traditional session-based auth:

```
Server stores session
```

JWT-based auth:

```
Server stores nothing
Token contains everything
```

That is why JWT is called:

```
Stateless Authentication
```

Better scalability for microservices.

> 🏫 **Analogy — Stamped Passport vs Entry Logbook:**
> **Session-based** = Each country (microservice) keeps a logbook of every visitor. When you cross a border, they look up your entry. If the logbook is lost, you're locked out. All countries must share one logbook (sticky sessions or shared cache).
> **JWT-based** = Your passport is stamped with all necessary visas (claims). Each border officer (microservice) can verify the passport independently by checking the government seal (signature). No shared logbook needed — perfect for distributed systems.

---

# 🟢 ⑪ Spring Security + JWT Flow

In Spring:

Instead of `UsernamePasswordAuthenticationFilter`, we use:

```
JwtAuthenticationFilter (Custom OncePerRequestFilter)
```

Flow:

```
Client Request
       ↓
JwtFilter
       ↓
Extract Bearer Token
       ↓
Validate Token
       ↓
Create Authentication Object
       ↓
SecurityContextHolder.setAuthentication()
       ↓
Controller
```

Example filter snippet:

```java
String token = request.getHeader("Authorization");

if (token != null && token.startsWith("Bearer ")) {
    token = token.substring(7);

    if (jwtService.validateToken(token)) {
        String username = jwtService.extractUsername(token);

        UserDetails userDetails = userDetailsService.loadUserByUsername(username);

        UsernamePasswordAuthenticationToken auth =
                new UsernamePasswordAuthenticationToken(
                        userDetails,
                        null,
                        userDetails.getAuthorities()
                );

        SecurityContextHolder.getContext().setAuthentication(auth);
    }
}
```

---

# 🔴 ⑫ Complete JWT System Diagram

```
CLIENT
   │
   │ 1. Send username/password
   ▼
SERVER (/login)
   │
   │ 2. Validate credentials
   │
   │ 3. Generate JWT (Header + Payload + Signature)
   ▼
CLIENT receives token
   │
   │ 4. Sends token in Authorization header
   ▼
SERVER (Every request)
   │
   │ 5. Validate signature
   │ 6. Check expiration
   │ 7. Set Authentication in SecurityContext
   ▼
Controller executes
```

---

# 🔵 ⑬ Important Security Notes (Professional Level)

* Never store sensitive data in payload.
* Always use HTTPS.
* Use short-lived access tokens.
* Use strong secret keys.
* Consider RS256 (public/private key) for microservices.
* Always validate expiration (exp).
* Blacklisting required for logout in stateless systems.

> 🏫 **Analogy — Safe Key Management:**
> Using a weak secret key is like locking your house with a key you leave under the doormat. Using RS256 (asymmetric) for microservices is like a master key system — the signing key is in a vault (private key), and each service only has a read-only copy (public key) to verify, but can never forge new tokens.

**Code — Using RS256 (asymmetric keys) for microservices:**

```java
// Generate key pair once (usually stored in a Key Management Service)
KeyPairGenerator kpg = KeyPairGenerator.getInstance("RSA");
kpg.initialize(2048);
KeyPair keyPair = kpg.generateKeyPair();

PrivateKey privateKey = keyPair.getPrivate(); // 🔐 ONLY the auth server has this
PublicKey publicKey  = keyPair.getPublic();   // 📢 All microservices get this

// Auth server signs with private key
String token = Jwts.builder()
        .subject(username)
        .signWith(privateKey, Jwts.SIG.RS256)
        .compact();

// Microservice verifies with public key — cannot forge new tokens
Jwts.parser()
        .verifyWith(publicKey)
        .build()
        .parseSignedClaims(token);
```

---

# 🔵 ① Complete JWT Authentication Flow in Spring Security (Beginner to Advanced Understanding)

Now this diagram represents something very important — **JWT-based authentication flow** in Spring Security.

Unlike form login (session-based authentication), JWT works in a **stateless** way. That means the server does not store session data. Instead, every request must carry authentication information inside a token.

---

# 🟢 ② Two Types of Requests in JWT System

From your diagram, you can see two flows:

1. **Login Request (`/login`)**
2. **Secured Requests (All other API calls)**

These two flows behave differently.

Login request is used to generate a token.
Secured requests are used to validate that token.

> 🏫 **Analogy — Two Queues at the Airport:**
> **Queue 1 (Check-in):** You show your passport and booking, receive your boarding pass (JWT).
> **Queue 2 (Boarding Gate):** You show your boarding pass directly — no need for your passport again. The boarding pass says everything needed.

---

# 🟣 ③ Step 1 – Login Request Flow (`/login`)

When a user sends:

```
POST /login
{
   "username": "rabbani",
   "password": "1234"
}
```

This request is considered a **non-secured authentication request**.

So it goes like this:

```
HTTP Request
   ↓
Security Filters
   ↓
Login Controller
```

Inside the Login Controller, you manually authenticate using:

```
AuthenticationManager
```

Example:

```java
@RestController
@RequestMapping("/auth")
public class AuthController {

    @Autowired
    private AuthenticationManager authenticationManager;

    @Autowired
    private JwtService jwtService;

    @PostMapping("/login")
    public String login(@RequestBody AuthRequest request) {

        Authentication authentication =
                authenticationManager.authenticate(
                        new UsernamePasswordAuthenticationToken(
                                request.getUsername(),
                                request.getPassword()
                        )
                );

        if (authentication.isAuthenticated()) {
            return jwtService.generateToken(request.getUsername());
        }

        throw new RuntimeException("Invalid credentials");
    }
}
```

**Code — DTO classes for clean API:**

```java
// 📥 Login request body
public record AuthRequest(String username, String password) {}

// 📤 Login response body
public record AuthResponse(
        String accessToken,
        String refreshToken,
        String username,
        List<String> roles
) {}
```

---

# 🟡 ④ AuthenticationManager During Login

When you call:

```java
authenticationManager.authenticate(...)
```

Spring does the following:

* ProviderManager loops through AuthenticationProviders
* DaoAuthenticationProvider is selected
* UserDetailsService loads user
* PasswordEncoder verifies password
* If correct → authenticated object returned

Once authentication is successful:

You generate a JWT token and return it to the client.

Example token:

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

Now the client stores this token (usually in localStorage or memory).

> ⚠️ **Security Note:** In browser apps, prefer storing tokens in `httpOnly` cookies rather than `localStorage` to prevent XSS attacks. `localStorage` is readable by any JavaScript on the page.

---

# 🟠 ⑤ Step 2 – Secured API Request Flow (With JWT)

Now suppose the client calls:

```
GET /api/dashboard
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

This is a secured request.

Now the request goes through:

```
HTTP Request
   ↓
Security Filters
   ↓
Internal Spring Security Filter Chain
   ↓
JwtAuthFilter
```

This is where JWT magic happens.

---

# 🔵 ⑥ JwtAuthFilter – The Heart of JWT Authentication

This filter runs **before UsernamePasswordAuthenticationFilter**.

Its job is:

1. Extract Authorization header
2. Extract token
3. Validate token
4. Extract username from token
5. Load user from database
6. Set authentication in SecurityContextHolder

Example:

```java
@Component
public class JwtAuthFilter extends OncePerRequestFilter {

    @Autowired
    private JwtService jwtService;

    @Autowired
    private UserDetailsService userDetailsService;

    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain filterChain)
                                    throws ServletException, IOException {

        String authHeader = request.getHeader("Authorization");

        if (authHeader == null || !authHeader.startsWith("Bearer ")) {
            filterChain.doFilter(request, response);
            return;
        }

        String token = authHeader.substring(7);
        String username = jwtService.extractUsername(token);

        if (username != null &&
            SecurityContextHolder.getContext().getAuthentication() == null) {

            UserDetails userDetails =
                    userDetailsService.loadUserByUsername(username);

            if (jwtService.validateToken(token, userDetails)) {

                UsernamePasswordAuthenticationToken authToken =
                        new UsernamePasswordAuthenticationToken(
                                userDetails,
                                null,
                                userDetails.getAuthorities()
                        );

                SecurityContextHolder.getContext()
                        .setAuthentication(authToken);
            }
        }

        filterChain.doFilter(request, response);
    }
}
```

> 🏫 **Analogy — The JwtAuthFilter is a bouncer at a club door:**
> Every person entering (every request) is stopped briefly. The bouncer checks for a wristband (Bearer token). If no wristband → let them through to the lobby but don't grant access to VIP areas. If wristband exists → scan it, verify it's genuine (validate), find their membership profile (loadUserByUsername), and clip a VIP badge to them (set SecurityContext). Then let them through the door.

---

# 🟣 ⑦ Why SecurityContextHolder Is Important Here?

Spring Security does not check JWT automatically.

It only checks:

```
Is there an Authentication object in SecurityContextHolder?
```

If yes → request is authenticated
If no → 401 Unauthorized

That's why your JwtAuthFilter must set authentication manually.

> 🏫 **Analogy:** Spring Security is like a strict doorman who only looks at one thing — does the person have a name badge on? (SecurityContext). Your JwtAuthFilter is the person who reads the token and puts the badge on the person. Without the badge, the doorman turns them away regardless.

---

# 🟡 ⑧ UsernamePasswordAuthenticationFilter in JWT Flow

In your diagram, you see:

```
UsernamePasswordAuthenticationFilter checks authentication in the SecurityContextHolder and continues the chain
```

Exactly.

If JwtAuthFilter already set authentication, then UsernamePasswordAuthenticationFilter will see that authentication exists and simply allow the request to proceed.

Then request reaches:

```
DispatcherServlet → Controller
```

Inside controller, you can get logged-in user:

```java
@GetMapping("/dashboard")
public String dashboard(Authentication authentication) {
    return "Welcome " + authentication.getName();
}
```

---

# 🟠 ⑨ JWT Service Example (Token Generation & Validation)

Here's a simple example:

```java
@Service
public class JwtService {

    private final String SECRET = "mysecretkey";

    public String generateToken(String username) {
        return Jwts.builder()
                .setSubject(username)
                .setIssuedAt(new Date())
                .setExpiration(new Date(System.currentTimeMillis() + 1000 * 60 * 60))
                .signWith(SignatureAlgorithm.HS256, SECRET)
                .compact();
    }

    public String extractUsername(String token) {
        return Jwts.parser()
                .setSigningKey(SECRET)
                .parseClaimsJws(token)
                .getBody()
                .getSubject();
    }

    public boolean validateToken(String token, UserDetails userDetails) {
        String username = extractUsername(token);
        return username.equals(userDetails.getUsername());
    }
}
```

**Code — Production-grade JwtService with proper key handling:**

```java
@Service
@Slf4j
public class JwtService {

    // ✅ Use environment variable or config server — NEVER hardcode in production
    @Value("${jwt.secret}")
    private String secret;

    @Value("${jwt.expiry.access:900000}")      // 15 min default
    private long accessTokenExpiry;

    @Value("${jwt.expiry.refresh:604800000}")   // 7 days default
    private long refreshTokenExpiry;

    private SecretKey getSigningKey() {
        // Key must be at least 256 bits for HS256
        return Keys.hmacShaKeyFor(secret.getBytes(StandardCharsets.UTF_8));
    }

    public String generateAccessToken(User user) {
        return buildToken(user, accessTokenExpiry);
    }

    public String generateRefreshToken(User user) {
        return buildToken(user, refreshTokenExpiry);
    }

    private String buildToken(User user, long expiry) {
        return Jwts.builder()
                .subject(user.getUsername())
                .claim("userId", user.getId())
                .claim("roles", user.getRoles())
                .issuedAt(new Date())
                .expiration(new Date(System.currentTimeMillis() + expiry))
                .signWith(getSigningKey())
                .compact();
    }

    public String extractUsername(String token) {
        return extractClaims(token).getSubject();
    }

    public boolean validateToken(String token, UserDetails userDetails) {
        String username = extractUsername(token);
        boolean notExpired = extractClaims(token).getExpiration().after(new Date());
        return username.equals(userDetails.getUsername()) && notExpired;
    }

    private Claims extractClaims(String token) {
        return Jwts.parser()
                .verifyWith(getSigningKey())
                .build()
                .parseSignedClaims(token)
                .getPayload();
    }
}
```

---

# 🔴 ⑩ Why JWT Is Stateless?

In session-based authentication:

* Server stores session in memory
* Client sends JSESSIONID cookie

In JWT:

* Server does NOT store anything
* Client sends token every time
* Token contains username & expiry
* Server validates token signature

That's why it scales better in microservices.

---

# 🔵 ⑪ Complete JWT Flow from Your Diagram (Connected Version)

```
Login Flow:

Client → `/login`
↓
Login Controller
↓
AuthenticationManager
↓
UserDetailsService + PasswordEncoder
↓
Generate JWT
↓
Return JWT to client

Secured Request Flow:

Client sends token in header
↓
Security Filter Chain
↓
JwtAuthFilter
↓
Extract token
↓
Extract username
↓
Validate token
↓
Load user
↓
Set Authentication in SecurityContextHolder
↓
Continue filter chain
↓
DispatcherServlet
↓
Controller
↓
Get user from SecurityContextHolder
```

---

# 🔵 JWT-Only Security — **Improved Notes with Detailed Comments (Interview Ready)**

---

# 🟢 ① WebSecurityConfig — Security Brain (With Comments)

```java
@Configuration
@RequiredArgsConstructor
@EnableMethodSecurity // 🔐 enables @PreAuthorize, @PostAuthorize etc.
public class WebSecurityConfig {

    private final JwtAuthFilter jwtAuthFilter;

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {

        http
            // ❌ Disable CSRF because we are using stateless JWT
            .csrf(csrf -> csrf.disable())

            // 🔥 VERY IMPORTANT: make Spring Security stateless
            // → no session will be created
            // → every request must carry JWT
            .sessionManagement(session ->
                session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))

            // 🔐 Authorization rules (URL level security)
            .authorizeHttpRequests(auth -> auth

                // ✅ Public endpoints (no authentication required)
                .requestMatchers("/public/**", "/auth/**").permitAll()

                // ✅ Only ADMIN role can access /admin/**
                // Spring internally checks for authority: ROLE_ADMIN
                .requestMatchers("/admin/**").hasRole("ADMIN")

                // ✅ Either DOCTOR or ADMIN can access
                .requestMatchers("/doctors/**")
                    .hasAnyRole("DOCTOR", "ADMIN")

                // 🔒 All other endpoints must be authenticated
                .anyRequest().authenticated()
            )

            // ⚠️ Exception handling for better API responses
            .exceptionHandling(ex -> ex

                // 🔴 401 → user is NOT authenticated
                .authenticationEntryPoint((req, res, e) -> {
                    res.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
                    res.getWriter().write("Unauthorized: Invalid or missing token");
                })

                // 🔴 403 → user authenticated but NO permission
                .accessDeniedHandler((req, res, e) -> {
                    res.setStatus(HttpServletResponse.SC_FORBIDDEN);
                    res.getWriter().write("Forbidden: Access denied");
                })
            )

            // 🔥 Add JWT filter BEFORE Spring's login filter
            // so token is validated early in filter chain
            .addFilterBefore(jwtAuthFilter, UsernamePasswordAuthenticationFilter.class);

        return http.build();
    }

    // 🔥 Expose AuthenticationManager bean
    // required for manual authentication in AuthService
    @Bean
    public AuthenticationManager authenticationManager(
            AuthenticationConfiguration config) throws Exception {
        return config.getAuthenticationManager();
    }

    // 🔐 Password encoder used during signup & login
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

> 🏫 **Analogy — WebSecurityConfig is the Building's Security Policy Document:**
> It defines who can enter which door (`authorizeHttpRequests`), which ID system to use (JWT), what happens if someone is caught without ID (401), and what happens if someone is verified but tries to enter a restricted zone (403). Think of it as the rulebook that all security staff follow.

**Code — Method-level security with `@PreAuthorize` (enabled by `@EnableMethodSecurity`):**

```java
@RestController
@RequestMapping("/api")
public class AdminController {

    // 🔐 Only users with ADMIN role can call this
    @PreAuthorize("hasRole('ADMIN')")
    @DeleteMapping("/users/{id}")
    public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
        userService.delete(id);
        return ResponseEntity.noContent().build();
    }

    // 🔐 User can only access their own data OR admin can access any
    @PreAuthorize("hasRole('ADMIN') or #username == authentication.name")
    @GetMapping("/users/{username}/profile")
    public ResponseEntity<UserDto> getProfile(@PathVariable String username) {
        return ResponseEntity.ok(userService.findByUsername(username));
    }

    // 🔐 Multiple roles allowed
    @PreAuthorize("hasAnyRole('DOCTOR', 'ADMIN')")
    @GetMapping("/patients")
    public ResponseEntity<List<PatientDto>> getAllPatients() {
        return ResponseEntity.ok(patientService.findAll());
    }
}
```

---

# 🟣 ② JwtAuthFilter — **Heart of JWT Authentication**

👉 Runs **on every request**

```java
@Component
@RequiredArgsConstructor
@Slf4j
public class JwtAuthFilter extends OncePerRequestFilter {

    private final AuthUtil authUtil;
    private final CustomUserDetailsService userDetailsService;

    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain filterChain)
                                    throws ServletException, IOException {

        // 🔍 Read Authorization header
        String authHeader = request.getHeader("Authorization");

        // ✅ If header missing OR not Bearer → skip filter
        if (authHeader == null || !authHeader.startsWith("Bearer ")) {
            filterChain.doFilter(request, response);
            return;
        }

        try {

            // ✂️ Extract token after "Bearer "
            String token = authHeader.substring(7);

            // 🔍 Extract username from JWT
            String username = authUtil.extractUsername(token);

            // ✅ Only authenticate if not already authenticated
            if (username != null &&
                SecurityContextHolder.getContext().getAuthentication() == null) {

                // 🔎 Load user from DB
                UserDetails userDetails =
                        userDetailsService.loadUserByUsername(username);

                // 🔐 Validate token
                if (authUtil.validateToken(token, userDetails)) {

                    // 🧠 Create authentication object
                    UsernamePasswordAuthenticationToken authentication =
                            new UsernamePasswordAuthenticationToken(
                                    userDetails,
                                    null,
                                    userDetails.getAuthorities());

                    // ✅ Store authentication in SecurityContext
                    SecurityContextHolder.getContext()
                            .setAuthentication(authentication);
                }
            }

        } catch (Exception ex) {
            // ⚠️ Token invalid / expired / malformed
            log.error("JWT validation failed: {}", ex.getMessage());
        }

        // 👉 Continue filter chain
        filterChain.doFilter(request, response);
    }
}
```

> 🏫 **Analogy — JwtAuthFilter is the night shift security guard:**
> Every person entering the building at night must show a key card (JWT). The guard doesn't care about the front desk or manager — they just scan the card (validate token), check the access database (loadUserByUsername), and either let you in or not. They don't stop the flow of people — they just add a clearance tag and let them continue walking (filterChain.doFilter).

---

# 🟡 ③ AuthUtil — JWT Utility (With Deep Comments)

```java
@Component
@Slf4j
public class AuthUtil {

    @Value("${jwt.secretKey}")
    private String jwtSecretKey;

    // 🔐 Create HMAC key from secret
    private SecretKey getSecretKey() {
        return Keys.hmacShaKeyFor(jwtSecretKey.getBytes(StandardCharsets.UTF_8));
    }

    // ===============================
    // 🔐 GENERATE JWT TOKEN
    // ===============================
    public String generateAccessToken(User user) {

        return Jwts.builder()
                .subject(user.getUsername())      // 👤 who is the user
                .claim("userId", user.getId())    // ➕ custom claim
                .issuedAt(new Date())             // ⏰ token creation time
                .expiration(new Date(
                        System.currentTimeMillis() + 1000 * 60 * 10)) // ⏳ expiry
                .signWith(getSecretKey())         // 🔐 sign token
                .compact();
    }

    // ===============================
    // 🔍 EXTRACT USERNAME
    // ===============================
    public String extractUsername(String token) {
        return getClaims(token).getSubject();
    }

    // ===============================
    // ✅ VALIDATE TOKEN
    // ===============================
    public boolean validateToken(String token, UserDetails userDetails) {

        String username = extractUsername(token);

        return username.equals(userDetails.getUsername())
                && !isTokenExpired(token);
    }

    // ===============================
    // 🔎 INTERNAL HELPERS
    // ===============================
    private Claims getClaims(String token) {
        return Jwts.parser()
                .verifyWith(getSecretKey())
                .build()
                .parseSignedClaims(token)
                .getPayload();
    }

    private boolean isTokenExpired(String token) {
        return getClaims(token).getExpiration().before(new Date());
    }
}
```

**Code — Extracting a custom claim from the token:**

```java
// Extract userId from token (custom claim added during generation)
public Long extractUserId(String token) {
    Claims claims = getClaims(token);
    return claims.get("userId", Long.class);
}

// Extract roles from token
public List<String> extractRoles(String token) {
    Claims claims = getClaims(token);
    return claims.get("roles", List.class);
}
```

---

# 🔴 ④ AuthService — Login & Signup (With Comments)

```java
@Service
@RequiredArgsConstructor
public class AuthService {

    private final AuthenticationManager authenticationManager;
    private final AuthUtil authUtil;
    private final UserRepository userRepository;
    private final PasswordEncoder passwordEncoder;
    private final PatientRepository patientRepository;

    // ===============================
    // 🔐 LOGIN FLOW
    // ===============================
    public LoginResponseDto login(LoginRequestDto request) {

        // 🔥 This triggers Spring Security authentication flow
        Authentication authentication =
                authenticationManager.authenticate(
                        new UsernamePasswordAuthenticationToken(
                                request.getUsername(),
                                request.getPassword()));

        // ✅ If credentials correct → principal contains User
        User user = (User) authentication.getPrincipal();

        // 🎫 Generate JWT
        String token = authUtil.generateAccessToken(user);

        return new LoginResponseDto(token, user.getId());
    }

    // ===============================
    // 🧾 SIGNUP FLOW
    // ===============================
    public SignupResponseDto signup(SignUpRequestDto dto) {

        // ❌ Prevent duplicate users
        if (userRepository.findByUsername(dto.getUsername()).isPresent()) {
            throw new IllegalArgumentException("User already exists");
        }

        // 🔐 Encode password before saving
        User user = User.builder()
                .username(dto.getUsername())
                .password(passwordEncoder.encode(dto.getPassword()))
                .roles(dto.getRoles())
                .providerType(AuthProviderType.EMAIL)
                .build();

        user = userRepository.save(user);

        // 👤 Create patient profile
        Patient patient = Patient.builder()
                .name(dto.getName())
                .email(dto.getUsername())
                .user(user)
                .build();

        patientRepository.save(patient);

        return new SignupResponseDto(user.getId(), user.getUsername());
    }
}
```

> 🏫 **Analogy — AuthService is the Front Desk:**
> **Login:** You show your face and PIN (credentials). The front desk calls the HR department (AuthenticationManager) to verify. If OK, they hand you a printed day-pass (JWT).
> **Signup:** You fill out a new employee form. The desk checks no one else has your name (duplicate check), seals your PIN in a vault (passwordEncoder), saves your details (userRepository), and sets up your workspace (patientRepository).

---

# 🟢 ⑤ Mental Flow (Interview Gold)

## 🔐 Login

```
Client → /auth/login
       → AuthenticationManager
       → UserDetailsService
       → PasswordEncoder
       → JWT generated
```

## 🔐 Secured Request

```
Client → Authorization: Bearer token
       → JwtAuthFilter
       → validate token
       → set SecurityContext
       → role check
       → Controller
```

> 🏫 **Quick Analogy Summary for Interviews:**
> - **Authentication** = Showing your ID (who are you?)
> - **Authorization** = Checking your pass level (what are you allowed to do?)
> - **JWT** = A tamper-proof badge you carry around
> - **SecurityContextHolder** = The clipboard in the security office that holds your verified identity for the duration of your visit
> - **Filter Chain** = The series of security checkpoints you pass through before reaching your destination
> - **ProviderManager** = The supervisor who assigns your identity check to the right specialist
> - **BCrypt** = The one-way vault where passwords are stored — you can check if something matches, but can never decode the original

---

> 📌 **Revision Checklist**
> - [ ] Can you explain the full filter chain flow?
> - [ ] Can you explain why SecurityContextHolder uses ThreadLocal?
> - [ ] Can you explain the difference between 401 and 403?
> - [ ] Can you explain how JWT signature prevents tampering?
> - [ ] Can you explain why payload is not encrypted?
> - [ ] Can you draw the Login flow vs Secured Request flow for JWT?
> - [ ] Can you explain Access Token vs Refresh Token?
> - [ ] Can you explain why CSRF is disabled in JWT-based apps?
