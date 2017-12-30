FROM docker.elastic.co/logstash/logstash:6.0.1
RUN rm -f /usr/share/logstash/pipeline/logstash.conf
RUN curl -vvvvvvvvvvvI https://rubygems.org/specs.4.8.gz
RUN JARS_SKIP=true logstash-plugin install --version 0.3.1 logstash-filter-kubernetes
ADD pipeline/ /usr/share/logstash/pipeline/
ADD config/ /usr/share/logstash/config/
