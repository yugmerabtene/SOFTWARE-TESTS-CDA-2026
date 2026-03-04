## Chapitre 2 — Panorama des niveaux de test

### Objectifs

* Comprendre chaque niveau de test : unitaire, intégration, fonctionnel, acceptance, smoke, E2E
* Comprendre ce que chaque test doit vérifier (et ne doit pas faire)
* Savoir où placer ces tests dans un projet Java
* Voir des exemples simples en Java sur le fil rouge “authentification”
* Pour chaque test : explication du test, puis code, puis explication du code

---

# 1) Base commune du chapitre (mini-projet Auth)

On va tester un mini-parcours “connexion” :

Règles (simples)

* Si email est `null` ⇒ refus
* Si secret est `null` ⇒ refus
* Si le secret ne respecte pas la politique (minLength) ⇒ refus
* Si l’utilisateur n’existe pas ⇒ refus
* Si secret incorrect ⇒ refus
* Sinon ⇒ succès

---

## Code de production (commun à tous les tests)

### `src/main/java/model/CredentialModel.java`

```java
package model;

public class CredentialModel {

    // Attribut : identifiant (ex : "alice")
    private final String username;

    // Attribut : secret (en clair ici pour l'apprentissage des tests)
    private final String password;

    public CredentialModel(String username, String password) {
        this.username = username;
        this.password = password;
    }

    public String getUsername() {
        return username;
    }

    public String getPassword() {
        return password;
    }
}
```

Explications du code

* Objet simple, déterministe.
* Le secret en clair sert uniquement à la pédagogie des tests.

---

### `src/main/java/model/UserModel.java`

```java
package model;

public class UserModel {

    // Attribut : email (clé fonctionnelle dans ce mini-projet)
    private final String email;

    // Attribut : identifiants
    private final CredentialModel credential;

    public UserModel(String email, String username, String password) {
        this.email = email;
        this.credential = new CredentialModel(username, password);
    }

    public String getEmail() {
        return email;
    }

    public CredentialModel getCredential() {
        return credential;
    }
}
```

Explications du code

* On reste minimal : email et credential suffisent pour tester un login.

---

### `src/main/java/repository/UserRepository.java`

```java
package repository;

import java.util.Optional;
import model.UserModel;

public interface UserRepository {

    // Sauvegarde un utilisateur
    void save(UserModel user);

    // Recherche un utilisateur par email
    Optional<UserModel> findByEmail(String email);
}
```

Explications du code

* On sépare stockage et logique métier.
* `Optional` exprime “présent ou absent”.

---

### `src/main/java/repository/InMemoryUserRepository.java`

```java
package repository;

import java.util.HashMap;
import java.util.Map;
import java.util.Optional;
import model.UserModel;

public class InMemoryUserRepository implements UserRepository {

    // Stockage en mémoire : email -> user
    private final Map<String, UserModel> usersByEmail = new HashMap<>();

    @Override
    public void save(UserModel user) {
        if (user == null) {
            return;
        }
        usersByEmail.put(user.getEmail(), user);
    }

    @Override
    public Optional<UserModel> findByEmail(String email) {
        if (email == null) {
            return Optional.empty();
        }

        UserModel found = usersByEmail.get(email);
        if (found == null) {
            return Optional.empty();
        }

        return Optional.of(found);
    }
}
```

Explications du code

* Repository réel mais simple, parfait pour tests d’intégration et scénarios.

---

### `src/main/java/service/AuthenticationFailedException.java`

```java
package service;

public class AuthenticationFailedException extends RuntimeException {

    public AuthenticationFailedException(String message) {
        super(message);
    }
}
```

Explications du code

* On refusera les connexions via une exception métier explicite.

---

### `src/main/java/service/PasswordPolicyService.java`

```java
package service;

public class PasswordPolicyService {

    // Attribut : longueur minimale
    private final int minLength;

    public PasswordPolicyService(int minLength) {
        this.minLength = minLength;
    }

    public boolean isValid(String password) {
        if (password == null) {
            return false;
        }
        return password.length() >= minLength;
    }
}
```

Explications du code

* Classe idéale pour un test unitaire.

---

### `src/main/java/service/PasswordAuthenticatorService.java`

```java
package service;

import model.UserModel;

public class PasswordAuthenticatorService {

    public boolean authenticate(UserModel user, String password) {
        if (user == null) {
            return false;
        }
        if (password == null) {
            return false;
        }
        if (user.getCredential() == null) {
            return false;
        }

        // Vérifie que le secret fourni correspond au secret stocké
        return user.getCredential().getPassword().equals(password);
    }
}
```

