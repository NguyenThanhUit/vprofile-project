# Bước 1: Sử dụng image OpenJDK có Maven đã được cài sẵn
FROM maven:3.8.4-openjdk-11-slim AS BUILD_IMAGE

# Bước 2: Cloning project từ GitHub
RUN git clone https://github.com/devopshydclub/vprofile-project.git

# Bước 3: Chuyển sang thư mục của dự án và cài đặt dự án
RUN cd vprofile-project && git checkout docker && mvn install

# Bước 4: Sử dụng image Tomcat và copy file WAR đã build vào thư mục Tomcat
FROM tomcat:9-jre11
LABEL "Project"="Vprofile"
LABEL "Author"="Imran"

# Copy file WAR đã build từ image trước vào Tomcat
COPY --from=BUILD_IMAGE vprofile-project/target/vprofile-v2.war /usr/local/tomcat/webapps/ROOT.war

# Mở cổng 8080 để Tomcat có thể phục vụ ứng dụng
EXPOSE 8080

# Lệnh chạy Tomcat khi container khởi động
CMD ["catalina.sh", "run"]
