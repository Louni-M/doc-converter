# ADR-002: Use JSZip for DOCX/XLSX Processing

## Status
**Accepted** - 2025-11-04

## Context
Modern document formats like DOCX and XLSX are ZIP archives containing XML files. We need a reliable way to:
- Extract and manipulate these archives
- Read and modify XML content
- Create new documents from scratch
- Handle compression/decompression efficiently

### Considered Alternatives

1. **JSZip (Pure JavaScript)**
   - Pros: No native dependencies, works in both Node.js and browser, well-maintained
   - Cons: Slower than native implementations, limited streaming support

2. **ADM-ZIP (Node.js native)**
   - Pros: Faster than JSZip, good for large files
   - Cons: Node.js only, less active maintenance, potential native compilation issues

3. **Node.js zlib (Built-in)**
   - Pros: No dependencies, fastest, built into Node.js
   - Cons: Low-level API, more code required, no ZIP format abstraction

4. **External Tools (unzip/zip CLI)**
   - Pros: Very fast, no JS dependencies
   - Cons: Platform-dependent, requires external binaries, harder to debug

5. **yauzl/yazl (Streaming-focused)**
   - Pros: Memory-efficient streaming, good for large files
   - Cons: More complex API, separate libraries for read/write

## Decision
We chose **JSZip** (v3.10.1) for the following reasons:

### Technical Factors
- **Cross-Process Compatibility**: Works in both Main and Renderer processes
- **Mature Library**: Stable, well-tested, active development
- **Simple API**: Intuitive interface for common operations
- **Promises Support**: Modern async/await patterns
- **Complete ZIP Support**: Handles all ZIP features we need

### Business Factors
- **No Native Dependencies**: Easier deployment, no compilation issues
- **Proven Track Record**: Used in many production applications
- **Good Documentation**: Comprehensive guides and examples
- **MIT License**: Permissive licensing

## Consequences

### Positive
- ✅ Pure JavaScript implementation (no native compilation)
- ✅ Works seamlessly with Electron's ASAR packaging
- ✅ Easy to use API with Promise support
- ✅ Handles both creation and extraction
- ✅ Good error handling and validation
- ✅ TypeScript definitions available
- ✅ Small bundle size (~100KB)

### Negative
- ❌ Slower than native implementations (acceptable for document sizes)
- ❌ Full file loading (not ideal for very large ZIP files >100MB)
- ❌ Memory usage increases with file size

### Mitigations
- For very large files (>50MB), implement file size warnings
- Use streaming where possible (temporary file extraction)
- Implement progress callbacks for user feedback
- Consider adding ADM-ZIP as fallback for large files in future

## Implementation Examples

### Reading DOCX Content
```javascript
const JSZip = require('jszip');
const fs = require('fs').promises;

async function extractDocxText(filePath) {
  const data = await fs.readFile(filePath);
  const zip = await JSZip.loadAsync(data);
  const documentXml = await zip.file('word/document.xml').async('string');
  // Parse XML and extract text
  return parseDocumentXml(documentXml);
}
```

### Creating DOCX
```javascript
async function createDocx(content, outputPath) {
  const zip = new JSZip();

  // Add required DOCX structure
  zip.file('[Content_Types].xml', contentTypesXml);
  zip.folder('word');
  zip.file('word/document.xml', generateDocumentXml(content));
  zip.file('word/_rels/document.xml.rels', relationshipsXml);

  const buffer = await zip.generateAsync({ type: 'nodebuffer' });
  await fs.writeFile(outputPath, buffer);
}
```

### Modifying XLSX
```javascript
async function modifyExcel(filePath, modifications) {
  const data = await fs.readFile(filePath);
  const zip = await JSZip.loadAsync(data);

  // Modify specific worksheet
  const sheetXml = await zip.file('xl/worksheets/sheet1.xml').async('string');
  const modifiedXml = applyModifications(sheetXml, modifications);
  zip.file('xl/worksheets/sheet1.xml', modifiedXml);

  return await zip.generateAsync({ type: 'nodebuffer' });
}
```

## Performance Considerations

### File Size Handling
| File Size | Expected Performance | Recommendation |
|-----------|---------------------|----------------|
| < 1 MB | < 100ms | Normal processing |
| 1-10 MB | < 1s | Normal processing |
| 10-50 MB | 1-5s | Show progress indicator |
| > 50 MB | > 5s | Consider warning user |

### Memory Usage
- JSZip loads entire file into memory
- Peak usage: ~3x file size during processing
- Mitigate by processing files sequentially, not in parallel

## Dependencies

```json
{
  "dependencies": {
    "jszip": "^3.10.1"
  }
}
```

**Transitive Dependencies**:
- `pako`: Compression/decompression (1.0.x)
- `lie`: Promise polyfill (3.3.x)
- `readable-stream`: Stream compatibility (2.3.x)
- `setimmediate`: setImmediate polyfill (1.0.x)

## Security Considerations

- Validate ZIP structure before processing (prevent zip bombs)
- Limit extraction size (prevent disk exhaustion)
- Sanitize file paths (prevent directory traversal)
- Timeout long operations (prevent DoS)

```javascript
// Security validation example
const MAX_UNCOMPRESSED_SIZE = 100 * 1024 * 1024; // 100MB

async function validateZip(zip) {
  let totalSize = 0;
  zip.forEach((relativePath, file) => {
    if (relativePath.includes('..')) {
      throw new Error('Invalid file path in ZIP');
    }
    totalSize += file._data.uncompressedSize;
  });

  if (totalSize > MAX_UNCOMPRESSED_SIZE) {
    throw new Error('ZIP file too large after decompression');
  }
}
```

## References
- [JSZip Documentation](https://stuk.github.io/jszip/)
- [DOCX Specification (Office Open XML)](https://learn.microsoft.com/en-us/openspecs/office_standards/ms-docx/)
- [XLSX Specification](https://learn.microsoft.com/en-us/openspecs/office_standards/ms-xlsx/)

## Related Decisions
- [ADR-001](./0001-electron-framework.md) - Electron framework choice
- [ADR-004](./0004-file-validation-strategy.md) - File validation approach

---
**Reviewed By**: Development Team
**Next Review**: When performance becomes an issue or when upgrading to JSZip v4
