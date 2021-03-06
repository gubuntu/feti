SHELL := /bin/bash
PROJECT_ID := feti

# ----------------------------------------------------------------------------
#    P R O D U C T I O N     C O M M A N D S
# ----------------------------------------------------------------------------

default: web

run: build web

deploy: run migrate collectstatic
	@echo
	@echo "------------------------------------------------------------------"
	@echo "Bringing up fresh instance "
	@echo "You can access it on http://localhost:63100"
	@echo "------------------------------------------------------------------"

build:
	@echo
	@echo "------------------------------------------------------------------"
	@echo "Building in production mode"
	@echo "------------------------------------------------------------------"
	@docker-compose -p $(PROJECT_ID) build

web:
	@echo
	@echo "------------------------------------------------------------------"
	@echo "Running in production mode"
	@echo "------------------------------------------------------------------"
	@docker-compose -p $(PROJECT_ID) up -d web
	# @docker-compose -p $(PROJECT_ID) up -d dbbackups
	# @dipall

migrate:
	@echo
	@echo "------------------------------------------------------------------"
	@echo "Running migrate static in production mode"
	@echo "------------------------------------------------------------------"
	@docker-compose -p $(PROJECT_ID) run uwsgi python manage.py migrate

makemigrations:
	@echo
	@echo "------------------------------------------------------------------"
	@echo "Running makemigrations in production mode"
	@echo "------------------------------------------------------------------"
	@docker-compose -p $(PROJECT_ID) run uwsgi python manage.py makemigrations

updatelongdescription:
	@echo
	@echo "------------------------------------------------------------------"
	@echo "Running updatelongdescription in production mode"
	@echo "------------------------------------------------------------------"
	@docker-compose -p $(PROJECT_ID) run uwsgi python manage.py updatelongdescription

updatecampuscourseentries:
	@echo
	@echo "------------------------------------------------------------------"
	@echo "Running updatecampuscourseentries in production mode"
	@echo "------------------------------------------------------------------"
	@docker-compose -p $(PROJECT_ID) run uwsgi python manage.py updatecampuscourseentries

rebuildindex:
	@echo
	@echo "------------------------------------------------------------------"
	@echo "Running rebuild_index in production mode"
	@echo "------------------------------------------------------------------"
	@docker-compose -p $(PROJECT_ID) run uwsgi python manage.py rebuild_index

updateindex:
	@echo
	@echo "------------------------------------------------------------------"
	@echo "Running update_index in production mode"
	@echo "------------------------------------------------------------------"
	@docker-compose -p $(PROJECT_ID) run uwsgi python manage.py update_index

collectstatic:
	@echo
	@echo "------------------------------------------------------------------"
	@echo "Collecting static in production mode"
	@echo "------------------------------------------------------------------"
	#@docker-compose -p $(PROJECT_ID) run uwsgi python manage.py collectstatic --noinput
	#We need to run collect static in the same context as the running
	# uwsgi container it seems so I use docker exec here
	@docker exec -t -i $(PROJECT_ID)_uwsgi_1 python manage.py collectstatic --noinput
	#
	# RM : this is needed to copy some library over
	# I don't know a better way to do this
	sudo cp ../django_project/feti/static/feti/js/ZeroClipboard.swf static/js/


kill:
	@echo
	@echo "------------------------------------------------------------------"
	@echo "Killing in production mode"
	@echo "------------------------------------------------------------------"

	@docker-compose -p $(PROJECT_ID) kill

rm: kill
	@echo
	@echo "------------------------------------------------------------------"
	@echo "Removing production instance!!! "
	@echo "------------------------------------------------------------------"
	@docker-compose -p $(PROJECT_ID) rm

logs:
	@echo
	@echo "------------------------------------------------------------------"
	@echo "Showing uwsgi logs in production mode"
	@echo "------------------------------------------------------------------"
	@docker-compose -p $(PROJECT_ID) logs uwsgi

nginxlogs:
	@echo
	@echo "------------------------------------------------------------------"
	@echo "Showing nginx logs in production mode"
	@echo "------------------------------------------------------------------"
	@docker-compose -p $(PROJECT_ID) logs web

shell:
	@echo
	@echo "------------------------------------------------------------------"
	@echo "Shelling in in production mode"
	@echo "------------------------------------------------------------------"
	@docker-compose -p $(PROJECT_ID) run uwsgi /bin/bash

dbshell:
	@echo
	@echo "------------------------------------------------------------------"
	@echo "Shelling in in production database"
	@echo "------------------------------------------------------------------"
	@docker exec -t -i $(PROJECT_ID)_db_1 psql -U docker -h localhost gis

