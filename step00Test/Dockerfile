# Step 1: Build stage
FROM openjdk:17-jdk-alpine AS build

# 작업 디렉토리 생성
WORKDIR /usr/src/myapp

# 소스 코드 복사
COPY . /usr/src/myapp

# Java 소스 컴파일
RUN javac Test.java

# Step 2: Run stage
FROM openjdk:17-alpine

# 빌드한 결과물을 복사
WORKDIR /usr/src/myapp
COPY --from=build /usr/src/myapp /usr/src/myapp

# 실행 명령
CMD ["java", "Test"]