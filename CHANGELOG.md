# Changelog

All notable changes to the Spring Boot 3.5 documentation will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [2025-06-14] - Documentation Review Update

### Added

#### New Chapter: Data Persistence (Chapter 5)
- **JPA Quickstart** with comprehensive entity, repository, and service examples
- **JPA Best Practices** covering loading strategies, paging, and transaction management
- **R2DBC Reactive Data Access** with full WebFlux integration examples
- Cross-references from Project Setup (Chapter 3) and Testing (Chapter 10)

#### Enhanced Auto-configuration Chapter (Chapter 4)
- **PlantUML sequence diagram** illustrating auto-configuration processing flow
- **@SpringBootTest integration test examples** for custom starter validation
- Detailed test case implementations with ApplicationContextRunner

#### Enhanced Preface (Chapter 0)
- **Explicit environment requirements** for Java 17, 21 (LTS), and 24 EA
- **Community links** to GitHub Issues and Discussions
- Detailed version compatibility matrix for build tools and OS

#### Enhanced Operations & Observability Chapter (Chapter 7)
- **ECS-compliant JSON logging examples** with complete configuration
- **Grafana Tempo/Zipkin tracing integration** with Docker Compose setup
- **Metrics smoke-testing guidance** with CI/CD pipeline examples
- Custom JSON layout configuration for production environments

#### Enhanced Security Chapter (Chapter 9)
- **Comprehensive in-memory user setup** with role-based access control
- **Spring Security version clarification** (6.4/6.5 differences)
- **WebAuthn sample application repository** link and implementation guidance
- Custom authentication entry point examples

#### Enhanced Testing Chapter (Chapter 10)
- **Testcontainers quick setup requirements** with Docker configuration
- **Parallel execution caching** (`TESTCONTAINERS_REUSE_ENABLE`) with performance metrics
- **Micrometer tracing test examples** using SpanRecorder and test containers
- E2E tracing validation with Zipkin integration

### Changed

#### Chapter Restructuring
- Renamed existing chapters 5-12 to 6-13 to accommodate new data persistence chapter
- Updated all chapter numbering and cross-references throughout the documentation
- Maintained consistent style and formatting across all chapters

#### Code Examples Enhancement
- **Public visibility** applied to controller and configuration classes (G-1 convention)
- **Inline comments** added for clarity in complex code examples (G-3 convention)
- **Java version compatibility** validated for 17, 21 (LTS), and 24 EA (G-2 convention)

#### Project Setup Chapter (Chapter 3)
- Made `HelloController` class public with improved method visibility
- Added **native image build commands** for both Maven and Gradle
- Enhanced build command table with native compilation options

### Infrastructure

#### Assets Directory
- Created `/assets/` directory for diagrams and screenshots
- Prepared structure for auto-configuration sequence diagrams
- Reserved space for Grafana Tempo screenshots

### Technical Details

#### Documentation Standards Applied
- **G-1**: Public visibility for controller/configuration classes
- **G-2**: Code validation across Java 17, 21 (LTS), and 24 EA
- **G-3**: Inline comments for code clarity

#### Cross-Reference Updates
- Added bidirectional links between related chapters
- Enhanced navigation with specific section references
- Improved coherence across the documentation suite

### Future Enhancements

#### Planned Additions
- CI/CD workflows for Java version validation (17, 21, 24)
- Native image build validation in CI pipeline
- Auto-configuration sequence diagrams in assets
- Grafana Tempo screenshots and dashboard examples

#### Reference Updates
- Micrometer 1.14 references
- Spring Data JPA latest features
- R2DBC specification updates
- Testcontainers ServiceConnection enhancements

---

### Contributors

This update was implemented as part of the June 2025 comprehensive documentation review to ensure the Spring Boot 3.5 guide reflects current best practices and includes all essential features for modern cloud-native Java development.