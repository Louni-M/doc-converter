# C4 Model - Level 2: Container Diagram

## Doc Converter - Container Architecture

```mermaid
C4Container
    title Container Diagram for Doc Converter

    Person(user, "User", "Document converter user")

    Container_Boundary(app, "Doc Converter Application") {
        Container(mainProcess, "Main Process", "Electron Main", "Manages application lifecycle, file system access, and process communication")
        Container(rendererProcess, "Renderer Process", "Electron Renderer + HTML/CSS/JS", "Provides user interface for file selection and conversion options")
        Container(conversionEngine, "Conversion Engine", "JavaScript/Node.js", "Handles document format conversion using JSZip and other libraries")

        ContainerDb(tempStorage, "Temporary Storage", "File System", "Stores temporary files during conversion process")
    }

    System_Ext(fileSystem, "File System", "User's documents")
    System_Ext(libreOffice, "LibreOffice", "Document processor")

    Rel(user, rendererProcess, "Interacts with", "UI")
    Rel(rendererProcess, mainProcess, "Sends conversion requests", "IPC")
    Rel(mainProcess, conversionEngine, "Delegates conversion", "Function calls")
    Rel(conversionEngine, tempStorage, "Uses for processing", "File I/O")
    Rel(mainProcess, fileSystem, "Reads/Writes", "File I/O")
    Rel(mainProcess, libreOffice, "Invokes", "CLI")

    UpdateLayoutConfig($c4ShapeInRow="2", $c4BoundaryInRow="1")
```

## Container Details

### Main Process (Electron Main)
**Technology**: Node.js / Electron Main Process

**Responsibilities**:
- Application lifecycle management (startup, shutdown)
- File system access and security
- Inter-process communication with renderer
- Coordination of conversion operations
- Integration with external tools (LibreOffice)

**Key Dependencies**:
- Electron Framework
- Node.js File System APIs

### Renderer Process (Electron Renderer)
**Technology**: Chromium / HTML / CSS / JavaScript

**Responsibilities**:
- User interface presentation
- File selection dialogs
- Conversion progress display
- User preferences management

**Key Dependencies**:
- Electron Renderer APIs
- Web technologies (HTML5, CSS3, ES6+)

### Conversion Engine
**Technology**: JavaScript / Node.js

**Responsibilities**:
- Document format detection
- Conversion logic implementation
- ZIP archive manipulation (via JSZip)
- Format-specific transformations
- Error handling and validation

**Key Dependencies**:
- JSZip (v3.10.1) - ZIP file manipulation
- Pako - Compression/decompression
- Additional format-specific libraries

### Temporary Storage
**Technology**: File System

**Responsibilities**:
- Storing intermediate conversion files
- Caching extracted archive contents
- Temporary workspace for complex conversions

## Communication Patterns

### IPC (Inter-Process Communication)
- **Renderer → Main**: File selection events, conversion requests
- **Main → Renderer**: Progress updates, completion notifications, errors

### File I/O
- **Synchronous**: Configuration reading, small file operations
- **Asynchronous**: Large file conversions, batch processing

## Security Considerations
- File system access controlled through Electron's context isolation
- User-selected files only (no automatic file scanning)
- Temporary files cleaned up after conversion
