# docker-elasticsearch

Ready to use, lean and highly configurable Elasticsearch container image.

[![Docker Repository on Quay.io](https://quay.io/repository/coolpalani/docker-elasticsearch/status "Docker Repository on Quay.io")](https://quay.io/repository/coolpalani/docker-elasticsearch)

## Current software

* Alpine Linux 3.9.4
* OpenJDK JRE 8u212
* Elasticsearch 6.7.2

## Run

Ready to use node for cluster `elasticsearch-default`:
```
docker run --name elasticsearch \
	--detach \
	--privileged \
	--volume /path/to/data_folder:/data \
        quay.io/coolpalani/docker-elasticsearch:6.7.2
```

Ready to use node for cluster `myclustername`:
```
docker run --name elasticsearch \
	--detach \
	--privileged \
	--volume /path/to/data_folder:/data \
	-e CLUSTER_NAME=myclustername \
        quay.io/coolpalani/docker-elasticsearch:6.7.2
```

Ready to use node for cluster `elasticsearch-default`, with 8GB heap allocated to Elasticsearch:
```
docker run --name elasticsearch \
	--detach \
	--privileged \
	--volume /path/to/data_folder:/data \
	-e ES_JAVA_OPTS="-Xms8g -Xmx8g" \
        quay.io/coolpalani/docker-elasticsearch:6.7.2
```

Ready to use node with plugins (x-pack and repository-gcs) pre installed. Already installed plugins are ignored:
```
docker run --name elasticsearch \
	--detach \
	--privileged \
	--volume /path/to/data_folder:/data \
	-e ES_JAVA_OPTS="-Xms8g -Xmx8g" \
	-e ES_PLUGINS_INSTALL="repository-gcs,x-pack" \
        quay.io/coolpalani/docker-elasticsearch:6.7.2
```

**Master-only** node for cluster `elasticsearch-default`:
```
docker run --name elasticsearch \
	--detach \
	--privileged \
	--volume /path/to/data_folder:/data \
	-e NODE_DATA=false \
	-e HTTP_ENABLE=false \
        quay.io/coolpalani/docker-elasticsearch:6.7.2
```

**Data-only** node for cluster `elasticsearch-default`:
```
docker run --name elasticsearch \
	--detach --volume /path/to/data_folder:/data \
	--privileged \
	-e NODE_MASTER=false \
	-e HTTP_ENABLE=false \
        quay.io/coolpalani/docker-elasticsearch:6.7.2
```

**Data-only** node for cluster `elasticsearch-default` with shard allocation awareness:
```
docker run --name elasticsearch \
	--detach --volume /path/to/data_folder:/data \
        --volume /etc/hostname:/dockerhost \
	--privileged \
	-e NODE_MASTER=false \
	-e HTTP_ENABLE=false \
    -e SHARD_ALLOCATION_AWARENESS=dockerhostname \
    -e SHARD_ALLOCATION_AWARENESS_ATTR="/dockerhost" \
        quay.io/coolpalani/docker-elasticsearch:6.7.2
```

**Client-only** node for cluster `elasticsearch-default`:
```
docker run --name elasticsearch \
	--detach \
	--privileged \
	--volume /path/to/data_folder:/data \
	-e NODE_MASTER=false \
	-e NODE_DATA=false \
        quay.io/coolpalani/docker-elasticsearch:6.7.2
```
### Backup
Mount a shared folder (for example via NFS) to `/backup` and make sure the `elasticsearch` user
has write access. Then, set the `REPO_LOCATIONS` environment variable to `"/backup"` and create
a backup repository:

`backup_repository.json`:
```
{
  "type": "fs",
  "settings": {
    "location": "/backup",
    "compress": true
  }
}
```

```bash
curl -XPOST http://<container_ip>:9200/_snapshot/nas_repository -d @backup_repository.json`
```

Now, you can take snapshots using:
```bash
curl -f -XPUT "http://<container_ip>:9200/_snapshot/nas_repository/snapshot_`date --utc +%Y_%m_%dt%H_%M`?wait_for_completion=true"
```
