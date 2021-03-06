FROM openshift/base-centos7

MAINTAINER Wesley Hearn <whearn@redhat.com>

EXPOSE 8080

ENV DNX_VERSION 1.0.0-beta7
ENV DNX_USER_HOME /opt/dnx
ENV DNX_RUNTIME coreclr

# https://github.com/aspnet/dnx/issues/1590#issuecomment-126965429
ENV MONO_THREADS_PER_CPU 2000

LABEL io.k8s.description="Platform for building and running .NET Next 1.0.0-beta7 applications" \
      io.k8s.display-name=".NET Next 1.0.0-beta7" \
      io.openshift.expose-services="8080:http" \
      io.openshift.tags="builder,dnx,dnx100beta7"

RUN rpm --import "http://keyserver.ubuntu.com/pks/lookup?op=get&search=0x3FA7E0328081BFF6A14DA29AA6A19B38D3D831EF" && \
    yum-config-manager --add-repo http://download.mono-project.com/repo/centos/ && \
    yum install -y mono-complete mono-devel autoconf automake libtool && \
    yum clean all -y

RUN LIBUV_VERSION=1.4.2 \
	&& curl -sSL https://github.com/libuv/libuv/archive/v${LIBUV_VERSION}.tar.gz | tar zxfv - -C /usr/local/src \
	&& cd /usr/local/src/libuv-$LIBUV_VERSION \
	&& sh autogen.sh && ./configure --prefix=/usr --libdir=/usr/lib64 && make && make install \
	&& rm -rf /usr/local/src/libuv-$LIBUV_VERSION \
	&& ldconfig

RUN curl -sSL https://raw.githubusercontent.com/aspnet/Home/dev/dnvminstall.sh | DNX_USER_HOME=$DNX_USER_HOME DNX_BRANCH=v$DNX_VERSION sh

RUN bash -c "source $DNX_USER_HOME/dnvm/dnvm.sh \
	&& dnvm install $DNX_VERSION -r $DNX_RUNTIME \
	&& dnvm alias default | xargs -i ln -s $DNX_USER_HOME/runtimes/{} $DNX_USER_HOME/runtimes/default"

# Copy the STI scripts from the specific language image to /usr/local/sti
COPY ./.sti/bin/ /usr/local/sti

# Each language image can have 'contrib' a directory with extra files needed to
# run and build the applications.
COPY ./contrib/ /opt/app-root/src

# Drop the root user and make the content of /opt/app-root owned by user 1001
RUN chown -R 1001:1001 /opt/app-root
USER 1001

ENV PATH $PATH:$DNX_USER_HOME/runtimes/default/bin

# Set the default CMD to print the usage of the language image
CMD ["usage"]
