########################################
#   Docker container for FUNCTEST
########################################
# All rights reserved. This program and the accompanying materials
# are made available under the terms of the Apache License, Version 2.0
# which accompanies this distribution, and is available at
# http://www.apache.org/licenses/LICENSE-2.0
#

FROM ubuntu:14.04
MAINTAINER Jose Lausuch <jose.lausuch@ericsson.com>
LABEL version="0.1" description="OPNFV Functest Docker container"

# Environment variables
ARG BRANCH=master
ARG RALLY_TAG=0.8.1
ARG TEMPEST_TAG=14.0.0
ARG ODL_TAG=release/beryllium-sr4
ARG OPENSTACK_TAG=stable/mitaka
ARG KINGBIRD_TAG=0.2.2
ARG VIMS_TAG=stable
ARG REPOS_DIR=/home/opnfv/repos
ARG FUNCTEST_BASE_DIR=/home/opnfv/functest
ARG FUNCTEST_CONF_DIR=${FUNCTEST_BASE_DIR}/conf
ARG FUNCTEST_DATA_DIR=${FUNCTEST_BASE_DIR}/data
ARG FUNCTEST_RESULTS_DIR=${FUNCTEST_BASE_DIR}/results
ARG FUNCTEST_REPO_DIR=${REPOS_DIR}/functest
ARG FUNCTEST_TEST_DIR=${FUNCTEST_REPO_DIR}/functest/opnfv_tests
ARG RELENG_MODULE_DIR=${REPOS_DIR}/releng/modules
ARG REPOS_VNFS_DIR=${REPOS_DIR}/vnfs

# Environment variables
ENV HOME /home/opnfv
ENV CONFIG_FUNCTEST_YAML ${FUNCTEST_REPO_DIR}/functest/ci/config_functest.yaml
ENV REPOS_DIR ${HOME}/repos
ENV creds ${FUNCTEST_CONF_DIR}/openstack.creds
ENV TERM xterm
ENV COLORTERM gnome-terminal

WORKDIR ${HOME}

# Packaged dependencies
RUN apt-get update && apt-get install -y \
build-essential \
bundler \
crudini \
curl \
gcc \
git \
libffi-dev \
libgmp3-dev \
libpq-dev \
libssl-dev \
libxml2-dev \
libxslt-dev \
python-dev \
python-mock \
python-pip \
postgresql \
ruby1.9.1-dev \
ssh \
sshpass \
wget \
--no-install-recommends

RUN pip install --upgrade pip

RUN mkdir -p ${REPOS_DIR} \
    && mkdir -p ${REPOS_VNFS_DIR} \
    && mkdir -p ${FUNCTEST_BASE_DIR}/results \
    && mkdir -p ${FUNCTEST_BASE_DIR}/conf \
    && mkdir -p /root/.ssh \
    && chmod 700 /root/.ssh

RUN git config --global http.sslVerify false

# OPNFV repositories
RUN git clone --depth 1 -b $BRANCH https://gerrit.opnfv.org/gerrit/functest ${REPOS_DIR}/functest
RUN git clone --depth 1 -b $BRANCH https://gerrit.opnfv.org/gerrit/copper ${REPOS_DIR}/copper
RUN git clone --depth 1 -b $BRANCH https://gerrit.opnfv.org/gerrit/sdnvpn ${REPOS_DIR}/sdnvpn
RUN git clone --depth 1 -b $BRANCH https://gerrit.opnfv.org/gerrit/domino ${REPOS_DIR}/domino
RUN git clone --depth 1 -b $BRANCH https://gerrit.opnfv.org/gerrit/parser ${REPOS_DIR}/parser
RUN git clone --depth 1 -b $BRANCH https://gerrit.opnfv.org/gerrit/doctor ${REPOS_DIR}/doctor
RUN git clone --depth 1 -b $BRANCH https://gerrit.opnfv.org/gerrit/ovno ${REPOS_DIR}/ovno
RUN git clone --depth 1 -b $BRANCH https://gerrit.opnfv.org/gerrit/promise ${REPOS_DIR}/promise
RUN git clone --depth 1 -b $BRANCH https://gerrit.opnfv.org/gerrit/netready ${REPOS_DIR}/netready
RUN git clone --depth 1 -b $BRANCH https://gerrit.opnfv.org/gerrit/sfc ${REPOS_DIR}/sfc
RUN git clone --depth 1 https://gerrit.opnfv.org/gerrit/securityscanning ${REPOS_DIR}/securityscanning
RUN git clone --depth 1 https://gerrit.opnfv.org/gerrit/releng ${REPOS_DIR}/releng