Explications du code

* Vérification volontairement simple pour se concentrer sur les niveaux de tests.

---

### `src/main/java/service/AuthService.java`

```java
package service;

import java.util.Optional;
import model.UserModel;
import repository.UserRepository;

public class AuthService {

    // Dépendances : stockage, authenticator, politique
    private final UserRepository userRepository;
    private final PasswordAuthenticatorService authenticator;
    private final PasswordPolicyService passwordPolicy;

    public AuthService(
            UserRepository userRepository,
            PasswordAuthenticatorService authenticator,
            PasswordPolicyService passwordPolicy
    ) {
        this.userRepository = userRepository;
        this.authenticator = authenticator;
        this.passwordPolicy = passwordPolicy;
    }

    public UserModel login(String email, String password) {
        // 1) Vérification des entrées
        if (email == null) {
            throw new AuthenticationFailedException("Email manquant");
        }
        if (password == null) {
            throw new AuthenticationFailedException("Mot de passe manquant");
        }

        // 2) Règle de politique de secret
        if (!passwordPolicy.isValid(password)) {
            throw new AuthenticationFailedException("Mot de passe non conforme");
        }

        // 3) Recherche utilisateur
        Optional<UserModel> found = userRepository.findByEmail(email);
        if (found.isEmpty()) {
            throw new AuthenticationFailedException("Utilisateur introuvable");
        }

        // 4) Vérification du secret
        UserModel user = found.get();
        if (!authenticator.authenticate(user, password)) {
            throw new AuthenticationFailedException("Secret invalide");
        }

        // 5) Succès
        return user;
    }
}
```

Explications du code

* Ce service orchestre les règles de connexion.
* Il sert de base pour tests d’intégration, fonctionnels, acceptance, E2E.

---

### `src/main/java/api/AuthApi.java` (point d’entrée simulé)

```java
package api;

import model.UserModel;
import service.AuthService;
import service.AuthenticationFailedException;

public class AuthApi {

    private final AuthService authService;

    public AuthApi(AuthService authService) {
        this.authService = authService;
    }

    public String login(String email, String password) {
        // Simule une réponse d'API : OK:<email> ou KO:<raison>
        try {
            UserModel user = authService.login(email, password);
            return "OK:" + user.getEmail();
        } catch (AuthenticationFailedException ex) {
            return "KO:" + ex.getMessage();
        }
    }
}
```

Explications du code

* Pour un E2E, on part souvent d’un point d’entrée (API/UI).
* Ici, `AuthApi` est une simulation simple.

---

# 2) La pyramide de tests (logique)

* Beaucoup de tests unitaires (rapides, précis)
* Moins de tests d’intégration (plus lents)
* Encore moins de tests E2E (les plus coûteux)

Raison

* Plus un test couvre large, plus il est lent et fragile.
* Plus un test est petit, plus il est stable et diagnostique facilement.
* <img width="1280" height="720" alt="image" src="https://github.com/user-attachments/assets/6d98633a-84f7-433c-a5fe-f14b51cddc4f" />


---

# 3) Exemples concrets par niveau de test

## 3.1 Test unitaire

### Explication du test

* Objectif : tester une petite règle isolée.
* Ici : on teste uniquement `PasswordPolicyService.isValid`.
* On ne touche ni au repository, ni au service de login, ni à l’API.

### Code : `src/test/java/testunit/PasswordPolicyServiceTest.java`

```java
package testunit;

import org.junit.jupiter.api.Test;
import service.PasswordPolicyService;

import static org.junit.jupiter.api.Assertions.*;

public class PasswordPolicyServiceTest {

    @Test
    void shouldRejectTooShortPassword() {
        // Arrange
        PasswordPolicyService policy = new PasswordPolicyService(8);

        // Act
        boolean result = policy.isValid("abc");

        // Assert
        assertFalse(result);
    }

    @Test
    void shouldAcceptLongEnoughPassword() {
        PasswordPolicyService policy = new PasswordPolicyService(8);

        boolean result = policy.isValid("abcdefgh");

        assertTrue(result);
    }
}
```

Explications du code

* Le test ne dépend de rien d’autre que la classe testée.
* Rapide, stable, diagnostique immédiat.

---

## 3.2 Test d’intégration

### Explication du test

* Objectif : vérifier que plusieurs composants réels fonctionnent ensemble.
* Ici : `AuthService` + `InMemoryUserRepository` + `PasswordPolicyService` + `PasswordAuthenticatorService`.
* On teste l’assemblage et le flux du login.

