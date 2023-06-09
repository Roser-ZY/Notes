[TOC]

#### 想在前端展示后端已上传的文件，后端响应的数据类型应该是什么

在前端展示后端已上传的文件，通常需要通过后端提供的接口获取文件的数据。在这种情况下，后端响应的数据类型应该是 **二进制数据流（binary data stream）** 或者 **base64 编码字符串（base64-encoded string）**。

如果使用二进制数据流，前端可以使用 `Blob` 对象来处理响应数据。`Blob` 对象表示一个不可变的、原始数据的类文件对象。可以使用 `URL.createObjectURL()` 方法创建一个指向 `Blob` 对象的 URL，然后将其赋值给一个 `img` 或 `video` 元素的 `src` 属性，或者使用 `FileReader` 对象读取 `Blob` 对象的内容并进行处理。

如果使用 base64 编码字符串，前端可以将其转换为 `data URI`，然后将其赋值给一个 `img` 或 `video` 元素的 `src` 属性。可以使用 `atob()` 函数将 base64 编码的字符串解码为二进制数据，然后使用 `Blob` 对象将其转换为 `data URI`。

以下是使用 JavaScript 在前端展示后端已上传文件的示例代码：

```
iniCopy code// 使用 Fetch API 获取文件数据
fetch('/api/getFileData')
  .then(response => response.arrayBuffer())
  .then(buffer => {
    // 将二进制数据流转换为 Blob 对象
    const blob = new Blob([buffer], { type: 'image/png' });
    // 创建 URL
    const url = URL.createObjectURL(blob);
    // 将 URL 赋值给 img 元素的 src 属性
    const img = document.createElement('img');
    img.src = url;
    document.body.appendChild(img);
  });

// 或者

// 使用 Fetch API 获取文件数据
fetch('/api/getFileData')
  .then(response => response.text())
  .then(base64String => {
    // 将 base64 编码字符串转换为 data URI
    const dataURI = `data:image/png;base64,${base64String}`;
    // 将 data URI 赋值给 img 元素的 src 属性
    const img = document.createElement('img');
    img.src = dataURI;
    document.body.appendChild(img);
  });
```

#### 后端使用Spring，想获取服务端本地文件并发送给前端展示。

在 Spring 中，可以使用 `Resource` 类来获取服务端本地文件，并将其发送给前端展示。以下是一个获取本地文件并发送给前端的示例代码：

```
javaCopy code@GetMapping("/file")
public ResponseEntity<Resource> getFile() throws IOException {
    // 获取本地文件
    Resource fileResource = new FileSystemResource("path/to/file.txt");
    // 创建响应实体
    HttpHeaders headers = new HttpHeaders();
    headers.add(HttpHeaders.CONTENT_DISPOSITION, "attachment; filename=file.txt");
    return ResponseEntity.ok()
            .headers(headers)
            .contentLength(fileResource.contentLength())
            .contentType(MediaType.APPLICATION_OCTET_STREAM)
            .body(fileResource);
}
```

在上面的代码中，首先使用 `FileSystemResource` 类获取本地文件。然后，创建一个 `ResponseEntity` 对象，并将文件内容设置为响应实体的正文。在设置响应头时，使用 `HttpHeaders.CONTENT_DISPOSITION` 头指定文件名，这样浏览器就可以正确地显示文件名。

对于文本文件，可以使用 `MediaType.TEXT_PLAIN` 作为响应实体的内容类型。对于其他类型的文件，可以使用 `MediaType.APPLICATION_OCTET_STREAM`。

在前端，可以使用 AJAX 或 Fetch API 发送 GET 请求来获取文件数据。以下是一个使用 Fetch API 获取文件并在页面中展示的示例代码：

```
javascriptCopy codefetch('/file')
  .then(response => response.blob())
  .then(blob => {
    // 创建 URL
    const url = URL.createObjectURL(blob);
    // 将 URL 赋值给 a 元素的 href 属性
    const a = document.createElement('a');
    a.href = url;
    a.download = 'file.txt';
    a.textContent = 'Download file';
    document.body.appendChild(a);
  });
```

在上面的代码中，首先使用 Fetch API 获取文件数据，并将其转换为 `Blob` 对象。然后，使用 `URL.createObjectURL()` 方法创建一个指向 `Blob` 对象的 URL，将其赋值给一个 `a` 元素的 `href` 属性，这样用户就可以通过点击链接来下载文件。

#### blob是啥？

在计算机科学中，`Blob`（Binary Large Object）是一种二进制数据对象，用于存储大量的二进制数据。`Blob` 对象通常用于存储文件或图像等二进制数据，可以在客户端和服务端之间传递。

