
```
 const fileStream = selectFile.stream ? selectFile.stream() : selectFile;
      const params = {
       Bucket: "osmefru",
        Key: selectFile.name,
        Body: fileStream,
      };
```


✅ ``` Bucket: "osmefru",     SUSTITUCIÓN POR >>>>    Bucket: "jtk-bucket-archivo", ```

---

## Permissions jtk-bucket-archivo

### Visión

```

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": [
        "arn:aws:s3:::YOUR_BUCKET_NAME/*",
        "arn:aws:s3:::YOUR_BUCKET_NAME"
      ]
    }
  ]
}

```

```
[
  {
    "AllowedHeaders": ["*"],
    "AllowedMethods": ["GET", "PUT", "POST", "DELETE", "HEAD"],
    "AllowedOrigins": ["http://localhost:3000"],
    "ExposeHeaders": ["ETag"]
  }
]

```

---------------------------------------------------------------------------------------


### Master

```

{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:*",
            "Resource": [
                "arn:aws:s3:::jtk-bucket-archivo/*",
                "arn:aws:s3:::jtk-bucket-archivo"
  
                 ]
        }
    ]
}

```

```

[
    {
        "AllowedHeaders": [
            "*"
        ],
        "AllowedMethods": [
            "GET",
            "PUT",
            "POST",
            "DELETE",
            "HEAD"
        ],
        "AllowedOrigins": [
            "http://localhost:3000"
        ],
        "ExposeHeaders": [
            "ETag"
        ]
    }
]

```




