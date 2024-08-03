+++
title = 'Use Docker in Windows 01'
date = 2024-08-03T08:38:08Z
draft = false
+++
Demostrate:  
```
docker run -it --name your-container-name --rm \
         -v //c/path/to/app:/app \
         your-image-name
```

In this demo:  

``` //c/path/to/app ```

In windows is C:\path\to\app
