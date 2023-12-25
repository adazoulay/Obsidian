# Running the project
```
docker compose -f docker-compose-dev.yml up --build
```

`docker-compose -f docker-compose-dev.yml restart web`

`docker-compose -f docker-compose-dev.yml exec`

### Logs
`make up`
`make down`
```
docker compose -f docker-compose-dev.yml logs celery_worker --follow | grep -E 'INFO/ForkPoolWorker-*' 
```
# Git workflow:
### Start
Create a PR from the `staging` branch
When starting a new ticket form staging
- `git pull origin staging` 
- The name of the branches should follow this structure:
	- `feat/<issue-id>-xxxxxxxxxx` 
	- `fix/<issue-id>-xxxxxxxxxx`
### Committing Changes
`git add .`
`git commit -m ""`
### Pushing Changes
**First time:**
- `git push -u origin feat/<issue-id>-xxxxxxxxxx`
Once branch head is set, can use
- `git push origin feat/<issue-id>-xxxxxxxxxx`
### Creating a PR
- Navigate to https://github.com/ysfbsf/promptify-api
- Then create the PR from there
- **NOTE:** Make sure you are creating the PR to **`Staging`**

## Making changes to `promptify_ai`
Make changes inside a submodule  
- `cd` inside the submodule directory.
- Make new branch in submodule 
- `git add/commit` the new changes.
- `git push` the new commit.
- `cd` back to the main repository.
- In `git status` you'll see that the submodule directory is modified.
- In `git diff` you'll see the old and new commit pointers.
- When you `git commit` in the main repository, it will update the pointer.
# DB and Migrations:
Make migration:
- `docker compose -f docker-compose-dev.yml exec web python3 manage.py makemigrations *app_name`
Migrate:
- `docker compose -f docker-compose-dev.yml exec web python3 manage.py migrate`

Access DB:
`docker compose exec db psql -U postgres -d promptify`
`make psql ``
password: **postgres**
# Prospector
`DJANGO_SETTINGS_MODULE=promptify_api.settings prospector > prospector-report.txt`
### Repos
#### [[3 promptify-ai]]
- https://github.com/ysfbsf/promptify-ai
#### [[4 promptify-api]]
- https://github.com/ysfbsf/promptify-api
### Setup:
After installing `promptify-ai` and `promptify-api`
From `promptify-api` root:
- `docker compose up --build`
## Superuser
Create superuser
`docker compose exec web python manage.py createsuperuser`

Admin URL:
http://localhost:8000/admin/
#### Credentials
username: adazoulay
email: [adam@promptify.com](mailto:adam@promptify.com)
Password: BostonC123!

## AWS:
[https://d-9067910c9d.awsapps.com/start/](https://d-9067910c9d.awsapps.com/start/)
username: adam
pw: BostonC123!