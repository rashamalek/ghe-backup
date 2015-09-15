# ghe-backup
docker stups AWS based backup for github enterprise at zalando

## create docker image
docker build --rm -t [repo name]:[tag] .  
e.g.  
docker build --rm -t pierone.stups.zalan.do/bus/ghe-backup:0.0.1 .  

## run the image
docker run -dit --name [repo name]:[tag]  
e.g.  
docker run -dit --name ghe-backup pierone.stups.zalan.do/bus/ghe-backup:0.0.1

## upload to pierone
docker push [repo name]:[tag]  
e.g.
docker push pierone.stups.zalan.do/bus/ghe-backup:0.0.1