`Blob` 对象可以通过多种方式创建，包括从 `ArrayBuffer`、`BufferSource`、`Blob`、`DOMString`、`ReadableStream` 和 `FormData` 等类型的数据中创建。对于文件或图像等二进制数据，通常可以使用 `FileReader` 或 `URL.createObjectURL()` 方法将 `Blob` 对象转换为可用于展示或下载的数据。

在浏览器中，`Blob` 对象通常用于将大量数据存储在客户端，例如在客户端对图像或视频进行处理时。在服务端，`Blob` 对象通常用于将文件或图像等二进制数据存储在数据库或文件系统中，或者将其发送给客户端进行下载或展示。

以下是一个创建 `Blob` 对象并将其转换为可用于展示或下载的数据的示例代码：

```
javascriptCopy code// 创建 Blob 对象
const data = new Uint8Array([0x48, 0x65, 0x6c, 0x6c, 0x6f]);
const blob = new Blob([data], { type: 'text/plain' });

// 将 Blob 对象转换为可用于展示或下载的数据
const reader = new FileReader();
reader.onload = function() {
  const result = reader.result;
  // 将 result 赋值给 img 元素的 src 属性或者 a 元素的 href 属性
};
reader.readAsDataURL(blob);
```

在上面的代码中，首先创建一个 `Uint8Array` 对象来存储二进制数据，然后使用 `Blob` 构造函数创建一个 `Blob` 对象。在将 `Blob` 对象转换为可用于展示或下载的数据时，使用 `FileReader` 对象读取 `Blob` 对象的内容，并将其转换为 `data URI`，然后将其赋值给 `img` 或 `a` 元素的属性。

#### 前面所说的Resource类型是啥，我可以使用MultipartFile类型来实现吗？

`Resource` 是 Spring 框架中的一个接口，用于表示一个资源，可以是文件、URL、类路径等。`Resource` 接口提供了一些方法，用于获取资源的元数据、输入流等信息。

`MultipartFile` 是 Spring 框架中的一个接口，用于表示一个上传的文件。`MultipartFile` 接口提供了一些方法，用于获取文件的名称、大小、输入流等信息。

在 Spring 中，可以使用 `Resource` 或 `MultipartFile` 类型来获取服务端本地文件，并将其发送给前端展示。如果你已经上传了文件并且使用了 `MultipartFile`，那么可以直接将其转换为 `Resource` 类型，然后将其发送给前端。以下是一个使用 `MultipartFile` 获取本地文件并发送给前端的示例代码：

```
javaCopy code@GetMapping("/file")
public ResponseEntity<Resource> getFile(@RequestParam("file") MultipartFile file) throws IOException {
    // 将 MultipartFile 转换为 Resource
    Resource fileResource = new InputStreamResource(file.getInputStream());
    // 创建响应实体
    HttpHeaders headers = new HttpHeaders();
    headers.add(HttpHeaders.CONTENT_DISPOSITION, "attachment; filename=" + file.getOriginalFilename());
    return ResponseEntity.ok()
            .headers(headers)
            .contentLength(file.getSize())
            .contentType(MediaType.APPLICATION_OCTET_STREAM)
            .body(fileResource);
}
```

在上面的代码中，首先将 `MultipartFile` 对象转换为 `InputStreamResource` 对象，然后创建一个 `ResponseEntity` 对象，并将文件内容设置为响应实体的正文。在设置响应头时，使用 `HttpHeaders.CONTENT_DISPOSITION` 头指定文件名，这样浏览器就可以正确地显示文件名。

对于文本文件，可以使用 `MediaType.TEXT_PLAIN` 作为响应实体的内容类型。对于其他类型的文件，可以使用 `MediaType.APPLICATION_OCTET_STREAM`。

在前端，可以使用 AJAX 或 Fetch API 发送 GET 请求来获取文件数据。以下是一个使用 Fetch API 获取文件并在页面中展示的示例代码：

```
javascriptCopy codefetch('/file?file=filename.txt')
  .then(response => response.blob())
  .then(blob => {
    // 创建 URL
    const url = URL.createObjectURL(blob);
    // 将 URL 赋值给 a 元素的 href 属性
    const a = document.createElement('a');
    a.href = url;
    a.download = 'filename.txt';
    a.textContent = 'Download file';
    document.body.appendChild(a);
  });
```

在上面的代码中，首先使用 Fetch API 获取文件数据，并将其转换为 `Blob` 对象。然后，使用 `URL.createObjectURL()` 方法创建一个指向 `Blob` 对象的 URL，将其赋值给一个 `a` 元素的 `href` 属性，这样用户就可以通过点击链接来下载文件。