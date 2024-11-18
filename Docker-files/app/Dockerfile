FROM openjdk:11 AS BUILD_IMAGE
RUN apt-get update -y --fix-missing && apt-get install -y maven git --security-opt apparmor=unconfined
RUN git clone https://github.com/devopshydclub/vprofile-project.git
RUN cd vprofile-project && git checkout docker && mvn install 

FROM tomcat:9-jre11
LABEL "Project"="Vprofile"
LABEL "Author"="Imran"
COPY --from=BUILD_IMAGE vprofile-project/target/vprofile-v2.war /usr/local/tomcat/webapps/ROOT.war

EXPOSE 8080
CMD ["catalina.sh", "run"]
