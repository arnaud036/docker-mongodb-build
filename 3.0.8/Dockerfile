FROM ubuntu:14.04
MAINTAINER Arnaud Delabarre <delabarre.arnaud@gmail.com>

ENV MONGO_VERSION 3.0.8
ENV MONGO_SRC /tmp/mongodb
ENV MONGO_TOOLS_SRC /tmp/mongo-tools

RUN apt-get update && \
    apt-get install -y build-essential libssl-dev libsasl2-dev python2.7 scons git pkg-config curl python-httplib2 dpkg-dev rpm debhelper fakeroot createrepo && \
    curl -sL https://storage.googleapis.com/golang/go1.5.3.linux-amd64.tar.gz | tar xzv -C /usr/local/

ENV PATH $PATH:/usr/local/go/bin

# building mongodb from sources
RUN git clone https://github.com/mongodb/mongo.git ${MONGO_SRC}

WORKDIR ${MONGO_SRC}
RUN git checkout tags/r${MONGO_VERSION} -b ${MONGO_VERSION}

RUN mkdir -p /tmp/mongodb-linux-x86_64-${MONGO_VERSION} && \
    scons --64 -j$(nproc) --ssl --use-sasl-client install --prefix=/tmp/mongodb-linux-x86_64-${MONGO_VERSION} && \
    cp ./distsrc/GNU-AGPL-3.0 /tmp/mongodb-linux-x86_64-${MONGO_VERSION} && \
    cp ./distsrc/README /tmp/mongodb-linux-x86_64-${MONGO_VERSION} && \
    cp ./distsrc/THIRD-PARTY-NOTICES /tmp/mongodb-linux-x86_64-${MONGO_VERSION}

# building mongodb-tools from sources
RUN git clone https://github.com/mongodb/mongo-tools.git ${MONGO_TOOLS_SRC}

WORKDIR ${MONGO_TOOLS_SRC}
RUN git checkout tags/r${MONGO_VERSION} -b ${MONGO_VERSION}

ENV GOPATH ${MONGO_TOOLS_SRC}/.gopath:${MONGO_TOOLS_SRC}/vendor
RUN ./set_gopath.sh && \
    mkdir -p bin && \
    LD_FLAGS="\"-X github.com/mongodb/mongo-tools/common/options.Gitspec=$(git rev-parse HEAD)\""; \
    TAGS="\"ssl sasl\""; \
    eval go build -ldflags ${LD_FLAGS} -o /tmp/mongodb-linux-x86_64-${MONGO_VERSION}/bin/bsondump bsondump/main/bsondump.go && \
    eval go build -ldflags ${LD_FLAGS} -o /tmp/mongodb-linux-x86_64-${MONGO_VERSION}/bin/mongotop -tags ${TAGS} mongotop/main/mongotop.go && \
    eval go build -ldflags ${LD_FLAGS} -o /tmp/mongodb-linux-x86_64-${MONGO_VERSION}/bin/mongodump -tags ${TAGS} mongodump/main/mongodump.go && \
    eval go build -ldflags ${LD_FLAGS} -o /tmp/mongodb-linux-x86_64-${MONGO_VERSION}/bin/mongostat -tags ${TAGS} mongostat/main/mongostat.go && \
    eval go build -ldflags ${LD_FLAGS} -o /tmp/mongodb-linux-x86_64-${MONGO_VERSION}/bin/mongofiles -tags ${TAGS} mongofiles/main/mongofiles.go && \
    eval go build -ldflags ${LD_FLAGS} -o /tmp/mongodb-linux-x86_64-${MONGO_VERSION}/bin/mongooplog -tags ${TAGS} mongooplog/main/mongooplog.go && \
    eval go build -ldflags ${LD_FLAGS} -o /tmp/mongodb-linux-x86_64-${MONGO_VERSION}/bin/mongoimport -tags ${TAGS} mongoimport/main/mongoimport.go && \
    eval go build -ldflags ${LD_FLAGS} -o /tmp/mongodb-linux-x86_64-${MONGO_VERSION}/bin/mongoexport -tags ${TAGS} mongoexport/main/mongoexport.go && \
    eval go build -ldflags ${LD_FLAGS} -o /tmp/mongodb-linux-x86_64-${MONGO_VERSION}/bin/mongorestore -tags ${TAGS} mongorestore/main/mongorestore.go

WORKDIR ${MONGO_SRC}/buildscripts
RUN tar cvzf /tmp/mongodb-linux-x86_64-${MONGO_VERSION}.tgz -C /tmp mongodb-linux-x86_64-${MONGO_VERSION} && \
    python packager.py -s ${MONGO_VERSION} -t /tmp/mongodb-linux-x86_64-${MONGO_VERSION}.tgz -p ~/.

WORKDIR /root
RUN mkdir -p ~/dist/debian/wheezy/mongodb-org/3.0/amd64 && \
    mkdir -p ~/dist/ubuntu/trusty/mongodb-org/3.0/amd64 && \
    mkdir -p ~/dist/ubuntu/precise/mongodb-org/3.0/amd64 && \
    mkdir -p ~/dist/amazon/2013.03/mongodb-org/3.0/x86_64 && \
    mkdir -p ~/dist/redhat/5/mongodb-org/3.0/x86_64 && \
    mkdir -p ~/dist/redhat/6/mongodb-org/3.0/x86_64 && \
    mkdir -p ~/dist/redhat/7/mongodb-org/3.0/x86_64 && \
    mkdir -p ~/dist/suse/11/mongodb-org/3.0/x86_64 && \
    mkdir -p ~/dist/tar/mongodb-org/3.0/x86_64/ && \
    cp -v ~/repo/apt/debian/dists/wheezy/mongodb-org/3.0/main/binary-amd64/*.deb ~/dist/debian/wheezy/mongodb-org/3.0/amd64 && \
    cp -v ~/repo/apt/ubuntu/dists/trusty/mongodb-org/3.0/multiverse/binary-amd64/*.deb ~/dist/ubuntu/trusty/mongodb-org/3.0/amd64 && \
    cp -v ~/repo/apt/ubuntu/dists/precise/mongodb-org/3.0/multiverse/binary-amd64/*.deb ~/dist/ubuntu/precise/mongodb-org/3.0/amd64 && \
    cp -v ~/repo/yum/amazon/2013.03/mongodb-org/3.0/x86_64/RPMS/*.rpm ~/dist/amazon/2013.03/mongodb-org/3.0/x86_64 && \
    cp -v ~/repo/yum/redhat/5/mongodb-org/3.0/x86_64/RPMS/*.rpm ~/dist/redhat/5/mongodb-org/3.0/x86_64 && \
    cp -v ~/repo/yum/redhat/6/mongodb-org/3.0/x86_64/RPMS/*.rpm ~/dist/redhat/6/mongodb-org/3.0/x86_64 && \
    cp -v ~/repo/yum/redhat/7/mongodb-org/3.0/x86_64/RPMS/*.rpm ~/dist/redhat/7/mongodb-org/3.0/x86_64 && \
    cp -v ~/repo/zypper/suse/11/mongodb-org/3.0/x86_64/RPMS/*.rpm ~/dist/suse/11/mongodb-org/3.0/x86_64 && \
    cp -v /tmp/mongodb-linux-x86_64-${MONGO_VERSION}.tgz ~/dist/tar/mongodb-org/3.0/x86_64/ && \
    rm -rfv /tmp/* ~/repo ~/dl ~/dst ~/rpmbuild ~/macros
