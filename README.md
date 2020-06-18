# GitLab-CI pipeline
## Variables
Variables defines in `./vars/default_vars`

    registration_token: ...
    gitlab_host_ip: ...
    nexus_host_ip: ...
    ....
    and etc.

## Deploy GitLab-CI server (inventory: ext_gitlabci_server)
- start deployment GitLab-CI server: 

      - absible-playbook 1_deploy_gitlab.yml

- set password: `admin123`
- create group: `my_project_group` (private or public)
- create project: `my_repo1` (private or public)
- copy project registration token to `./vars/default_vars.yml` **(Project -> Settings -> CI/CD -> Runners)**
- set project variables **(Project -> Settings -> CI/CD -> Variables):**

      CI_NEXUS_HOST = nexus.example.com:18091
      CI_NEXUS_USER = gitlabci-user
      CI_NEXUS_PASS = gitpass123 

## Deploy Docker server (if it's the first start)
- start deployment Docker server (from repository: docker_systemd): 

      - absible-playbook deploy_containers.yml

- copy **private_key** to ./gitlab-ci/hosts/Docker-server/certs directory with **chmod 0600** or `cp -p` command

## Run and Register gitlab-runner
- start `2_setting_ci.yml` wich contains 2 plays:

      - absible-playbook 2_setting_ci.yml

     - install and register `gitlab-runner` in docker container (**resolve DNS** via `docker_container` parameter **etc_hosts** and in `config.template.toml` - **extra_hosts**)
     - Copy Nexus SSL-certificate for connection between Docker and Nexus servers. For Docker server resolve DNS names by adding hosts to `/etc/hosts` file.

## Configure Nexus server for GitLab-CI project
- create repository: **my_app1** (parameters: docker hosted, **https port - 18091**, enable Docker API)
- create Role with "admin" and "view" privileges to **my_app1** repository (example: GitLab-CI Role)
- create repository user: **gitlabci-user**. And grant Role created in the previous step 

## Push test project from ./gitlab-ci/repository_files/ to the created repository on GitLab-CI server
    - git remote add origin http://gitlab.example.com/my_project_group/my_repo1.git
    - git  push -u origin master