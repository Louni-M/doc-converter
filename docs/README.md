# Doc Converter - Architecture Documentation

## 📚 Documentation Overview

Welcome to the comprehensive architecture documentation for **Doc Converter**, a macOS desktop application built with Electron for converting documents between various formats.

---

## 📂 Documentation Structure

### 🏗️ Architecture Models

#### [C4 Model Diagrams](./architecture/c4-model/)
Hierarchical architecture visualization following the C4 model:

1. **[System Context](./architecture/c4-model/01-context-diagram.md)** - High-level view of the system and its users
2. **[Container Diagram](./architecture/c4-model/02-container-diagram.md)** - Application structure and major components
3. **[Component Diagram](./architecture/c4-model/03-component-diagram.md)** - Internal component relationships and responsibilities

#### [ARC42 Documentation](./architecture/arc42/)
Comprehensive architecture documentation following the ARC42 template:

- **[Architecture Documentation](./architecture/arc42/architecture-documentation.md)** - Complete system architecture (12 sections):
  - Introduction and Goals
  - Constraints
  - System Scope and Context
  - Solution Strategy
  - Building Block View
  - Runtime View
  - Deployment View
  - Cross-Cutting Concerns
  - Architecture Decisions
  - Quality Requirements
  - Risks and Technical Debt
  - Glossary

### 📐 Technical Diagrams

#### [Deployment Architecture](./architecture/diagrams/deployment-diagram.md)
- macOS Application Bundle Structure
- Process Architecture
- Memory Layout
- File System Locations
- System Requirements
- Installation & Distribution

#### [Security Architecture](./architecture/diagrams/security-architecture.md)
- Security Model and Layers
- Renderer Process Sandboxing
- Input Validation Strategies
- ZIP Bomb Protection
- Temporary File Security
- Threat Model and Mitigations

### 📝 Architecture Decision Records (ADRs)

Decision history and rationale for key architectural choices:

- **[ADR-001](./adr/0001-electron-framework.md)** - Choice of Electron Framework
- **[ADR-002](./adr/0002-jszip-for-docx.md)** - Using JSZip for DOCX/XLSX Processing
- **[ADR-003](./adr/0003-libreoffice-integration.md)** - LibreOffice Integration for Complex Conversions

---

## 🚀 Quick Start Guide

### For Developers
1. Start with the **[System Context Diagram](./architecture/c4-model/01-context-diagram.md)** to understand the big picture
2. Review the **[Container Diagram](./architecture/c4-model/02-container-diagram.md)** to see how the app is structured
3. Dive into the **[Component Diagram](./architecture/c4-model/03-component-diagram.md)** for implementation details
4. Read the **[ADRs](./adr/)** to understand why decisions were made

### For Architects
1. Review the **[ARC42 Documentation](./architecture/arc42/architecture-documentation.md)** for complete system overview
2. Check the **[Security Architecture](./architecture/diagrams/security-architecture.md)** for security considerations
3. Examine the **[Deployment Architecture](./architecture/diagrams/deployment-diagram.md)** for infrastructure details

