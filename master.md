
```
 const fileStream = selectFile.stream ? selectFile.stream() : selectFile;
      const params = {
       Bucket: "osmefru",
        Key: selectFile.name,
        Body: fileStream,
      };
```


✅ Bucket: "osmefru",     SUSTITUCIÓN POR >>>>    Bucket: "jtk-bucket-archivo",
