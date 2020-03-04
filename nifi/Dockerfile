FROM apache/nifi:latest
ADD --chown=nifi:nifi https://repo1.maven.org/maven2/org/apache/nifi/nifi-kite-nar/1.11.3/nifi-kite-nar-1.11.3.nar /opt/nifi/nifi-current/lib
COPY --chown=nifi:nifi ./start.sh /opt/nifi/scripts/start.sh