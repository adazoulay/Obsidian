
# AWS
### Flow
- EC2: 
		- Main instance: `generate` and `adam_test`

- `ssh adam_test`

- bottom left `><` button
	- remote ssh
- `adam_test`
	- New vs code window

- `tmux`:
	- ssh session closes
	- session persists 
	- `tmux` -> close terminal -> `tmux a`


To run
- Focal Retouch Service Example:
	- `python server/app.py`

##### To copy stuff:
- `rsync -avz source_path/ adam_test:~/dest_path/`
	- Note `/` at end 


# AWS services
> Ask gpt 
- SQS 
- S3
- DynamoDB
- IAM
- EC2
	- ECS (soon)
- Route 53
- AWS lambda 




#### for Youssef
Why isn't this used:
- /image/remove_background/correct/V2

- Use chat service to generate image
	- Get transparent image form Raph
	- Pass as s3 url: `s3://test-rr694/adam_experiments/watch_1-removebg_expand.png`
	- if need to download look at : S3 utils: s3 link to pre-signed url
- evaluate images with `image/evaluation`
- use image generate no upscale to regenerate tries (up to 3)

If doesn't work:
- prompt_library.py -> VISION_EVALUATION_PROMPT tweak 