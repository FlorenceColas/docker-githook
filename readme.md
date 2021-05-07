# Manage git hooks with Docker

This repository explains how to manage git hooks in a dockerized environment. 
It is interesting to share hooks between team members.

There are three points to configure:
- the docker/hooks directory: this directory contains the hooks files. These will be the reference hooks files.

```
-- docker
+-- hooks
|   +-- post-checkout
|   +-- pre-commit
```

- the dockerfile.githooks: this dockerfile contains a sequence of commands which will:
  - make the files located in docker/hooks executable
  - clean up all the existing symlinks in the .git/hooks directory of the project
  - create symlinks in this .git/hooks directory to the reference hooks files
  
```dockerfile
FROM busybox:latest

ENTRYPOINT sh -c "cd /tmp/docker/hooks && ls | xargs chmod +x && cd /tmp/.git/hooks && find /tmp/.git/hooks/ -maxdepth 1 -type l -delete && find ../../docker/hooks -type f -exec ln -sf {} /tmp/.git/hooks/ \;"
```

- the docker-compose.yml file: this file contains the definition of the container which will be build with 
  the dockerfile.githooks.

```yaml
  # Git hooks
  githooks-installer:
    build:
      context: .
      dockerfile: './docker/dockerfile.githooks'
    volumes:
      - ./.git:/tmp/.git
      - ./docker/hooks:/tmp/docker/hooks
```

Run `docker-compose up -d` to create the symlinks.
Now you can test running a `git checkout` or `git commit` command. You will see in your terminal 
respectively `post-checkout hook run` and `pre-commit hook run`.