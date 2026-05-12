# GitHub Copilot Instructions — Spring PetClinic

These instructions apply to the entire repository and are intended for both the
**GitHub Copilot coding agent (cloud)** and **Copilot Chat in IDE mode**
(VS Code, JetBrains, Visual Studio). Follow them on every task in this repo.

## Project snapshot

- **Project**: Spring PetClinic — canonical Spring Boot sample app.
- **Language / runtime**: Java 17 (required for build and runtime).
- **Framework**: Spring Boot 4.0.x (`spring-boot-starter-parent` in [pom.xml](pom.xml)).
- **Stack**: Spring MVC + Thymeleaf, Spring Data JPA, Bean Validation, Caffeine cache, Spring Boot Actuator.
- **Databases**: H2 (default, in-memory), MySQL, PostgreSQL. Profiles: `mysql`, `postgres`.
- **Build tools**: Maven (`./mvnw`, primary) and Gradle (`./gradlew`). Both must keep working.
- **Frontend**: SCSS compiled to CSS via the Maven `css` profile; static assets under `src/main/resources/static`.
- **Entry point**: `org.springframework.samples.petclinic.PetClinicApplication`.
- **Main packages** under `src/main/java/org/springframework/samples/petclinic/`:
  `model/`, `owner/`, `vet/`, `system/`, plus `PetClinicApplication`, `PetClinicRuntimeHints`.
- **Tests**: JUnit 5 under `src/test/java/...`. Notable integration entry points:
  `PetClinicIntegrationTests`, `MySqlTestApplication`, `PostgresIntegrationTests`
  (Testcontainers / Docker Compose for MySQL / Postgres).
- **Container**: no `Dockerfile`; image is built via `./mvnw spring-boot:build-image`.
- **Kubernetes**: sample manifests in `k8s/` (`db.yml`, `petclinic.yml`).
- **License**: Apache 2.0. All contributions must follow DCO (see "Commits & PRs").

## Ground rules

1. **Stay minimal and focused.** Only change what the task requires. Do not
   refactor unrelated code, rename packages, reformat untouched files, or
   "modernize" code that already works.
2. **Preserve the architecture.** Spring MVC + Thymeleaf + Spring Data JPA.
   Do not introduce alternative web stacks (WebFlux, JAX-RS), templating
   engines, ORMs, or DI frameworks unless explicitly asked.
3. **Keep both build systems green.** If you touch dependencies, plugins,
   or build configuration, update **both** [pom.xml](pom.xml) and
   [build.gradle](build.gradle) so Maven and Gradle stay in sync.
4. **Java 17 baseline.** Do not use language features beyond Java 17. Do not
   bump `java.version` without an explicit request.
5. **Do not commit secrets** or environment-specific configuration. The app
   defaults to H2; profile-specific properties live in
   `src/main/resources/application-mysql.properties` and
   `application-postgres.properties`.
6. **Do not add new top-level files** (Dockerfile, CI workflows, scripts,
   docs) unless the task explicitly requires it. Prefer editing existing
   files over creating new ones.
7. **No `target/` edits.** Never modify files under `target/` — it is build
   output and is regenerated.

## Code style and conventions

- Follow the existing Spring Java Format (`io.spring.javaformat` plugin is
  enforced by the build). Run `./mvnw spring-javaformat:apply` if formatting
  fails the build.
- **Checkstyle / nohttp** are enforced via `src/checkstyle/`. Use `https://`
  in any new URLs in source, properties, or docs.
- Match existing patterns:
  - Controllers in feature packages (e.g. `owner/OwnerController.java`),
    not in a global `controller/` package.
  - Entities extend `BaseEntity` / `NamedEntity` from `model/`.
  - Repositories are Spring Data JPA interfaces co-located with their entity.
  - Validation uses Jakarta Bean Validation annotations on entities/DTOs.
  - Views are Thymeleaf templates under `src/main/resources/templates/`,
    grouped by feature (`owners/`, `pets/`, `vets/`, `fragments/`).
- Keep messages internationalized via `src/main/resources/messages/messages*.properties`.
  When adding user-facing strings, add the key to `messages.properties` and,
  where reasonable, to existing locale variants.
- Prefer constructor injection. Avoid field injection and `@Autowired` on
  fields in new code.
- Do not add Lombok; the project does not use it.

## Behavior-Driven Development (BDD)

This repository ships a project skill at
[.github/skills/bdd-scenario-design/SKILL.md](.github/skills/bdd-scenario-design/SKILL.md).
**Load and follow that skill** for any task that adds or changes user-visible
behavior, including:

- new features and enhancements
- converting a user story or issue into acceptance criteria
- defining expected behavior before implementation
- regression scenarios for bug fixes
- clarifying edge cases, validation, empty-result, and not-found behavior

Do **not** invoke the BDD skill for pure cleanup, CI-only changes,
dependency-only upgrades, or formatting-only work.

When the BDD skill applies, follow this outside-in flow before writing
production code:

1. Produce a **BDD Scenario Pack** in the format defined by the skill
   (Feature intent, Scope, Domain Language, Gherkin scenarios, Acceptance
   Criteria, Test Strategy, Implementation Guidance, Open Questions).
2. Write Gherkin scenarios in **PetClinic domain language** (owner, pet,
   veterinarian, visit, specialty, clinic staff, owner details page,
   validation message). Avoid controller/repository/class/table names
   inside `Given/When/Then` steps.
3. Cover the happy path, validation failures, empty/not-found cases, and
   any regression behavior. Use `Scenario Outline` for input-table
   validation and `Background` only when it removes real duplication.