### For Security Reviewers
1. Start with the **[Security Architecture](./architecture/diagrams/security-architecture.md)**
2. Review relevant **[ADRs](./adr/)** for security-related decisions
3. Check the **[ARC42 Cross-Cutting Concerns](./architecture/arc42/architecture-documentation.md#8-cross-cutting-concepts)** section

---

## 🏛️ System Architecture at a Glance

### Technology Stack
- **Framework**: Electron (Chromium + Node.js)
- **Language**: JavaScript/Node.js
- **Document Processing**: JSZip (v3.10.1)
- **External Tools**: LibreOffice (optional)
- **Platform**: macOS (cross-platform capable)

### Key Components
```
┌──────────────────────────────────────┐
│     Main Process (Orchestration)     │
│  • Application Lifecycle             │
│  • File System Access                │
│  • IPC Coordination                  │
└──────────────────────────────────────┘
                  │
        ┌─────────┴─────────┐
        │                   │
┌───────▼──────┐   ┌────────▼────────┐
│   Renderer   │   │   Conversion    │
│   Process    │   │     Engine      │
│   (UI)       │   │  • Format Det.  │
│              │   │  • ZIP Handler  │
│              │   │  • PDF Conv.    │
│              │   │  • Validation   │
└──────────────┘   └─────────────────┘
```

### Supported Formats
**Input**: DOCX, DOC, XLSX, XLS, PDF, ODT, ODS, TXT, HTML
**Output**: PDF, DOCX, XLSX, HTML, TXT

---

## 🔒 Security Highlights

- ✅ **Sandboxed Renderer**: UI isolated from system resources
- ✅ **Context Isolation**: Controlled API bridge between processes
- ✅ **Input Validation**: Path sanitization, file type checking
- ✅ **ZIP Bomb Protection**: Compression ratio and size limits
- ✅ **Process Isolation**: External tools run in isolated processes
- ✅ **Offline Only**: No network access, fully local operation

---

## 📊 Quality Attributes

| Attribute | Target | Status |
|-----------|--------|--------|
| **Usability** | < 5 min learning time | ✅ Achieved |
| **Performance** | < 2s for small files | ✅ Achieved |
| **Reliability** | > 95% success rate | 🟡 Measuring |
| **Security** | Zero critical vulnerabilities | ✅ Achieved |
| **Maintainability** | Modular architecture | ✅ Achieved |

---

## 🛠️ Development Resources

### Build and Package
```bash
# Install dependencies
npm install

# Run in development
npm start

# Build for macOS
npm run build:mac

# Package as .app bundle
npm run package
```

### Testing
```bash
# Run unit tests
npm test

# Run integration tests
npm run test:integration

# Security audit
npm audit
```

### Code Quality
- **Linting**: ESLint with Airbnb config
- **Formatting**: Prettier
- **Type Checking**: JSDoc with TypeScript checking

---

## 📈 Roadmap & Future Enhancements

### Planned Features
- [ ] Batch conversion support
- [ ] Custom conversion presets
- [ ] Cloud storage integration (optional)
- [ ] Command-line interface
- [ ] Auto-update mechanism

### Technical Improvements
- [ ] Automated testing suite
- [ ] Performance profiling
- [ ] CI/CD pipeline
- [ ] Error telemetry (opt-in)

---

## 🤝 Contributing

### Documentation Updates
When updating architecture:
1. Update relevant C4 diagrams
2. Create an ADR for significant decisions
3. Update ARC42 documentation
4. Update this README if structure changes

### Architecture Review Process
- **Minor Changes**: Team review, update docs
- **Major Changes**: Create ADR, architecture meeting, consensus required
- **Review Cycle**: Quarterly architecture reviews

---

## 📞 Support & Contact

### Documentation Questions
- **Format**: See [ARC42 Glossary](./architecture/arc42/architecture-documentation.md#12-glossary)
- **Architecture**: Review [C4 Diagrams](./architecture/c4-model/)
- **Decisions**: Check [ADRs](./adr/)

### Security Issues
- **Report**: Create GitHub issue with `security` label
- **Review**: See [Security Architecture](./architecture/diagrams/security-architecture.md)

---

## 📄 License & Attribution

**Documentation License**: CC BY 4.0
**Last Updated**: 2025-11-04
**Version**: 1.0.0

---

## 🗺️ Quick Navigation

| You want to... | Go to... |
|----------------|----------|
| Understand the system | [System Context](./architecture/c4-model/01-context-diagram.md) |
| See how it's built | [Container Diagram](./architecture/c4-model/02-container-diagram.md) |
| Understand components | [Component Diagram](./architecture/c4-model/03-component-diagram.md) |
| Review complete docs | [ARC42 Documentation](./architecture/arc42/architecture-documentation.md) |
| Check security | [Security Architecture](./architecture/diagrams/security-architecture.md) |
| See deployment | [Deployment Diagram](./architecture/diagrams/deployment-diagram.md) |
| Understand decisions | [ADRs](./adr/) |

---

**Happy Architecting! 🎉**
