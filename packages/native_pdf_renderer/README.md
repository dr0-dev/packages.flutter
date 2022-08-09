> ## Plugin renamed and republished as `[pdfx]`
>
> [[pdfx] on pub.dev](https://pub.dev/packages/pdfx)
>
> Some smaller api changes
> 
>


Migration:
1. Replace dependencies
```diff
dependencies:
-   native_pdf_renderer: ^4.0.1
+   pdfx: ^1.0.0
```
2. Renamed `PdfPageFormat` -> `PdfPageImageFormat`
3. Re-case values `PdfPageImageFormat{JPEG,PNG,WEBP}` -> `PdfPageImageFormat{jpeg,png,webp}`

# PDF Renderer

`Flutter` Plugin to render PDF pages as images on **Web**, **MacOs 10.11+**, **Android 5.0+**, **iOS** and **Windows**.

**We also support the package for easy display PDF documents [native_pdf_view](https://pub.dev/packages/native_pdf_view)**

## Getting Started
In your flutter project add the dependency:

[![pub package](https://img.shields.io/pub/v/native_pdf_renderer.svg)](https:///packages/native_pdf_renderer)

```yaml
dependencies:
  native_pdf_renderer: any
```

For web add lines in index.html before importing main.dart.js:<br/>
**note that the files have different names**
```html
<script src="https://cdn.jsdelivr.net/npm/pdfjs-dist@2.7.570/build/pdf.js" type="text/javascript"></script>
<script type="text/javascript">
  pdfjsLib.GlobalWorkerOptions.workerSrc = "https://cdn.jsdelivr.net/npm/pdfjs-dist@2.7.570/build/pdf.worker.min.js";
  pdfRenderOptions = {
    cMapUrl: 'https://cdn.jsdelivr.net/npm/pdfjs-dist@2.7.570/cmaps/',
    cMapPacked: true,
  }
</script>
```

for windows the pdfium version used can be overridden by the base flutter application by adding the following line to the host apps CMakeLists.txt file:
```
set(PDFIUM_VERSION "4638" CACHE STRING "")
```

## Usage example

```dart
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';
import 'package:native_pdf_renderer/native_pdf_renderer.dart';

void main() async {
  try {
    final document = await PdfDocument.openAsset('assets/sample.pdf');
    final page = await document.getPage(1);
    final pageImage = await page.render(width: page.width, height: page.height);
    await page.close();
    runApp(
      MaterialApp(
        home: Scaffold(
          body: Center(
            child: Image(
              image: MemoryImage(pageImage.bytes),
            ),
          ),
        ),
        color: Colors.white,
      )
    );
  } on PlatformException catch (error) {
    print(error);
  }
}
```

## Api

### PdfDocument

| Parameter  | Description                                                                                | Default |
|------------|--------------------------------------------------------------------------------------------|---------|
| sourceName | Needed for toString method. Contains a method for opening a document (file, data or asset) | -       |
| id         | Document unique id. Generated when opening document.                                       | -       |
| pagesCount | All pages count in document. Starts from 1.                                                | -       |
| isClosed   | Is the document closed                                                                     | -       |

**Local document open:**
```dart
// From assets (Android, Ios, MacOs, Web)
PdfDocument.openAsset('assets/sample.pdf')

// From file (Android, Ios, MacOs)
PdfDocument.openFile('path/to/file/on/device')

// From data (Android, Ios, MacOs, Web)
PdfDocument.openData((FutureOr<Uint8List>) data)
```
**Network document open:**

Install [[network_file]](https://pub.dev/packages/internet_file) package (supports all platforms):
```shell
flutter pub add internet_file
```

And use it
```dart
import 'package:internet_file/internet_file.dart';

PdfDocument.openData(InternetFile.get(
    'https://github.com/rbcprolabs/packages.flutter/raw/fd0c92ac83ee355255acb306251b1adfeb2f2fd6/packages/native_pdf_renderer/example/assets/sample.pdf',
))
```

**Open page:**
```dart
final page = document.getPage(pageNumber); // Starts from 1
```

**Close document:**
```dart
document.close();
```

### PdfPage

| Parameter | Description                                                                         | Default |
|-----------|-------------------------------------------------------------------------------------|---------|
| document  | Parent document                                                                     | Parent  |
| id        | Page unique id. Needed for rendering and closing page. Generated when opening page. | -       |
| width     | Page source width in pixels, int                                                    | -       |
| height    | Page source height in pixels, int                                                   | -       |
| isClosed  | Is the page closed                                                                  | false   |

**Render image:**
```dart
final pageImage = page.render(
  // rendered image width resolution, required
  width: page.width * 2,
  // rendered image height resolution, required
  height: page.height * 2,

  // Rendered image compression format, also can be PNG, WEBP*
  // Optional, default: PdfPageFormat.PNG
  // Web not supported
  format: PdfPageFormat.JPEG,

  // Image background fill color for JPEG
  // Optional, default '#ffffff'
  // Web not supported
  backgroundColor: '#ffffff',

  // Crop rect in image for render
  // Optional, default null
  // Web not supported
  cropRect: Rect.fromLTRB(left, top, right, bottom),
);
```

### PdfPageImage

| Parameter  | Description                                                                        | Default           |
|------------|------------------------------------------------------------------------------------|-------------------|
| id         | Page unique id. Needed for rendering and closing page. Generated when render page. | -                 |
| pageNumber | Page number. The first page is 1.                                                  | -                 |
| width      | Width of the rendered area in pixels, int                                          | -                 |
| height     | Height of the rendered area in pixels, int                                         | -                 |
| bytes      | Rendered image result, Uint8List                                                   | -                 |
| format     | Rendered image compression format, for web always PNG                              | PdfPageFormat.PNG |

**Close page:**
<br>
Before open new page android asks to close the past. <br>
If this is not done, the application may crash with an error
```dart
page.close();
```

\* __PdfPageFormat.WEBP support only on android__

## Rendering additional info

### On Web
This plugin uses the [PDF.js](https://mozilla.github.io/pdf.js/)

### On Android
This plugin uses the Android native [PdfRenderer](https://developer.android.com/reference/android/graphics/pdf/PdfRenderer)

### On Ios & MacOs
This plugin uses the IOS native [CGPDFPage](https://developer.apple.com/documentation/coregraphics/cgpdfdocument/cgpdfpage)

### On Windows
This plugin use [PDFium](https://pdfium.googlesource.com/pdfium/+/master/README.md)