# OpenStack repositories
RUN git clone --depth 1 -b $OPENSTACK_TAG https://github.com/openstack/networking-bgpvpn ${REPOS_DIR}/bgpvpn
#RUN git clone --depth 1 -b $KINGBIRD_TAG https://github.com/openstack/kingbird.git ${REPOS_DIR}/kingbird
RUN git clone --depth 1 -b $RALLY_TAG https://github.com/openstack/rally.git ${REPOS_DIR}/rally
RUN git clone --depth 1 -b $TEMPEST_TAG https://github.com/openstack/tempest.git ${REPOS_DIR}/tempest

# other repositories
RUN git clone --depth 1 -b $ODL_TAG https://git.opendaylight.org/gerrit/p/integration/test.git ${REPOS_DIR}/odl_test
RUN git clone --depth 1 -b $VIMS_TAG https://github.com/boucherv-orange/clearwater-live-test ${REPOS_VNFS_DIR}/vims-test
RUN git clone --depth 1 https://github.com/wuwenbin2/OnosSystemTest.git ${REPOS_DIR}/onos

RUN pip install -r ${REPOS_DIR}/rally/requirements.txt
RUN pip install -r ${REPOS_DIR}/tempest/requirements.txt

RUN cd ${FUNCTEST_REPO_DIR} \
    && pip install -r requirements.txt \
    && pip install .

RUN cd ${RELENG_MODULE_DIR} \
    && pip install .

RUN find ${FUNCTEST_REPO_DIR} -name "*.py" \
    -not -path "*tests/unit*" |xargs grep __main__ |cut -d\: -f 1 |xargs chmod -c 755 \
    && find ${FUNCTEST_REPO_DIR} -name "*.sh" |xargs grep \#\! |cut -d\:  -f 1 |xargs chmod -c 755

RUN /bin/bash ${REPOS_DIR}/parser/tests/parser_install.sh ${REPOS_DIR}
RUN ${REPOS_DIR}/rally/install_rally.sh --yes

ADD http://download.cirros-cloud.net/0.3.4/cirros-0.3.4-x86_64-disk.img ${FUNCTEST_BASE_DIR}/data/
ADD http://download.cirros-cloud.net/0.3.4/cirros-0.3.4-x86_64-lxc.tar.gz ${FUNCTEST_BASE_DIR}/data/
ADD http://205.177.226.237:9999/onosfw/firewall_block_image.img ${FUNCTEST_BASE_DIR}/data/

RUN gpg --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3
RUN curl -L https://get.rvm.io | bash -s stable

# SNAPS integration
RUN git clone --depth 1 https://gerrit.cablelabs.com/snaps-provisioning ${REPOS_DIR}/snaps
RUN pip install -e ${REPOS_DIR}/snaps/

# SFC integration
RUN /bin/bash -c ". ${REPOS_DIR}/sfc/sfc/tests/functest/setup_scripts/tacker_client_install.sh"
RUN cd ${REPOS_DIR}/sfc && pip install .

# SDNVPN integration
RUN cd ${REPOS_DIR}/sdnvpn && pip install .

RUN cd ${REPOS_DIR}/bgpvpn && pip install .
#RUN cd ${REPOS_DIR}/kingbird && pip install -e .

RUN /bin/bash -c ". /etc/profile.d/rvm.sh \
    && cd ${REPOS_VNFS_DIR}/vims-test \
    && rvm autolibs enable"
RUN /bin/bash -c ". /etc/profile.d/rvm.sh \
    && cd ${REPOS_VNFS_DIR}/vims-test \
    && rvm install 1.9.3"
RUN /bin/bash -c ". /etc/profile.d/rvm.sh \
    && cd ${REPOS_VNFS_DIR}/vims-test \
    && rvm use 1.9.3"
RUN /bin/bash -c ". /etc/profile.d/rvm.sh \
    && cd ${REPOS_VNFS_DIR}/vims-test \
    && bundle install"

RUN sh -c 'curl -sL https://deb.nodesource.com/setup_4.x | sudo -E bash -' \
    && sudo apt-get install -y nodejs \
    && cd ${REPOS_DIR}/promise && sudo npm -g install npm@latest \
    && cd ${REPOS_DIR}/promise/source && npm install

RUN echo "set nocompatible \n\
set backspace=2" \
>> ${HOME}/.vimrc
RUN echo set nocompatible >> ${HOME}/.exrc
RUN echo "alias ll='ls -lh' \n\
. ${FUNCTEST_REPO_DIR}/functest/cli/functest-complete.sh" \
>> ${HOME}/.bashrc
