In some contexts, such as in a URL path or the filename parameter of a multipart/form-data request, web servers may strip any directory traversal sequences before passing your input to the application. You can sometimes bypass this kind of sanitization by URL encoding, or even double URL encoding, the **../** characters. This results in 

```
%2e%2e%2f and %252e%252e%252f
```
respectively. Various non-standard encodings, such as 

```
..%c0%af
```

or 

```
..%ef%bc%8f
```
, may also work.

An application may require the user-supplied filename to start with the expected base folder, such as **/var/www/images**. In this case, it might be possible to include the required base folder followed by suitable traversal sequences. For example: 

```
filename=/var/www/images/../../../etc/passwd
```

An application may require the user-supplied filename to end with an expected file extension, such as .png. In this case, it might be possible to use a null byte to effectively terminate the file path before the required extension. For example: 

```
filename=../../../etc/passwd%00.png
```