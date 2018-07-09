---
description: 下载
---

# Download

下载的话有几种方法，下面这个转自[博客](https://blog.csdn.net/zknxx/article/details/72633444)，经测好使。 具体细节先不做解释，核心方法应该就是flushBuffer，再说吧，先拿来用。

```text
    @RequestMapping("downloadFileAction")
    public void downloadFileAction(HttpServletRequest request, HttpServletResponse response) {

        response.setCharacterEncoding(request.getCharacterEncoding());
        response.setContentType("application/octet-stream");
        FileInputStream fis = null;
        try {
            File file = new File("C:\\Users\\carry\\Pictures\\198109742591463b0b7396936.jpg");
            fis = new FileInputStream(file);
            response.setHeader("Content-Disposition", "attachment; filename=" + file.getName());
            IOUtils.copy(fis, response.getOutputStream());
            response.flushBuffer();
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (fis != null) {
                try {
                    fis.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
```

下面是第二种方法，这是我自己测试从ftp上下载文件时测试的，也是从某博客上粘的，但已无从考证，原理依旧不解释，等我熟悉了再说。。。

```text
    @GetMapping("downloadTest")
    public ResponseEntity<InputStreamResource> downloadTest() throws Exception {
        File file = new File("C:\\Users\\carry\\Pictures\\zhizi.jpg");
        InputStream inputStream = new FileInputStream(file);
        InputStreamResource inputStreamResource = new InputStreamResource(inputStream);
        HttpHeaders headers = new HttpHeaders();
        headers.add("Cache-Control", "no-cache, no-store, must-revalidate");
        headers.add("Content-Disposition", String.format("attachment; filename=\"%s\"", file.getName()));
        headers.add("Pragma", "no-cache");
        headers.add("Expires", "0");
        return ResponseEntity
                .ok()
                .headers(headers)
                .contentLength(file.length())
                .contentType(MediaType.parseMediaType("application/octet-stream"))
                .body(inputStreamResource);
    }
```

