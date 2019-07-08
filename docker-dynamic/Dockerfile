FROM tomcat

COPY DevOps_Zip_War_File/companyNews.war /usr/local/tomcat/webapps/

RUN rm -r /usr/local/tomcat/webapps/ROOT

RUN mv /usr/local/tomcat/webapps/companyNews.war /usr/local/tomcat/webapps/ROOT.war