### Code : `src/test/java/testintegration/AuthServiceIntegrationTest.java`

```java
package testintegration;

import model.UserModel;
import org.junit.jupiter.api.Test;
import repository.InMemoryUserRepository;
import service.AuthService;
import service.AuthenticationFailedException;
import service.PasswordAuthenticatorService;
import service.PasswordPolicyService;

import static org.junit.jupiter.api.Assertions.*;

public class AuthServiceIntegrationTest {

    @Test
    void shouldLoginSuccessfullyWithRealComponents() {
        // Arrange : repository réel en mémoire avec un utilisateur
        InMemoryUserRepository repo = new InMemoryUserRepository();
        repo.save(new UserModel("alice@example.com", "alice", "abcdefgh"));

        // Arrange : service complet avec ses dépendances réelles
        AuthService authService = new AuthService(
                repo,
                new PasswordAuthenticatorService(),
                new PasswordPolicyService(8)
        );

        // Act : on tente de se connecter
        UserModel user = authService.login("alice@example.com", "abcdefgh");

        // Assert : on vérifie le résultat observable
        assertEquals("alice@example.com", user.getEmail());
    }

    @Test
    void shouldFailWhenUserDoesNotExist() {
        InMemoryUserRepository repo = new InMemoryUserRepository();

        AuthService authService = new AuthService(
                repo,
                new PasswordAuthenticatorService(),
                new PasswordPolicyService(8)
        );

        // Act + Assert : on attend une exception
        assertThrows(AuthenticationFailedException.class, () -> {
            authService.login("missing@example.com", "abcdefgh");
        });
    }
}
```

Explications du code

* On assemble de vraies classes : ce n’est plus un test unitaire.
* On vérifie un flux complet au niveau service, sans point d’entrée API.

---

## 3.3 Test fonctionnel

### Explication du test

* Objectif : vérifier un scénario métier lisible.
* Ici : “un secret non conforme entraîne un refus explicite”.
* On teste l’effet visible : refus avec une raison.

### Code : `src/test/java/testfunctional/AuthScenarioFunctionalTest.java`

```java
package testfunctional;

import model.UserModel;
import org.junit.jupiter.api.Test;
import repository.InMemoryUserRepository;
import service.AuthService;
import service.AuthenticationFailedException;
import service.PasswordAuthenticatorService;
import service.PasswordPolicyService;

import static org.junit.jupiter.api.Assertions.*;

public class AuthScenarioFunctionalTest {

    @Test
    void scenarioShouldRejectNonCompliantPassword() {
        // Arrange
        InMemoryUserRepository repo = new InMemoryUserRepository();
        repo.save(new UserModel("alice@example.com", "alice", "abcdefgh"));

        AuthService authService = new AuthService(
                repo,
                new PasswordAuthenticatorService(),
                new PasswordPolicyService(8)
        );

        // Act + Assert : on vérifie le comportement métier (refus)
        AuthenticationFailedException ex = assertThrows(AuthenticationFailedException.class, () -> {
            authService.login("alice@example.com", "abc");
        });

        // Assert : raison attendue
        assertEquals("Mot de passe non conforme", ex.getMessage());
    }
}
```

Explications du code

* Le test est écrit comme un scénario : on prépare, on exécute, on observe.
* On vérifie une raison précise : comportement visible du système.

---

## 3.4 Test d’acceptance

### Explication du test

* Objectif : valider les critères d’acceptation d’une user story.
* On vérifie plusieurs critères, mais uniquement ceux demandés.
* Ici : user story “connexion” avec 4 critères minimaux.

### Code : `src/test/java/testacceptance/AuthAcceptanceTest.java`

```java
package testacceptance;

import model.UserModel;
import org.junit.jupiter.api.Test;
import repository.InMemoryUserRepository;
import service.AuthService;
import service.PasswordAuthenticatorService;
import service.PasswordPolicyService;

import static org.junit.jupiter.api.Assertions.*;

public class AuthAcceptanceTest {

    @Test
    void acceptanceLoginMeetsCriteria() {
        // Arrange
        InMemoryUserRepository repo = new InMemoryUserRepository();
        repo.save(new UserModel("alice@example.com", "alice", "abcdefgh"));

        AuthService authService = new AuthService(
                repo,
                new PasswordAuthenticatorService(),
                new PasswordPolicyService(8)
        );

        // Critère 1 : utilisateur absent -> refus
        assertTrue(throwsAuthFail(authService, "missing@example.com", "abcdefgh"));

        // Critère 2 : secret incorrect -> refus
        assertTrue(throwsAuthFail(authService, "alice@example.com", "zzzzzzzz"));

        // Critère 3 : secret non conforme -> refus
        assertTrue(throwsAuthFail(authService, "alice@example.com", "abc"));

        // Critère 4 : succès -> pas d'exception
        assertDoesNotThrow(() -> authService.login("alice@example.com", "abcdefgh"));
    }

    private boolean throwsAuthFail(AuthService authService, String email, String password) {
        try {
            authService.login(email, password);
            return false;
        } catch (RuntimeException ex) {
            return true;
        }
    }
}
```

