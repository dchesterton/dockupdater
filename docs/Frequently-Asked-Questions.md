
## Configuration

* [Timezone Configuration](#timezone-configuration)

### Timezone Configuration

To more closely monitor docupdater' actions and for accurate log ingestion, you can change the timezone of the container from UTC by setting the [`TZ`](http://www.gnu.org/software/libc/manual/html_node/TZ-Variable.html) environment variable like so:

```
docker run -d --name docupdater \
  -e TZ=America/Chicago \
  -v /var/run/docker.sock:/var/run/docker.sock \
  docupdater/docupdater
```

### Exclude docupdater to be updated

Your can add label `docupdater.disable="true"` on the container or service to disable auto update.

If your run a standalone container for docupdater:
```
docker run -d --name docupdater \
  -v /var/run/docker.sock:/var/run/docker.sock \
  --label docupdater.disable="true"
  docupdater/docupdater
```

If your run docupdater on a stack:
```
version: "3.6"

services:
  docupdater:
    image: docupdater/docupdater
    deploy:
      labels:
        docupdater.disable: "true"
```