sentry:
	@echo
	@echo "--------------------------"
	@echo "Running sentry production mode"
	@echo "--------------------------"
	@docker-compose  -p $(PROJECT_ID) up -d sentry
	# @dipall

maillogs:
	@echo
	@echo "------------------------------------------------------------------"
	@echo "Showing smtp logs in production mode"
	@echo "------------------------------------------------------------------"
	@docker exec -t -i $(PROJECT_ID)_smtp_1 tail -f /var/log/mail.log

mailerrorlogs:
	@echo
	@echo "------------------------------------------------------------------"
	@echo "Showing smtp error logs in production mode"
	@echo "------------------------------------------------------------------"
	@docker exec -t -i $(PROJECT_ID)_smtp_1 tail -f /var/log/mail.err

mapproxylogs:
	@echo
	@echo "--------------------------"
	@echo "Showing mapproxy logs in production mode"
	@echo "--------------------------"
	@docker-compose -p $(PROJECT_ID) logs mapproxy

seed:
	@echo
	@echo "--------------------------"
	@echo "Seeding mapproxy in production mode"
	@echo "--------------------------"
	@docker-compose -p $(PROJECT_ID) run mapproxy /usr/local/bin/mapproxy-seed -f /mapproxy/mapproxy.yaml /mapproxy/seed.yaml

map:
	@echo
	@echo "--------------------------"
	@echo "Getting current 6 hour slot as pdf map"
	@echo "Make sure you have started the services with make run first"
	@echo "--------------------------"
	# Experimental way we had of getting it using QGIS Server
	#wget -O map.pdf "http://172.17.0.6/cgi-bin/qgis_mapserv.fcgi?map=/web/jk-floods.qgs&SERVICE=wms&LAYERS=Rw&REQUEST=GetPrint&TEMPLATE=6Hour&map0:EXTENT=11862318.594,-714081.378,11922154.957,-675870.096&map0:ROTATION=0&FORMAT=pdf&DPI=300"
	docker exec -t -i $(PROJECT_ID)_cron_1 xvfb-run python /home/web/cron-scripts/pdf_report_generator.py
	@echo "--------------------------"
	@echo "PDF Generated - written to reports"
	@echo "--------------------------"
	evince reports/jk-floods-latest.pdf

qgis:
	@echo
	@echo "------------------------------------------------------------------"
	@echo "Running QGIS desktop in production mode"
	@echo "Make sure you have started the services with make run first"
	@echo "------------------------------------------------------------------"
	@xhost +; docker run --rm --name="qgis-desktop-2-8" \
	-i -t \
	--link $(PROJECT_ID)_db_1:db \
	--link $(PROJECT_ID)_qgisserver_1:qgisserver \
	--link $(PROJECT_ID)_mapproxy_1:mapproxy \
	--volumes-from $(PROJECT_ID)_qgisserver_1 \
	-v ${HOME}:/home/$(USER) \
	-v /tmp/.X11-unix:/tmp/.X11-unix \
	-e DISPLAY=unix$(DISPLAY) \
	kartoza/qgis-desktop:2.8.0; xhost -

reload:
	@echo
	@echo "------------------------------------------------------------------"
	@echo "Reload django project in production mode"
	@echo "------------------------------------------------------------------"
	@docker exec -t -i $(PROJECT_ID)_uwsgi_1 uwsgi --reload  /tmp/django.pid



# ----------------------------------------------------------------------------
#    S T A G I N G     C O M M A N D S
# ----------------------------------------------------------------------------

staging-run: staging-build staging-web

staging-deploy: staging-run staging-migrate staging-collectstatic
	@echo
	@echo "------------------------------------------------------------------"
	@echo "Bringing up fresh staging instance "
	@echo "You can access it on http://localhost:63101"
	@echo "------------------------------------------------------------------"

staging-build:
	@echo
	@echo "------------------------------------------------------------------"
	@echo "Building in staging mode"
	@echo "------------------------------------------------------------------"
	@docker-compose -p $(PROJECT_ID)-staging build

staging-web:
	@echo
	@echo "------------------------------------------------------------------"
	@echo "Running in staging mode"
	@echo "------------------------------------------------------------------"
	@docker-compose -p $(PROJECT_ID)-staging up -d stagingweb
	# @dipall

staging-migrate:
	@echo
	@echo "------------------------------------------------------------------"
	@echo "Running migrate static in staging mode"
	@echo "------------------------------------------------------------------"
	@docker-compose -p $(PROJECT_ID)-staging run uwsgi python manage.py migrate

staging-updatelongdescription:
	@echo
	@echo "------------------------------------------------------------------"
	@echo "Running updatelongdescription in staging mode"
	@echo "------------------------------------------------------------------"
	@docker-compose -p $(PROJECT_ID)-staging run uwsgi python manage.py updatelongdescription