Explications du code

* On vérifie plusieurs critères, mais seulement ceux qui permettent d’accepter la fonctionnalité.
* `assertDoesNotThrow` vérifie le succès.
* La méthode `throwsAuthFail` est une aide simple pour rester lisible.

---

## 3.5 Smoke test

### Explication du test

* Objectif : vérifier vite que le chemin vital minimal fonctionne.
* Ici : “un utilisateur connu peut se connecter”.
* Très court, très stable.

### Code : `src/test/java/testsmoke/SmokeAuthTest.java`

```java
package testsmoke;

import api.AuthApi;
import model.UserModel;
import org.junit.jupiter.api.Test;
import repository.InMemoryUserRepository;
import service.AuthService;
import service.PasswordAuthenticatorService;
import service.PasswordPolicyService;

import static org.junit.jupiter.api.Assertions.*;

public class SmokeAuthTest {

    @Test
    void smokeLoginReturnsOk() {
        // Arrange
        InMemoryUserRepository repo = new InMemoryUserRepository();
        repo.save(new UserModel("alice@example.com", "alice", "abcdefgh"));

        AuthService authService = new AuthService(
                repo,
                new PasswordAuthenticatorService(),
                new PasswordPolicyService(8)
        );

        AuthApi api = new AuthApi(authService);

        // Act
        String response = api.login("alice@example.com", "abcdefgh");

        // Assert
        assertEquals("OK:alice@example.com", response);
    }
}
```

Explications du code

* Le smoke test part d’un point d’entrée (`AuthApi`) : “le système répond”.
* Il teste un seul cas nominal vital.

---

## 3.6 Test E2E (simulé)

### Explication du test

* Objectif : tester un parcours complet depuis un point d’entrée.
* Ici : API simulée → service → repository → réponse.
* On garde 2 cas : succès et échec.

### Code : `src/test/java/teste2e/AuthE2ETest.java`

```java
package teste2e;

import api.AuthApi;
import model.UserModel;
import org.junit.jupiter.api.Test;
import repository.InMemoryUserRepository;
import service.AuthService;
import service.PasswordAuthenticatorService;
import service.PasswordPolicyService;

import static org.junit.jupiter.api.Assertions.*;

public class AuthE2ETest {

    @Test
    void e2eLoginSuccess() {
        // Arrange
        InMemoryUserRepository repo = new InMemoryUserRepository();
        repo.save(new UserModel("alice@example.com", "alice", "abcdefgh"));

        AuthApi api = buildApi(repo);

        // Act
        String response = api.login("alice@example.com", "abcdefgh");

        // Assert
        assertEquals("OK:alice@example.com", response);
    }

    @Test
    void e2eLoginFailure() {
        // Arrange
        InMemoryUserRepository repo = new InMemoryUserRepository();
        repo.save(new UserModel("alice@example.com", "alice", "abcdefgh"));

        AuthApi api = buildApi(repo);

        // Act
        String response = api.login("alice@example.com", "wrongpass");

        // Assert
        assertTrue(response.startsWith("KO:"));
    }

    private AuthApi buildApi(InMemoryUserRepository repo) {
        AuthService authService = new AuthService(
                repo,
                new PasswordAuthenticatorService(),
                new PasswordPolicyService(8)
        );
        return new AuthApi(authService);
    }
}
```

Explications du code

* On passe par un point d’entrée : on teste la chaîne complète.
* E2E doit rester peu nombreux et robuste.

---

# 4) Points clés à retenir

* Chaque niveau de test a une mission différente.
* Plus c’est “large” (E2E), plus c’est lent et fragile, donc moins il en faut.
* Les tests unitaires sont la base : rapides, stables, précis.
* Acceptance valide des critères métier, smoke valide le vital, E2E valide un parcours complet.
