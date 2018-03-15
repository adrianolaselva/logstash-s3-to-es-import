FROM docker.elastic.co/logstash/logstash:5.6.5
RUN rm -f /usr/share/logstash/pipeline/logstash.conf/

RUN mkdir /tmp/servidor/
RUN mkdir /usr/share/logstash/queue/
RUN mkdir /usr/share/logstash/logs/
RUN mkdir /tmp/logstash/

RUN chown -R logstash.logstash /tmp/servidor/
RUN chmod 777 -R /tmp/servidor/

RUN chown -R logstash.logstash /usr/share/logstash/data/
RUN chmod 777 -R /usr/share/logstash/data/

RUN chown -R logstash.logstash /usr/share/logstash/queue/
RUN chmod 777 -R /usr/share/logstash/queue/

RUN chown -R logstash.logstash /usr/share/logstash/logs/
RUN chmod 777 -R /usr/share/logstash/logs/

RUN chown -R logstash.logstash /tmp/logstash/
RUN chmod 777 -R /tmp/logstash/

#Instalação ded plugin logstash
#RUN /usr/share/logstash/bin/logstash-plugin install logstash-output-jdbc