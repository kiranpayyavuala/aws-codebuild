#buildspec.yml
--------------
version: 0.2

phases:
  pre_build:
    commands:
      - echo Entered the install phase…
      - apt-get update -y
      - apt-get install -y openjdk-8-jdk
      - apt-get install -y maven
      - aws --version
      - echo Logging in to Amazon ECR...
      - $(aws ecr get-login --no-include-email | sed 's|https://||')
      - REPOSITORY_URI=719615471860.dkr.ecr.us-east-1.amazonaws.com/springbootapp
      - COMMIT_HASH=$(echo $CODEBUILD_RESOLVED_SOURCE_VERSION | cut -c 1-7)
      - IMAGE_TAG=build-$(echo $CODEBUILD_BUILD_ID | awk -F":" '{print $2}')
  build:
    commands:
      - mvn clean install
      - echo Build started on `date`
      - echo Building the Docker image...
      - docker build -t $REPOSITORY_URI:latest .
      - docker tag $REPOSITORY_URI:latest $REPOSITORY_URI:$IMAGE_TAG
      - docker images
  post_build:
    commands:
      - echo Build completed on `date`
      - echo Pushing the Docker images...
      - docker push $REPOSITORY_URI:latest
      - docker push $REPOSITORY_URI:$IMAGE_TAG
      - docker push $REPOSITORY_URI:$IMAGE_TAG
      - echo Writing image definitions file...
      - printf '[{"name":"spp","imageUri":"%s"}]' $REPOSITORY_URI:$IMAGE_TAG > imagedefinitions.json
      - cat imagedefinitions.json 
artifacts:
    files: imagedefinitions.json
