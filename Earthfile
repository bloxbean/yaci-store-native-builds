VERSION 0.8

build:
  LOCALLY
  ARG PROFILE

  WORKDIR .

  #read branch name from branch file
  LET branch=$(cat branch)

  RUN rm -rf tmp
  RUN rm -rf output
  RUN mkdir -p tmp
  WORKDIR tmp
  RUN git clone --depth 1 --branch ${branch} https://github.com/bloxbean/yaci-store.git yaci-store

  IF [ "$PROFILE" = "n2c" ]
    RUN cp ../profiles/n2c.application.properties yaci-store/applications/all/src/main/resources/application.properties
  ELSE
    RUN cp ../profiles/default.application.properties yaci-store/applications/all/src/main/resources/application.properties
  END

  # Get version from gradle.properties
  RUN cat yaci-store/gradle.properties | grep version | cut -d'=' -f2 | tr -d '[:space:]' > version

  WORKDIR yaci-store

  RUN ./gradlew --no-daemon -i clean build nativeCompile

  RUN cp applications/all/build/native/nativeCompile/yaci-store-all yaci-store-all
  BUILD +zip --PROFILE=${PROFILE}

zip:
  LOCALLY
  ARG PROFILE
  ARG TARGETOS
  ARG TARGETARCH
  RUN rm -rf output

  RUN  echo $TARGETARCH > tmp/arch_name
  IF [ "$TARGETOS" = "darwin" ]
    RUN  echo "mac" > tmp/os_name
  ELSE IF [ "$TARGETOS" = "darwin" ]
     RUN  echo $TARGETOS > tmp/os_name
  END

  FROM alpine:latest
  WORKDIR /app
  RUN apk add --no-cache zip
  COPY tmp/os_name /tmp/os_name
  COPY tmp/arch_name /tmp/arch_name
  COPY tmp/version /tmp/version

  LET os_name=$(cat /tmp/os_name)
  LET arch_name=$(cat /tmp/arch_name)
  LET version=$(cat /tmp/version)

  IF [ "$PROFILE" = "n2c" ]
    LET zip_dir="yaci-store-${version}_${PROFILE}"
  ELSE
    LET zip_dir="yaci-store-${version}"
  END

  RUN mkdir -p /app/${zip_dir}
  RUN mkdir -p /app/${zip_dir}/config
  COPY tmp/yaci-store/LICENSE  /app/${zip_dir}/LICENSE
  COPY tmp/yaci-store/config  /app/${zip_dir}/config
  COPY tmp/yaci-store/applications/all/build/native/nativeCompile/yaci-store-all /app/${zip_dir}/yaci-store

  IF [ "$PROFILE" = "n2c" ]
    RUN cd /app && zip -r yaci-store-${version}-${os_name}_${arch_name}_${PROFILE}.zip .
    RUN ls /app
    SAVE ARTIFACT /app/yaci-store-${version}-${os_name}_${arch_name}_${PROFILE}.zip AS LOCAL output/yaci-store-${version}-${os_name}_${arch_name}_${PROFILE}.zip
  ELSE
    RUN cd /app && zip -r yaci-store-${version}-${os_name}_${arch_name}.zip .
    SAVE ARTIFACT /app/yaci-store-${version}-${os_name}_${arch_name}.zip AS LOCAL output/yaci-store-${version}-${os_name}_${arch_name}.zip
  END