staging-updatecampuscourseentries:
	@echo
	@echo "------------------------------------------------------------------"
	@echo "Running updatecampuscourseentries in staging mode"
	@echo "------------------------------------------------------------------"
	@docker-compose -p $(PROJECT_ID)-staging run uwsgi python manage.py updatecampuscourseentries

staging-rebuildindex:
	@echo
	@echo "------------------------------------------------------------------"
	@echo "Running rebuild_index in staging mode"
	@echo "------------------------------------------------------------------"
	@docker-compose -p $(PROJECT_ID)-staging run uwsgi python manage.py rebuild_index

staging-updateindex:
	@echo
	@echo "------------------------------------------------------------------"
	@echo "Running update_index in staging mode"
	@echo "------------------------------------------------------------------"
	@docker-compose -p $(PROJECT_ID)-staging run uwsgi python manage.py update_index

staging-collectstatic:
	@echo
	@echo "------------------------------------------------------------------"
	@echo "Collecting static in staging mode"
	@echo "------------------------------------------------------------------"
	#@docker-compose -p $(PROJECT_ID)-staging run uwsgi python manage.py collectstatic --noinput
	#We need to run collect static in the same context as the running
	# uwsgi container it seems so I use docker exec here
	@docker exec -t -i $(PROJECT_ID)staging_uwsgi_1 python manage.py collectstatic --noinput
	#
	# RM : this is needed to copy some library over
	# I don't know a better way to do this
	sudo cp ../django_project/feti/static/feti/js/ZeroClipboard.swf static/js/

staging-kill:
	@echo
	@echo "------------------------------------------------------------------"
	@echo "Killing in staging mode"
	@echo "------------------------------------------------------------------"

	@docker-compose -p $(PROJECT_ID)-staging kill

staging-rm: kill
	@echo
	@echo "------------------------------------------------------------------"
	@echo "Removing staging instance!!! "
	@echo "------------------------------------------------------------------"
	@docker-compose -p $(PROJECT_ID)-staging rm

staging-logs:
	@echo
	@echo "------------------------------------------------------------------"
	@echo "Showing uwsgi logs in staging mode"
	@echo "------------------------------------------------------------------"
	@docker-compose -p $(PROJECT_ID)-staging logs uwsgi

staging-nginxlogs:
	@echo
	@echo "------------------------------------------------------------------"
	@echo "Showing nginx logs in staging mode"
	@echo "------------------------------------------------------------------"
	@docker-compose -p $(PROJECT_ID)-staging logs web

staging-shell:
	@echo
	@echo "------------------------------------------------------------------"
	@echo "Shelling in in staging mode"
	@echo "------------------------------------------------------------------"
	@docker-compose -p $(PROJECT_ID)-staging run uwsgi /bin/bash

staging-dbshell:
	@echo
	@echo "------------------------------------------------------------------"
	@echo "Shelling in in staging database"
	@echo "------------------------------------------------------------------"
	@docker exec -t -i $(PROJECT_ID)staging_db_1 psql -U docker -h localhost gis

staging-qgis:
	@echo
	@echo "------------------------------------------------------------------"
	@echo "Running QGIS desktop in staging mode"
	@echo "Make sure you have started the services with make run first"
	@echo "------------------------------------------------------------------"
	@xhost +; docker run --rm --name="qgis-desktop-2-8.0" \
	-i -t \
	--link $(PROJECT_ID)staging_db_1:db \
	--link $(PROJECT_ID)staging_qgisserver_1:qgisserver \
	--volumes-from $(PROJECT_ID)_qgisserver_1 \
	-v ${HOME}:/home/$(USER) \
	-v /tmp/.X11-unix:/tmp/.X11-unix \
	-e DISPLAY=unix$(DISPLAY) \
	kartoza/qgis-desktop:2.8; xhost -

# ----------------------------------------------------------------------------
#    D E V     C O M M A N D S
# ----------------------------------------------------------------------------

dev-run: dev-build dev-web

dev-deploy: dev-run dev-migrate dev-collectstatic
	@echo
	@echo "------------------------------------------------------------------"
	@echo "Bringing up fresh dev instance "
	@echo "You can access it on http://localhost:63101"
	@echo "------------------------------------------------------------------"

dev-build:
	@echo
	@echo "------------------------------------------------------------------"
	@echo "Building in dev mode"
	@echo "------------------------------------------------------------------"
	@docker-compose -f docker-compose-dev.yml -p $(PROJECT_ID)-dev build

dev-web:
	@echo
	@echo "------------------------------------------------------------------"
	@echo "Running in dev mode"
	@echo "------------------------------------------------------------------"
	@docker-compose -f docker-compose-dev.yml -p $(PROJECT_ID)-dev up -d web
	# @dipall

