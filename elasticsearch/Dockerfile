FROM elasticsearch:2.4.5

RUN /usr/share/elasticsearch/bin/plugin install lmenezes/elasticsearch-kopf/2.1.1; \
	/usr/share/elasticsearch/bin/plugin install http://dl.bintray.com/content/imotov/elasticsearch-plugins/org/elasticsearch/elasticsearch-analysis-morphology/2.4.5/elasticsearch-analysis-morphology-2.4.5.zip

# File permissions
ARG PARENT_USER_ID
RUN usermod -u $PARENT_USER_ID elasticsearch

EXPOSE 9200 9300
ENTRYPOINT ["/docker-entrypoint.sh"]
CMD ["elasticsearch"]