4. Map each scenario to the **smallest suitable test type** using the
   existing stack:
   - Controller / form / view behavior → `@WebMvcTest` + MockMvc.
   - Domain validation → unit / Bean Validation tests.
   - Repository / JPA queries → `@DataJpaTest`.
   - End-to-end Spring behavior → `@SpringBootTest` (e.g.
     `PetClinicIntegrationTests`, `MySqlTestApplication`,
     `PostgresIntegrationTests`).
5. Implement **test-first**: add or update the failing test for the first
   scenario, make the smallest production change to pass it, then iterate
   through edge cases.
6. **Do not introduce Cucumber** or any new BDD runtime. BDD scenarios are
   planning and test-design artifacts; they drive existing JUnit 5 /
   Spring Boot Test / MockMvc / repository / integration tests.
7. Use `./mvnw test` for the narrowest validation loop and `./mvnw -B verify`
   before declaring work done.
8. If a scenario is ambiguous, surface it as an **Open Question** in the
   Scenario Pack instead of inventing behavior silently.

For regression fixes, write the regression as a scenario that **fails before
the fix and passes after it**, and keep the fix scoped to the behavior
described by that scenario.

## Tests

- Add or update tests for every behavior change. Place tests under
  `src/test/java/...` mirroring the production package.
- Use the existing testing stack: JUnit 5, Spring Boot Test, MockMvc,
  AssertJ, Testcontainers (for DB integration where appropriate).
- For controller changes, prefer `@WebMvcTest` + MockMvc tests like the
  existing ones in `owner/` and `vet/`.
- For repository/JPA changes, prefer `@DataJpaTest`.
- Do not delete or weaken existing assertions to make a build pass.

## Build, run, and verify

Use these commands locally and in the coding agent before declaring work done.
Prefer Maven unless the task is Gradle-specific.

- Compile + unit tests: `./mvnw -B verify`
- Run the app (H2): `./mvnw spring-boot:run`
- Rebuild CSS from SCSS: `./mvnw package -P css`
- Build a container image: `./mvnw spring-boot:build-image`
- Gradle equivalents: `./gradlew check`, `./gradlew bootRun`.
- On Windows, use `./mvnw.cmd` / `./gradlew.bat` when invoking from `cmd.exe`.

Before opening a PR, the **minimum bar** is:

1. `./mvnw -B verify` passes locally (or in the agent sandbox).
2. No new Checkstyle / nohttp / Spring Java Format violations.
3. Application starts successfully against the default H2 profile.
4. Any new or changed behavior is covered by tests.

If MySQL or PostgreSQL behavior is changed, also verify with the matching
profile (`-Dspring-boot.run.profiles=mysql` / `postgres`) using the provided
`docker-compose.yml`.

## Database changes

- Schema lives in `src/main/resources/db/{h2,mysql,postgres}/schema.sql`.
- Seed data lives in `src/main/resources/db/{h2,mysql,postgres}/data.sql`.
- Any schema change **must** be applied to **all three** dialects and kept
  consistent. Update entities and repositories accordingly.

## Dependencies

- Prefer versions managed by `spring-boot-starter-parent`. Only pin a version
  in `<properties>` when Spring Boot does not manage it (see existing
  `webjars-*`, `checkstyle.version`, etc.).
- Do not introduce new heavyweight dependencies (security frameworks, message
  brokers, cloud SDKs) without an explicit task requirement.
- Keep Maven and Gradle dependency sets aligned.

## Commits & pull requests

- Keep commits focused; one logical change per commit where practical.
- Every commit message **must** include a `Signed-off-by:` trailer (DCO is
  enforced — see `.github/dco.yml`). Use `git commit -s`.
- PR descriptions should explain *what* changed and *why*, list test
  evidence (commands run and outcome), and call out any DB schema or
  build-tool changes.
- Do not force-push shared branches or rewrite history on `main`.

## Coding-agent-specific guidance (cloud runs)

When running as the GitHub Copilot coding agent:

- Treat the sandbox as ephemeral; rely on `./mvnw` / `./gradlew` wrappers,
  not on globally installed tools.
- Run `./mvnw -B verify` as the canonical validation step.
- Do not attempt to reach external networks beyond what Maven/Gradle need
  for dependency resolution.
- Do not modify GitHub Actions workflows under `.github/workflows/`
  (`maven-build.yml`, `gradle-build.yml`, `deploy-and-test-cluster.yml`)
  unless the task is explicitly about CI.
- Keep diffs small and reviewable. If a task seems to require sweeping
  changes, stop and surface the scope concern in the PR description rather
  than expanding silently.

## IDE-mode guidance (Copilot Chat in editor)

- Prefer suggestions that compile against the current `pom.xml` /
  `build.gradle` without adding new dependencies.
- When proposing edits, reference existing files and patterns in this repo
  (e.g. `OwnerController`, `VetController`) instead of inventing new
  conventions.
- For UI changes, edit the relevant Thymeleaf template under
  `src/main/resources/templates/<feature>/` and, if styling is involved,
  the matching SCSS in `src/main/scss/` (then regenerate CSS via the Maven
  `css` profile).
- Localized strings go in `messages.properties` (and locale variants where
  reasonable), not hardcoded in templates or controllers.

## Out of scope by default

Unless the task explicitly says otherwise, do **not**:

- Switch templating engines, ORMs, or web frameworks.
- Add authentication/authorization, multi-tenancy, or external service
  integrations.
- Introduce Docker, Helm, Terraform, or cloud-deployment assets beyond what
  already exists in `k8s/` and `docker-compose.yml`.
- Reformat or restructure files you did not otherwise need to change.