dev-migrate:
	@echo
	@echo "------------------------------------------------------------------"
	@echo "Running migrate static in dev mode"
	@echo "------------------------------------------------------------------"
	@docker-compose -f docker-compose-dev.yml -p $(PROJECT_ID)-dev run uwsgi python manage.py migrate

dev-updatelongdescription:
	@echo
	@echo "------------------------------------------------------------------"
	@echo "Running updatelongdescription in dev mode"
	@echo "------------------------------------------------------------------"
	@docker-compose -p $(PROJECT_ID)-dev run uwsgi python manage.py updatelongdescription

dev-updatecampuscourseentries:
	@echo
	@echo "------------------------------------------------------------------"
	@echo "Running updatecampuscourseentries in dev mode"
	@echo "------------------------------------------------------------------"
	@docker-compose -p $(PROJECT_ID)-dev run uwsgi python manage.py updatecampuscourseentries

dev-rebuildindex:
	@echo
	@echo "------------------------------------------------------------------"
	@echo "Running rebuild_index in dev mode"
	@echo "------------------------------------------------------------------"
	@docker-compose -p $(PROJECT_ID)-dev run uwsgi python manage.py rebuild_index

dev-updateindex:
	@echo
	@echo "------------------------------------------------------------------"
	@echo "Running update_index in dev mode"
	@echo "------------------------------------------------------------------"
	@docker-compose -p $(PROJECT_ID)-dev run uwsgi python manage.py update_index

dev-collectstatic:
	@echo
	@echo "------------------------------------------------------------------"
	@echo "Collecting static in dev mode"
	@echo "------------------------------------------------------------------"
	#@docker-compose -f docker-compose-dev.yml -p $(PROJECT_ID)-dev run uwsgi python manage.py collectstatic --noinput
	#We need to run collect static in the same context as the running
	# uwsgi container it seems so I use docker exec here
	@docker exec -t -i $(PROJECT_ID)dev_uwsgi_1 python manage.py collectstatic --noinput
	#
	# RM : this is needed to copy some library over
	# I don't know a better way to do this
	sudo cp ../django_project/feti/static/feti/js/ZeroClipboard.swf static/js/

dev-kill:
	@echo
	@echo "------------------------------------------------------------------"
	@echo "Killing in dev mode"
	@echo "------------------------------------------------------------------"

	@docker-compose -f docker-compose-dev.yml -p $(PROJECT_ID)-dev kill

dev-rm: kill
	@echo
	@echo "------------------------------------------------------------------"
	@echo "Removing dev instance!!! "
	@echo "------------------------------------------------------------------"
	@docker-compose -f docker-compose-dev.yml -p $(PROJECT_ID)-dev rm

dev-logs:
	@echo
	@echo "------------------------------------------------------------------"
	@echo "Showing uwsgi logs in dev mode"
	@echo "------------------------------------------------------------------"
	@docker-compose -f docker-compose-dev.yml -p $(PROJECT_ID)-dev logs uwsgi

dev-nginxlogs:
	@echo
	@echo "------------------------------------------------------------------"
	@echo "Showing nginx logs in dev mode"
	@echo "------------------------------------------------------------------"
	@docker-compose -f docker-compose-dev.yml -p $(PROJECT_ID)-dev logs web

dev-shell:
	@echo
	@echo "------------------------------------------------------------------"
	@echo "Shelling in in dev mode"
	@echo "------------------------------------------------------------------"
	@docker-compose -f docker-compose-dev.yml -p $(PROJECT_ID)-dev run uwsgi /bin/bash

dev-dbshell:
	@echo
	@echo "------------------------------------------------------------------"
	@echo "Shelling in in dev database"
	@echo "------------------------------------------------------------------"
	@docker exec -t -i $(PROJECT_ID)dev_db_1 psql -U docker -h localhost gis

dev-qgis:
	@echo
	@echo "------------------------------------------------------------------"
	@echo "Running QGIS desktop in dev mode"
	@echo "Make sure you have started the services with make run first"
	@echo "------------------------------------------------------------------"
	@xhost +; docker run --rm --name="qgis-desktop-2-8.0" \
	-i -t \
	--link $(PROJECT_ID)dev_db_1:db \
	--link $(PROJECT_ID)dev_qgisserver_1:qgisserver \
	--volumes-from $(PROJECT_ID)_qgisserver_1 \
	-v ${HOME}:/home/$(USER) \
	-v /tmp/.X11-unix:/tmp/.X11-unix \
	-e DISPLAY=unix$(DISPLAY) \
	kartoza/qgis-desktop:2.8; xhost -
