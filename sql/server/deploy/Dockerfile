################################################################################
# Build the artifacts
################################################################################

# we use x86-64 in the cloud so we need to duplicate the build steps
# here - we can't reuse existing images that might be arm64
FROM --platform=linux/amd64 skiplabs/skdb-base as skdb

COPY . /skfs

WORKDIR /skfs

RUN make clean && make build/skdb && make build/init.sql

# we use native for skgw as gradle does NOT like being emulated and
# this is faster anyway
FROM eclipse-temurin:20 as skgw

COPY . /skfs

WORKDIR /skfs/sql/server/skgw
RUN ../gradlew jar --no-daemon --console plain

################################################################################
# Build the production image by copying out artifacts
################################################################################

FROM --platform=linux/amd64 eclipse-temurin:20 as prod

WORKDIR /skfs

COPY --from=skdb /skfs/build ./build
COPY --from=skgw /skfs/sql/server/skgw/build/libs/skgw.jar /skfs/build/server.jar
COPY --from=skgw /skfs/sql/server/deploy/start_prod.sh /skfs/build/start.sh

CMD /skfs/build/start.sh
